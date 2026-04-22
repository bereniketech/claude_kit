---
name: input-stack-libinput-udev
description: Configure the Linux input stack (evdev + libinput + udev + xkbcommon) for a custom Wayland compositor with macOS-grade trackpad feel. Covers the event pipeline, per-device quirks, gesture recognition, latency budgets, and the compositor integration points Smithay exposes.
---

# Input Stack: libinput + udev

Use when your L3 compositor needs input that feels as good as macOS. The hard part isn't parsing events — it's per-device tuning, gesture recognition, and latency budgets.

**Related:** `linux-kernel-config-for-custom-os` (evdev + HID symbols), `hardware-profiling-and-driver-mapping` (touchpad IDs), `compositor-input-routing-and-gestures` (Phase 3 — compositor side).

---

## 1. The Pipeline

```
  Hardware (touchpad, keyboard, touchscreen, tablet)
      │
  Kernel HID driver  →  /dev/input/eventN  (evdev)
      │
  libinput  (reads evdev, applies quirks, emits semantic events)
      │
  Compositor  (smithay::backend::libinput)
      │
  xkbcommon  (maps keycodes → keysyms + compose + layout)
      │
  Wayland clients
```

**Rule:** Your compositor never reads `/dev/input/eventN` directly. libinput handles palm rejection, multi-finger gesture recognition, tap-to-click timing, and quirks — reinventing them costs years and ships worse.

---

## 2. What libinput Gives You

Semantic events, already cleaned up:

| Event type | Examples |
|---|---|
| Pointer | relative motion, button press/release, scroll (smooth, axis-based) |
| Keyboard | key press/release (raw keycodes — xkb turns them into symbols) |
| Touch | finger down/up/move with slot IDs for multi-finger |
| Tablet | pen tip, pressure, tilt, pad buttons, ring/strip |
| Gesture | pinch (with scale), swipe (3/4/5 finger), hold |
| Switch | lid closed/open, tablet-mode toggle |

**Rule:** Gestures above 2 fingers are libinput's job, not yours. You consume `LIBINPUT_EVENT_GESTURE_SWIPE_*` and `LIBINPUT_EVENT_GESTURE_PINCH_*` directly — no finger tracking needed. Inertial scrolling and easing curves are the compositor's job (because they belong to animation/render timing).

**Rule:** 2-finger scroll is a scroll event, not a gesture. libinput emits it as `POINTER_AXIS` with `LIBINPUT_POINTER_AXIS_SOURCE_FINGER` — use the source to distinguish from wheel scroll.

---

## 3. What libinput Does NOT Give You

- **XKB layout** — use `libxkbcommon` in the compositor
- **Focus + cursor image** — compositor decides who gets the event
- **Latency enforcement** — that's your render-loop budget, not libinput's
- **Multi-display coordinate space** — compositor maps pointer to an output
- **Key repeat** — compositor runs the repeat timer (configurable per-seat)
- **Drag thresholds for DnD** — application or toolkit decision
- **Inertial scroll easing** — compositor-side smoothing after the touch ends

---

## 4. Required Kernel Config

From `linux-kernel-config-for-custom-os`:
```
CONFIG_INPUT=y
CONFIG_INPUT_EVDEV=y              # /dev/input/event*
CONFIG_INPUT_MOUSEDEV=n           # legacy /dev/input/mice — not needed
CONFIG_INPUT_JOYDEV=y             # only if you care about joysticks
CONFIG_HID=y
CONFIG_HID_GENERIC=y
CONFIG_HID_MULTITOUCH=y           # multi-finger support
CONFIG_I2C_HID_ACPI=y             # modern laptop touchpads
CONFIG_I2C_HID_OF=y               # DT-based systems (aarch64 laptops)
CONFIG_HID_APPLE=y                # Apple keyboards — future Asahi target
CONFIG_INPUT_LEDS=y
```

Explicitly disable in production:
```
CONFIG_INPUT_UINPUT=m             # userspace synthetic devices — keep as module only
CONFIG_HID_SENSOR_HUB=y           # accelerometer/gyro for tablet mode detection
```

**Rule:** `uinput` stays a module. Load it on demand (rare — the most common reason is an accessibility or gaming tool) and guard it with udev + capabilities so agents can't silently inject keystrokes.

---

## 5. udev: Device Access via seats

Wayland compositors get device access via **systemd-logind** (or **seatd**). Logind owns `/dev/input/eventN` and hands file descriptors to the compositor running on the "seat."

The `uaccess` tag in udev rules is how logind knows a device belongs to the active session user:

```
# /etc/udev/rules.d/70-agentos-input.rules
# Mark input devices as user-accessible on the active seat
SUBSYSTEM=="input", KERNEL=="event*", TAG+="uaccess"
SUBSYSTEM=="input", KERNEL=="mouse*", TAG+="uaccess"
# DRM nodes already handled by systemd's shipped rules — don't duplicate
```

Most distro rules already tag input devices. You usually don't need to ship your own unless:
- You want to restrict a specific device to a specific user
- You want to hide a device from the compositor entirely (e.g. a TPM button, a rarely-used sensor)

**Rule:** Never `chmod 666 /dev/input/event*`. Use logind/seatd. Ambient access to raw input devices = keylogger-level compromise.

---

## 6. libinput Quirks

Quirks are per-device `.quirks` files that tell libinput about hardware bugs or idiosyncrasies. Distro-shipped quirks live under `/usr/share/libinput/`, yours go under `/etc/libinput/local-overrides.quirks`.

**Why they matter:** A touchpad's palm rejection threshold, thumb detection, and trackpoint multiplier are all per-device. Without the right quirk, a good touchpad feels terrible.

**Example — Dell XPS 13 Precision touchpad:**
```ini
# /etc/libinput/local-overrides.quirks
[Dell XPS 13 9310 Touchpad]
MatchBus=i2c
MatchName=DELL08AF:00 06CB:7E7E Touchpad
AttrPalmSizeThreshold=200
AttrThumbPressureThreshold=70
AttrEventCode=-ABS_MT_TOUCH_MAJOR
```

**How to find quirks:**
1. Start with `libinput list-devices` — grab the device name
2. Read `man libinput-quirks` for available attributes
3. Measure: run `libinput debug-events --show-keycodes` and reproduce the problem
4. Tune one attribute at a time, reload with `libinput quirks list <device>`

**Rule:** Ship a quirks file for every officially-supported laptop model in the OS. "Works on my machine" quirks that aren't committed become irreproducible disasters.

---

## 7. The Latency Budget

This is the difference between "feels Linux" and "feels macOS."

**Target budgets** (from finger touch to pixel on screen):

| Action | Budget |
|---|---|
| Tap-to-click (finger up → visual click) | ≤16 ms (one frame @60Hz) |
| Pointer move → cursor redraw | ≤8 ms |
| Scroll flick → content moves | ≤16 ms |
| Key press → character on screen | ≤30 ms |
| Four-finger swipe → workspace starts moving | ≤16 ms |

**Where latency comes from:**
1. Hardware sample rate (most modern touchpads report at 125Hz–1000Hz — not the bottleneck)
2. libinput processing (sub-millisecond — not the bottleneck)
3. Compositor event loop (this is the bottleneck if you block on anything synchronous)
4. Render loop (the other bottleneck — frame pacing against vblank)
5. Display refresh (16.6 ms @60Hz — the irreducible minimum)

**Rule:** libinput events must land in the compositor's input handler within 1 ms of arrival. Never run a blocking operation on the libinput fd — dispatch events immediately, buffer state in the compositor.

**Rule:** For `feels like macOS`, you also need:
- 120Hz display if available (cuts all budgets in half)
- Atomic modesetting with per-vblank page flips (no tearing, no unpredictable latency)
- Predictive cursor rendering (use the hardware cursor plane, not composited)
- Inertial scroll easing in compositor animation, not in the input stack

---

## 8. Gesture Taxonomy

Gestures libinput gives you (consume these directly):

| Gesture | Event | Typical mapping |
|---|---|---|
| 3-finger swipe | `GESTURE_SWIPE_*` | app-switcher |
| 4-finger swipe | `GESTURE_SWIPE_*` (4 fingers) | workspace switching |
| 5-finger pinch | `GESTURE_PINCH_*` | show desktop / mission control |
| 2-finger pinch | `GESTURE_PINCH_*` (2 fingers) | zoom in content |
| Hold | `GESTURE_HOLD_*` | right-click or secondary action |

**Rule:** 2-finger gestures are content (pinch-to-zoom in a specific app). 3+ finger gestures are system (workspace / app switch / desktop show). The boundary is non-negotiable — users learn it from macOS.

**Missing from libinput (you implement in the compositor):**
- Swipe with momentum (inertial easing)
- Edge gestures (swipe from display edge)
- Corner triggers (hot corners on mouse) — use pointer-position + output-layout geometry

---

## 9. Keyboard Path + xkbcommon

libinput sends raw keycodes. You run them through `xkbcommon`:

```rust
// Conceptual — Smithay wraps this in XkbContext
let context = xkb::Context::new(CONTEXT_NO_FLAGS);
let keymap = xkb::Keymap::new_from_names(&context, "", "", "us", "", None, 0)?;
let state  = xkb::State::new(&keymap);

// on key event:
let keysym = state.key_get_one_sym(keycode + 8);  // evdev offset
let utf8 = state.key_get_utf8(keycode + 8);
```

Things xkbcommon handles:
- Layout (us, dvorak, fr, etc.)
- Variants (colemak, intl-altgr)
- Compose sequences (dead_acute + e = é)
- Modifier state (shift, ctrl, alt, super, caps, num)
- Level (shift gives level 2, altgr gives level 3)

**Rule:** Never map keycodes yourself. xkbcommon's layout files are authoritative. Compose key + IME are already solved.

**IME integration:** you'll need Wayland's `input-method-v2` and `text-input-v3` protocols for CJK/complex scripts. Out of scope for this skill — flagged for Phase 3 compositor work.

---

## 10. Switch Events

Don't ignore these — they matter for agentic OS UX:
- `SW_LID` — lid closed/opened. Pause agent vision loop. Sleep/hibernate based on policy.
- `SW_TABLET_MODE` — 2-in-1 laptop flipped into tablet. Change input handling (no touchpad, onscreen keyboard, larger hit targets).
- `SW_HEADPHONE_INSERT` — drives WirePlumber device switch.

libinput emits them as `SWITCH_TOGGLE` events. Your compositor dispatches to the right handler.

---

## 11. Testing + Debugging

Tools you'll use constantly:
```bash
libinput list-devices                 # what's visible
libinput debug-events --show-keycodes # live event dump
libinput record > rec.yml             # save an event sequence
libinput replay rec.yml               # replay it (needs uinput)
libinput quirks list <device>         # which quirks apply
libinput measure touchpad-tap         # measure tap latency
evtest /dev/input/event5              # raw evdev, bypass libinput
sudo libinput debug-tablet            # tablet-specific
```

**Rule:** Reproduce every input bug with `libinput record` before fixing. The saved recording makes the bug deterministic and testable.

---

## 12. Integration with Smithay

```rust
// Smithay session + libinput backend
let session = LibSeatSession::new()?;
let mut libinput = LibinputInputBackend::new(session.into(), None);
// libinput → Smithay input event stream
event_loop.handle().insert_source(libinput, |event, _, data| {
    match event {
        InputEvent::PointerMotion { .. } => { /* update cursor */ }
        InputEvent::Keyboard { event, .. } => { /* xkb state update */ }
        InputEvent::GestureSwipeBegin { .. } => { /* start workspace swipe */ }
        // ...
    }
})?;
```

Smithay's `LibinputInputBackend` handles the fd poll + event translation. You implement the handlers.

**Rule:** Use Smithay's wrappers. Writing libinput's FD poll loop yourself is boilerplate you'll get subtly wrong.

---

## 13. Handoff Checklist

- [ ] Every input device in the hardware inventory has a libinput quirk file committed (or an explicit "default quirks sufficient" note)
- [ ] `libinput list-devices` shows touchpad, keyboard, touchscreen (if present), lid switch, tablet-mode switch (if present)
- [ ] `libinput debug-events` emits `GESTURE_SWIPE_*` events when you swipe 3 fingers
- [ ] `libinput measure touchpad-tap` reports tap latency under 16 ms
- [ ] `libinput record` can save a session; `libinput replay` can play it back (dev use)
- [ ] logind/seatd is the seat manager; no `chmod` hacks
- [ ] udev rules don't duplicate distro-shipped `uaccess` tagging
- [ ] xkbcommon layout works for your target locales (test with dead keys)
- [ ] Switch events wire up to lid/tablet-mode handlers

When ticked, the compositor phase (Phase 3) can consume Smithay's input backend directly.
