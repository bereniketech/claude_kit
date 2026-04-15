---
name: hardware-profiling-and-driver-mapping
description: Produce a complete chip → kernel module → firmware blob inventory for one target machine before committing to build a custom OS on it. Identifies broken-support blockers at Phase 0 so they don't kill the project later.
---

# Hardware Profiling & Driver Mapping

Use before any custom-OS work on a specific machine. This is the Phase 0 blocker check: if a critical chip has no Linux driver or a proprietary blob you can't ship, you must know *now*, not after building L2/L3.

**Related:** `linux-kernel-config-for-custom-os` (uses your inventory as input), `drm-kms-and-mesa-basics` (deep on GPU), `pipewire-wireplumber-session` (deep on audio).

---

## 1. The Command Battery

Run on the target machine, booted into a full live distro with every peripheral attached. Capture all outputs into an `inventory/` directory in your OS repo.

```bash
mkdir -p inventory && cd inventory

# System board / BIOS
sudo dmidecode > dmidecode.txt
sudo dmidecode -t system -t bios -t baseboard > system-summary.txt

# Full hardware tree
sudo lshw -json > lshw.json
sudo lshw -short > lshw-short.txt

# PCI devices — the money view
lspci -nnk > lspci.txt                      # [vendor:device] + kernel driver
lspci -nnkvvv > lspci-verbose.txt           # full detail
lspci -t -v > lspci-tree.txt                # topology

# USB
lsusb > lsusb.txt
lsusb -vv 2>/dev/null > lsusb-verbose.txt

# Summary (human-readable)
inxi -Fxxxz > inxi.txt

# Everything udev knows
sudo udevadm info --export-db > udev-db.txt

# CPU
cat /proc/cpuinfo > cpuinfo.txt
lscpu > lscpu.txt

# Memory
sudo dmidecode -t memory > memory.txt

# Firmware currently loaded
cat /sys/kernel/debug/dri/0/name 2>/dev/null > gpu-driver.txt
dmesg > dmesg.txt
dmesg | grep -i firmware > firmware-loaded.txt

# UEFI status
ls /sys/firmware/efi > efi.txt 2>&1
mokutil --sb-state 2>/dev/null > secureboot-state.txt

# Sensors (thermal, battery)
sensors > sensors.txt 2>&1
upower -d > power.txt 2>&1

# Input devices
libinput list-devices > libinput-devices.txt 2>&1

# Audio
aplay -l > audio-cards.txt
arecord -l > audio-inputs.txt
```

**Rule:** Every file committed. Future debugging of "why doesn't X work" always comes back to one of these outputs.

---

## 2. The Inventory Table

Build this from the captured output. It is the *single* artifact every downstream skill consumes.

| Slot | Function | Vendor:Device | Kernel Module | Firmware Blob(s) | Status |
|---|---|---|---|---|---|
| CPU | — | (lscpu) | — | microcode | ✅/⚠/❌ |
| iGPU | graphics | PCI ID | `i915`/`amdgpu`/`nouveau` | list | status |
| dGPU | graphics | PCI ID | `nvidia`/`amdgpu` | list | status |
| Wi-Fi | wireless | PCI/USB ID | `iwlwifi`/`mt76`/... | ucode | status |
| Bluetooth | radio | PCI/USB ID | `btusb`+`btintel`/... | `.sfi`/`.hcd` | status |
| Audio (HDA) | sound | PCI ID | `snd_hda_intel` + codec | SoF blobs if SoF | status |
| Audio codec | DSP | USB/I2C/SOF | `snd_soc_*` | topology `.tplg` | status |
| NVMe | storage | PCI ID | `nvme` | — | status |
| Touchpad | input | I2C HID | `i2c_hid_acpi`+`hid_multitouch` | — | status |
| Keyboard | input | (internal) | `atkbd`/`hid_generic` | — | status |
| Webcam | video | USB ID | `uvcvideo` | — | status |
| Thunderbolt | bus | PCI ID | `thunderbolt` | — | status |
| TPM | security | (dmidecode) | `tpm_crb`/`tpm_tis` | — | status |
| Ethernet | wired | PCI/USB ID | `e1000e`/`r8169`/... | — | status |
| Card reader | storage | PCI ID | `rtsx_pci_sdmmc`/... | — | status |
| Fingerprint | biometric | USB ID | `fprintd`-compatible? | often proprietary | status |
| Cellular modem | WWAN | USB ID | `cdc_mbim`/`qmi_wwan` | often blob | status |

**Status legend:**
- ✅ working in-tree with free firmware (or no firmware needed)
- ⚠ working but needs quirks, proprietary firmware, or workarounds (usable with caveats)
- ❌ BLOCKER — no viable driver, no free firmware, or hardware upstream maintainers have rejected

**Rule:** Every row is filled in. "Unknown" means you haven't finished profiling. "N/A" only for slots the machine doesn't have.

---

## 3. Mapping Chip → Module → Firmware

For each PCI/USB device:

**Step A — extract the ID:**
```bash
lspci -nnk | grep -A2 -i 'network\|audio\|vga\|usb\|sd host'
# e.g. "8086:a0f0 ... Kernel driver in use: iwlwifi"
```

**Step B — confirm the module and find its firmware needs:**
```bash
modinfo iwlwifi | grep -E 'firmware|license|version'
# firmware:       iwlwifi-QuZ-a0-hr-b0-77.ucode
# firmware:       iwlwifi-QuZ-a0-hr-b0-72.ucode
# license:        GPL
```

**Step C — confirm the blob is present and was loaded:**
```bash
# Present on disk?
ls /lib/firmware/iwlwifi* 2>/dev/null | head

# Actually loaded at boot?
dmesg | grep -i 'iwlwifi.*loaded firmware'
```

**Step D — record the licence.** Firmware blobs come in three licences:
- **Redistributable, free** — ship it in the OS image without worry (most Intel, some AMD)
- **Redistributable, proprietary** — ship with the vendor's licence notice (Broadcom, older Nvidia)
- **Non-redistributable** — **BLOCKER**. You can't legally include it in a distributed image.

`linux-firmware` repo's `WHENCE` file is the authoritative map. Read it for every blob you ship.

---

## 4. Worked Example — Dell XPS 13 9310 (Tiger Lake-UP3)

| Slot | Function | Vendor:Device | Kernel Module | Firmware Blob(s) | Status |
|---|---|---|---|---|---|
| CPU | i7-1165G7 | (Tiger Lake) | — | `intel-ucode/06-8c-01` (microcode) | ✅ |
| iGPU | Iris Xe G7 96EU | `8086:9a49` | `i915` | `i915/tgl_dmc_ver2_08.bin`, `i915/tgl_guc_*.bin`, `i915/tgl_huc_*.bin` | ✅ |
| Wi-Fi | Intel AX201 | `8086:a0f0` | `iwlwifi` | `iwlwifi-QuZ-a0-hr-b0-77.ucode` | ✅ |
| Bluetooth | Intel (integrated w/ AX201) | (USB `8087:0026`) | `btusb` + `btintel` | `intel/ibt-0040-1050.sfi`, `intel/ibt-0040-1050.ddc` | ✅ |
| HDA controller | Tiger Lake-LP SST | `8086:a0c8` | `snd_soc_sof_intel_hda_common` | `intel/sof/sof-tgl.ri`, `intel/sof/sof-tgl.ldc`, `intel/sof-tplg/sof-hda-generic-2ch.tplg` | ⚠ needs all three SoF blobs or audio is silent |
| Audio codec | Realtek ALC5682 | `10ec:5682` (I2C) | `snd_soc_rl5682` | (via SoF topology) | ⚠ depends on SoF working |
| NVMe | SK hynix PC711 | `1c5c:1959` | `nvme` | — | ✅ |
| Touchpad | DELL08AF Precision | `06cb:7e7e` (I2C HID) | `hid_multitouch` + `i2c_hid_acpi` | — | ⚠ needs libinput quirk for palm rejection |
| Keyboard | (internal PS/2-style) | (internal) | `atkbd` + `hid_generic` | — | ✅ |
| Webcam | Realtek integrated | `0bda:5634` (USB) | `uvcvideo` | — | ✅ |
| Thunderbolt | Tiger Lake TBT | `8086:9a23` | `thunderbolt` | — | ✅ |
| TPM | fTPM (Intel PTT) | (firmware) | `tpm_crb` | — | ✅ (TPM2) |
| Card reader | Realtek RTS5261 | `10ec:5261` | `rtsx_pci_sdmmc` | — | ✅ |
| Fingerprint | Goodix 53xc | `27c6:5395` (USB) | — | proprietary, userspace `libfprint-2-tod1-goodix` | ❌ blocked — proprietary userspace, non-redistributable |

**Blocker analysis:** fingerprint reader is the only ❌. Workaround: disable the feature in the OS, rely on TPM2-backed password unlock instead. Acceptable trade-off. Everything else is ✅ or ⚠ with known workarounds.

**Firmware manifest** (commit to OS repo):
```
linux-firmware/
  i915/tgl_dmc_ver2_08.bin
  i915/tgl_guc_70.1.1.bin
  i915/tgl_huc_7.9.3.bin
  iwlwifi-QuZ-a0-hr-b0-77.ucode
  intel/ibt-0040-1050.sfi
  intel/ibt-0040-1050.ddc
  intel/sof/sof-tgl.ri
  intel/sof/sof-tgl.ldc
  intel/sof-tplg/sof-hda-generic-2ch.tplg
  intel-ucode/06-8c-01
```

Nothing else from the 400+ MB upstream `linux-firmware` repo is shipped. Every file is justified by a row in the inventory table.

---

## 5. The Phase 0 Blocker Report

If you find any ❌, write a blocker report in this shape and hand it to `os-userland-architect` before anyone writes kit Phase 1+ content:

```
BLOCKER REPORT — <target machine>

Chip: <vendor:device>  (<function>)
Why it's blocking: <no driver / non-redistributable firmware / upstream-rejected>
Upstream status: <link to LKML / driver repo / bug tracker>
Workaround today: <disable feature / use USB dongle / replace component>
Time-to-fix estimate: <months, or UNKNOWN>
Impact: <can-live-without / critical-feature / dealbreaker>
Recommendation: <proceed with feature disabled / wait / replace hardware / cancel>
```

Never paper over a blocker with "we'll deal with it later." That sentence has killed every ambitious custom-OS project.

---

## 6. Freshness Note

Kernel and firmware support changes every release (~10 weeks). An inventory produced against kernel 6.8 may look different against 6.10:
- A ⚠ today can become ✅ when the driver lands
- A ❌ can become ⚠ if an out-of-tree driver goes upstream
- A ✅ can theoretically become ❌ if a driver gets removed (rare)

**Rule:** Re-run the full command battery at every major kernel bump and diff the inventory. Regressions are real.

**Rule:** Every row in the inventory carries an "as of kernel X.Y" note. The table goes stale; say so.

---

## 7. Checklist Before Leaving This Skill

- [ ] All command-battery outputs captured and committed
- [ ] Inventory table complete — every row has a status
- [ ] Firmware manifest produced, every blob justified
- [ ] Licence checked for every blob (`WHENCE` consulted)
- [ ] All ⚠ rows have a named workaround
- [ ] All ❌ rows have a blocker report
- [ ] No row says "unknown"
- [ ] Inventory is dated (kernel version, distro version, date)

If every box is ticked, hand off to `linux-kernel-config-for-custom-os` to produce the `.config`.
