---
name: pipewire-wireplumber-session
description: Wire up PipeWire + WirePlumber for a custom agentic OS — audio, screencapture, and camera routing under one graph daemon. Covers the graph model, policy responsibility split, required services, firmware, WirePlumber Lua policy, and the agent capture path.
---

# PipeWire + WirePlumber Session

Use when your custom OS needs audio out/in, screencapture (for xdg-desktop-portal), or camera routing. PipeWire replaces PulseAudio and JACK simultaneously and is the **only** audio/video-capture daemon on a modern Linux userland.

**Related:** `linux-kernel-config-for-custom-os` (ALSA + SND_SOC symbols), `hardware-profiling-and-driver-mapping` (codec IDs + SoF firmware needs), `systemd-user-units-for-agents` (packaging as user units — Phase 4).

---

## 1. What PipeWire Is

A single graph-based media server daemon that:
- Handles **audio** (replaces PulseAudio and JACK)
- Handles **screencapture** on Wayland (the *only* source — screen pixels flow through PipeWire)
- Handles **camera** (v4l2 devices show up as source nodes)
- Provides **low-latency** pro-audio (drops JACK, replaces it protocol-compatibly)
- Exposes Pulse + JACK protocol shims so existing apps "just work"

**The graph:** PipeWire runs a directed graph of media nodes. Sources (mic, app playing audio, desktop capture) push buffers into the graph. Sinks (speakers, recording apps, screencast client) consume them. Links between nodes route the flow. Ports are typed (audio channels, video frames) and capability-gated (sample rate, format, format-modifiers for video).

**Rule:** You don't hand-link nodes. That's the session manager's job — and WirePlumber is your session manager.

---

## 2. Responsibility Split: PipeWire vs WirePlumber

| Concern | Where |
|---|---|
| The graph itself, buffers, transport, realtime | **PipeWire** (daemon) |
| What connects to what, default devices, volume policy, routing rules | **WirePlumber** (session manager, Lua policies) |
| Pulse protocol compatibility | `pipewire-pulse` (thin shim) |
| JACK protocol compatibility | `pipewire-jack` (thin shim) |

**Rule:** Never edit PipeWire config for routing decisions. That goes in WirePlumber. "Which microphone is default" is a policy question, not a daemon question.

---

## 3. Required Daemons / Services

Ship as systemd **user** units (one instance per user session, not system-wide):

```
pipewire.service            # the media graph daemon
pipewire.socket             # socket activation
pipewire-pulse.service      # PA protocol shim
pipewire-pulse.socket
wireplumber.service         # policy / session manager
```

Enable for each user via `systemctl --user enable pipewire pipewire-pulse wireplumber`, or ship enabled in `/etc/systemd/user/default.target.wants/`.

**Rule:** PipeWire is a *per-user* daemon. Do not run it as a system service. A second user session gets its own graph.

**Ordering:** `wireplumber` depends on `pipewire`. `pipewire-pulse` depends on `pipewire`. systemd handles this via `Requires=pipewire.service` in their unit files — don't override.

---

## 4. Required Kernel + Firmware

From the kernel side (owned by `linux-kernel-config-for-custom-os`):
```
CONFIG_SND=y
CONFIG_SND_HDA_INTEL=y           # HDA controller
CONFIG_SND_HDA_CODEC_<VENDOR>=y  # matched to your inventory
CONFIG_SND_SOC=y                 # Sound Open Firmware on Tiger Lake+
CONFIG_SND_SOC_SOF_INTEL_TOPLEVEL=y    # SoF Intel support
CONFIG_SND_SOC_SOF_TOPLEVEL=y
CONFIG_SND_USB_AUDIO=y
CONFIG_MEDIA_SUPPORT=y
CONFIG_VIDEO_DEV=y
CONFIG_USB_VIDEO_CLASS=y         # UVC webcams
```

From the firmware side:
- **HDA** on legacy Intel: no firmware
- **SoF** on Tiger Lake+ Intel: you *must* ship the three-file set per platform:
  - `/lib/firmware/intel/sof/sof-<platform>.ri` (firmware)
  - `/lib/firmware/intel/sof/sof-<platform>.ldc` (log decoder)
  - `/lib/firmware/intel/sof-tplg/sof-<codec>.tplg` (topology)
  Miss any one → silent boot, no audio, no obvious error without SoF debug on.

**Verification post-boot:**
```bash
cat /proc/asound/cards        # at least one card listed
dmesg | grep -iE 'sof|hda'    # SoF firmware loaded, topology parsed
aplay -l                      # userspace sees the devices
pw-cli list-objects | head    # PipeWire sees them as nodes
```

**Rule:** If `aplay -l` shows cards but PipeWire doesn't, the problem is WirePlumber Lua policy, not the kernel. If `aplay -l` shows nothing, the kernel/firmware side is broken — go back to the hw-profiling skill.

---

## 5. WirePlumber Lua Policy

WirePlumber's config lives in `/etc/wireplumber/` (system) and `$XDG_CONFIG_HOME/wireplumber/` (per-user). As of WirePlumber 0.5, the config format is Lua 5.4.

**Default priorities we want in a custom OS:**
1. Wired headphones > Bluetooth > internal speakers (on playback)
2. USB headset mic > internal mic > Bluetooth (on capture)
3. HDMI audio available but *not* auto-selected on hotplug
4. Default microphone: **none** until the user (or agent, with capability) grants access
5. Default camera: **none** until capability granted

**Rule:** No device is auto-routed for capture (mic or camera). Capture access is a capability, not a default. The L4 agent layer's privacy model depends on this.

**Example fragment** — prefer headphones when present:
```lua
-- /etc/wireplumber/wireplumber.conf.d/50-default-output.lua
rule = {
  matches = {
    {
      { "node.name", "matches", "alsa_output.pci-*.analog-stereo" },
      { "device.icon_name", "matches", "audio-headphones*" },
    },
  },
  apply_properties = {
    ["priority.session"] = 2000,   -- > default speaker priority (~1000)
  },
}
table.insert(alsa_monitor.rules, rule)
```

**Rule:** Put all custom policies in `wireplumber.conf.d/<NN>-<name>.lua` drop-ins. Don't edit the main config — you'll lose it on update.

---

## 6. The Screencapture Path (Wayland)

PipeWire is the pipe between the Wayland compositor and any screencasting consumer (OBS, meeting apps, agent vision loop). The flow:

```
L3 Compositor (implements ext-image-copy-capture-v1 or older wlr-screencopy-v1)
    │
    ▼
xdg-desktop-portal-* (prompts user, checks permissions)
    │
    ▼
PipeWire creates a stream node the client connects to
    │
    ▼
Client (OBS / agent vision loop) reads frames from the node
```

**What you have to ship:**
- `xdg-desktop-portal` (the parent daemon)
- A portal backend — for your own compositor this will be a custom one (kit Phase 3 `xdg-desktop-portals-implementation`), but until then you can use `xdg-desktop-portal-wlr` as a stopgap on wlroots-derived compositors
- PipeWire with video support (default in modern builds)

**Rule:** Do not let clients open `/dev/video*` or screen capture devices directly. Everything goes through the portal → PipeWire. That's your capability boundary for camera + screen.

---

## 7. Camera Routing

Cameras on Linux = V4L2 devices (`/dev/video*`). PipeWire ingests them via the `libcamera` backend (modern) or direct `v4l2` backend (legacy). As of PipeWire 1.0+, `libcamera` is preferred — it handles modern ISPs and complex camera pipelines.

Config fragment — enable libcamera monitor:
```
# /etc/pipewire/pipewire.conf.d/50-libcamera.conf
context.modules = [
  { name = libpipewire-module-libcamera-monitor }
]
```

**Rule:** Only *one* of `libpipewire-module-libcamera-monitor` or `libpipewire-module-v4l2-monitor` should be enabled at a time, or both will enumerate the same cameras.

**Privacy:** Cameras are capture devices. Same rule as mics — never auto-route, always capability-gated.

---

## 8. Agent Integration Points

For the L4 agentic layer (kit Phases 6–7), PipeWire is the single integration surface for:

| Agent capability | Wired through |
|---|---|
| Hearing the user (voice control) | PipeWire source → Whisper.cpp client |
| Seeing the screen (vision loop) | xdg-desktop-portal → PipeWire stream → VLM client |
| Speaking (TTS output) | Piper/Whisper → PipeWire sink |
| Hearing system audio (captioning) | PipeWire monitor source (loopback of sink) |

**Rule:** Each of those becomes one MCP tool (`mcp-server-audio-and-network` in kit Phase 7). They are *tools*, not ambient capabilities. Agent needs to call `voice.start_listening` and provide a capability token — PipeWire doesn't enforce that, the MCP server does.

---

## 9. Troubleshooting Decision Tree

**No audio at all**
→ `aplay -l` empty? Kernel/firmware broken. Back to `hardware-profiling-and-driver-mapping`.
→ `aplay -l` has cards but no sound? Try `pw-cat -p /usr/share/sounds/alsa/Front_Center.wav`. If silent, check volume: `pw-cli ls Node | grep -A5 'default.audio.sink'` — is there a default sink? Is it muted?
→ Still silent? Check WirePlumber logs: `journalctl --user -u wireplumber -b`

**SoF audio silent on Tiger Lake+**
→ `dmesg | grep sof` — firmware loaded? If not, missing blob — go to firmware manifest.
→ Firmware loaded but silent? Topology blob wrong (`.tplg` mismatched to codec). Check codec chip from `lspci -nnk` and match against `linux-firmware/intel/sof-tplg/` filenames.

**Bluetooth audio crackles / disconnects**
→ Usually codec negotiation: enable mSBC for HFP, opt into LDAC/aptX via `wireplumber.conf.d`. Document which codecs are shipped and which are optional (LDAC patent).

**Mic has high latency**
→ PipeWire graph rate mismatch. Force `default.clock.rate = 48000`, `default.clock.quantum = 1024` in `pipewire.conf.d`. Trade-off: lower quantum = less latency, more CPU.

**Screencapture fails from portal**
→ Check `xdg-desktop-portal` is running. Check the backend (`xdg-desktop-portal-wlr` or your own) is registered. Check PipeWire video support: `pw-cli list-objects | grep -i video`.

---

## 10. What NOT to Do

**Rule:** Do not run PulseAudio as a daemon alongside PipeWire. `pipewire-pulse` *replaces* `pulseaudio` — if both run, chaos.

**Rule:** Do not run JACK daemon alongside PipeWire. `pipewire-jack` replaces it; `jackd2` alongside PipeWire will fight over device grabs.

**Rule:** Do not hardcode device names in user-facing code. Names change across reboots ("alsa_output.pci-0000_00_1f.3.analog-stereo" is not stable). Use the session manager's concept of "default sink" / "default source."

**Rule:** Do not expose `/dev/video*` or `/dev/snd/*` directly to sandboxed apps or agents. All access goes through PipeWire + portal.

---

## 11. Handoff Checklist

- [ ] `pw-cli list-objects` shows all expected sources and sinks after boot
- [ ] A reference playback (`pw-cat -p test.wav`) produces sound on default sink
- [ ] Volume up/down works through `pactl set-sink-volume @DEFAULT_SINK@ +5%`
- [ ] Bluetooth audio pairs, connects, produces sound (if the hardware is shipped)
- [ ] Webcam shows up as a node (`pw-cli list-objects | grep camera`)
- [ ] Screencapture works end-to-end with `grim` or similar (validates the portal path)
- [ ] WirePlumber policy commits live in `wireplumber.conf.d/`, not the main file
- [ ] SoF firmware manifest (if applicable) is complete and licence-checked
- [ ] No PulseAudio, no JACK daemon, no ALSA grabs outside PipeWire

When every box is ticked, audio + capture are ready for the L3 compositor and eventually the L4 agent layer.
