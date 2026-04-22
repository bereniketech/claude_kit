---
name: drm-kms-and-mesa-basics
description: The minimum DRM/KMS, GBM, and EGL/Vulkan knowledge needed to drive a custom Wayland compositor on Linux without writing GPU drivers. Covers card detection, modesetting, buffer allocation, and the handoff points Smithay/wlroots-rs rely on.
---

# DRM/KMS and Mesa Basics

Use when your L3 compositor needs to talk to the GPU and you want enough of the mental model to pick the right Smithay/wlroots-rs backend, debug a black screen, and set a mode without re-inventing mode-setting.

**You do not write GPU drivers.** In-tree Mesa + the in-tree kernel driver is the plan. This skill teaches you *where to stop*.

**Related:** `linux-kernel-config-for-custom-os` (DRM symbols), `hardware-profiling-and-driver-mapping` (which GPU), `wayland-compositor-with-smithay` (Phase 3 — how the compositor consumes this).

---

## 1. The Stack in One Diagram

```
 ┌──────────────────────────────┐
 │  L3 Compositor (your Rust)   │  uses smithay::backend::drm
 └───────────────┬──────────────┘
                 │ libgbm + libdrm (userspace)
                 │ EGL / Vulkan (Mesa)
 ┌───────────────┴──────────────┐
 │  Mesa (userspace driver)     │  iris / radeonsi / zink / nvk
 └───────────────┬──────────────┘
                 │ DRM/KMS ioctls on /dev/dri/cardN
 ┌───────────────┴──────────────┐
 │  Kernel DRM driver           │  i915 / amdgpu / nouveau
 └──────────────────────────────┘
```

**Rule:** Your code touches the top box. libdrm + libgbm + EGL/Vulkan are all existing libraries — you link them, you don't reimplement them.

---

## 2. DRM vs KMS vs GBM vs EGL

Four acronyms, distinct jobs:

| Layer | Owner | Does |
|---|---|---|
| **DRM** (Direct Rendering Manager) | kernel + libdrm | Authority over `/dev/dri/cardN`, GPU resource ioctls, buffer objects (GEM), fences |
| **KMS** (Kernel Mode Setting) | kernel | Connectors, encoders, CRTCs, planes — the modesetting half of DRM |
| **GBM** (Generic Buffer Manager) | libgbm (Mesa) | Allocates DMA-BUF buffers you can both render into (via EGL) and scan out (via KMS) |
| **EGL / Vulkan** | Mesa | Draw things into those buffers |

**Rule:** Your compositor loop is:
1. GBM allocates a buffer
2. EGL/Vulkan renders into it
3. KMS scans it out to a CRTC → connector → physical display
4. Page-flip on vblank; repeat

Smithay wraps all four — `smithay::backend::drm`, `smithay::backend::allocator::gbm`, `smithay::backend::renderer::gles` or `vulkan`. You compose these, you don't rebuild them.

---

## 3. Device Enumeration

`/dev/dri/card0`, `/dev/dri/card1`, `/dev/dri/renderD128`...

- `cardN` — primary nodes, allow modesetting (privileged — need `CAP_SYS_ADMIN` or `seat` access via logind)
- `renderDN` — render nodes, allocation + drawing only, no modesetting (safe for unprivileged clients)

**In a compositor:** open the primary node (`card0`) — you're the one doing modesetting. Use logind or seatd to get the fd without running as root.

```rust
// conceptual — Smithay handles the details
let session = LibSeatSession::new()?;
let (device_fd, ..) = session.open_device("/dev/dri/card0")?;
let drm = DrmDevice::new(device_fd, ..)?;
```

**In client apps:** open a render node — they only need to draw into a shared buffer.

**Rule:** Never `chmod 666 /dev/dri/card0`. Use logind/seatd. Ambient root is an architectural failure at the L2 layer.

---

## 4. The KMS Object Model

Four object types you must understand:

| Object | Role |
|---|---|
| **Connector** | A physical output socket (eDP-1 internal display, HDMI-A-1, DP-2) |
| **Encoder** | Glue between CRTC and connector (mostly transparent to you) |
| **CRTC** (Cathode Ray Tube Controller — legacy name) | The scanout engine that composites planes and outputs to an encoder |
| **Plane** | A layer within a CRTC (primary, overlay, cursor) |

**Modesetting =** assign CRTC → connector, set a mode (resolution+refresh), attach a framebuffer to the primary plane, commit.

**Inspecting:**
```bash
ls /sys/class/drm/card0-*                # one dir per connector
cat /sys/class/drm/card0-eDP-1/status    # "connected" / "disconnected"
cat /sys/class/drm/card0-eDP-1/modes     # modes this display reports
```

Or use `modetest` (from `libdrm`):
```bash
modetest -M i915                          # list objects
modetest -M i915 -s 32@42:1920x1080@60   # set a mode — shows test pattern
```

**Rule:** Before your compositor runs, `modetest` must work. If `modetest` can't set a mode, your compositor can't either.

---

## 5. Buffer Allocation via GBM

GBM gives you buffer objects that can be **both** scanout-safe (KMS can display them) and render-targets (EGL/Vulkan can draw into them).

Flags to know:
- `GBM_BO_USE_SCANOUT` — usable as a KMS framebuffer
- `GBM_BO_USE_RENDERING` — usable as an EGL/Vulkan render target
- `GBM_BO_USE_LINEAR` — linear layout (cross-vendor buffer sharing)
- Modifiers — the tile/compression format; query supported modifiers via KMS and intersect with the renderer's supported list

**Modifiers are the hard part.** Intel GPUs want X-tiled or Y-tiled layouts; AMD wants DCC; Nvidia wants something else. Always query, intersect, pick the highest-preference common modifier. Smithay's `GbmBufferedSurface` does this; don't roll your own.

**Rule:** Never allocate a linear scanout buffer when a tiled modifier is available — you lose substantial GPU perf and power. Only fall back to linear for cross-vendor handoff (e.g. multi-GPU hybrid systems).

---

## 6. Atomic Modesetting

Old (legacy) KMS uses separate ioctls for each plane update. Atomic KMS updates everything in one commit — the kernel rejects inconsistent states at submit time, so you get all-or-nothing frames.

**Always use atomic.** Smithay defaults to atomic on drivers that support it (every modern driver: i915, amdgpu, nouveau, msm, rockchip...).

Atomic commit types:
- **ATOMIC_TEST_ONLY** — "would this work?" — no state change. Use this during plane assignment to discover what the hardware accepts.
- **ATOMIC_NONBLOCK** — normal frame, returns immediately; vblank event delivered later
- (blocking) — for modesetting and startup, not per-frame

**Rule:** Always TEST_ONLY before the real commit when assigning overlay planes to client surfaces — driver rejections depend on resolution, format, and modifier combos that are hardware-specific.

---

## 7. EGL vs Vulkan for the Compositor

**EGL + GLES3** — what wlroots and Smithay's `GlesRenderer` use. Simpler, works on every driver. The default choice.

**Vulkan** — Smithay has a `VulkanRenderer`. More complex, but:
- Better multi-GPU story
- Explicit synchronization (helps with latency debugging)
- Timeline semaphores — the cleanest way to sync renderer and display
- Future-proof for HDR and advanced color management

**Rule:** Start with `GlesRenderer`. Migrate to `VulkanRenderer` only when EGL+GLES can't express what you need (explicit sync for a hard latency budget, multi-GPU, HDR pipeline). "Because Vulkan is newer" is not a reason.

---

## 8. Display Setup Order for First Boot

When the compositor first starts, do these in order:

1. **Open the session and DRM device** (logind/seatd → `card0` fd)
2. **Scan connectors** — find which are connected
3. **Read EDID** to get supported modes and preferred mode
4. **Pick a mode** — prefer the connector's preferred mode, fall back to highest resolution at ≥60Hz
5. **Allocate a GBM surface** — matching format+modifiers
6. **Create a GL/Vulkan context** against the GBM surface
7. **Render the first frame** (can be solid color)
8. **Atomic commit** — assign CRTC to connector, set mode, attach framebuffer, submit
9. **Wait for vblank** — acknowledge and render frame 2

If step 8 fails, nothing on-screen. Your debug tools at that point are `dmesg`, the atomic commit's ioctl errno, and `modetest` to verify your mode is actually valid.

**Rule:** On first failure, `modetest` to isolate: is it the driver, the EDID, your format choice, or your code?

---

## 9. Hybrid Graphics (iGPU + dGPU)

Laptops with Intel iGPU + Nvidia/AMD dGPU are the common trap.

**Options for a custom OS:**
- **iGPU only** — blacklist the dGPU module. Simplest, lowest power, no dGPU benefits. Good for Phase 1.
- **dGPU only** — power-hungry, but avoids muxing complexity
- **PRIME render offload** — render on dGPU, present on iGPU. Works but complicates buffer sharing (DMA-BUF across devices, modifier hell)
- **Reverse PRIME** — external HDMI is wired to dGPU, internal display to iGPU — common on gaming laptops

**Rule:** Pick one upfront. "We'll support PRIME later" is a 2-person-month surprise. For a single target laptop, iGPU-only is the right Phase 1 choice unless the machine has no iGPU.

---

## 10. Debugging a Black Screen

Order of checks:
1. `dmesg | grep -iE 'drm|gpu|i915|amdgpu|nouveau'` — driver bound? firmware loaded?
2. `ls /sys/class/drm/card0-*` — connectors enumerated?
3. `cat /sys/class/drm/card0-*/status` — is *any* connector `connected`?
4. `modetest -M <driver>` — can any client set a mode?
5. Run `weston --backend=drm-backend.so` — known-good Wayland compositor. If Weston works, your code is the bug.
6. `strace -e ioctl` on your compositor — does the `DRM_IOCTL_MODE_ATOMIC` return errno? Which?
7. `drm.debug=0x1ff` on the kernel cmdline — floods dmesg with DRM debug output

**Rule:** Always bisect with Weston. It removes all variables except the driver and your hardware.

---

## 11. What Smithay Wraps (So You Don't Have To)

| Concern | Smithay API |
|---|---|
| Open DRM device via session | `smithay::backend::session::libseat::LibSeatSession` + `DrmDeviceFd` |
| DRM device lifecycle | `smithay::backend::drm::DrmDevice` |
| GBM allocator | `smithay::backend::allocator::gbm::GbmAllocator` |
| Surface compositing to scanout | `smithay::backend::drm::compositor::DrmCompositor` |
| Renderer abstraction | `smithay::backend::renderer::gles::GlesRenderer` or `vulkan::VulkanRenderer` |
| Modifier negotiation | `GbmBufferedSurface` + `DrmSurface` |

**Rule:** If you find yourself writing raw libdrm ioctl calls, stop. Smithay has a wrapper. If it doesn't, contribute one upstream — don't fork.

---

## 12. Handoff Checklist to the Compositor

Before you consider this skill's work done and hand off to `wayland-compositor-with-smithay`:

- [ ] `modetest` sets a mode on every connector the laptop has (internal + HDMI + DP)
- [ ] Weston boots and runs the `simple-egl` client without artifacts
- [ ] DMC/GuC/HuC firmware (or equivalent) confirmed loaded in dmesg
- [ ] Atomic modesetting supported (`cat /sys/module/drm/parameters/driver_load_retry` style checks)
- [ ] GBM modifier query returns non-LINEAR modifiers for scanout (performance-critical)
- [ ] Decision made: GLES3 or Vulkan renderer for this target
- [ ] Decision made: iGPU-only / dGPU-only / PRIME — documented

If any row is not ticked, the Phase 3 compositor work will hit it and stall.

---

## 13. What I Won't Do

**Rule:** This skill will not walk you through writing a DRM driver. If the in-tree driver doesn't work, the answer is either "fix the firmware blob," "file an upstream bug," or "use different hardware" — not "fork it."

**Rule:** This skill will not try to replace `libdrm`, `libgbm`, `Mesa`, or `libinput`. Those are each many person-years of work, and Smithay already bridges them.
