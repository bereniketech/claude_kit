---
name: linux-kernel-config-for-custom-os
description: Produce a minimal, reproducible Linux kernel .config tailored to one specific target machine for a custom agentic OS. Covers defconfig vs localmodconfig, what to strip, what is load-bearing for L2/L3/L4, diff tracking, and verification.
---

# Linux Kernel Config for a Custom OS

Use when you need to build your own kernel package for a custom OS and the target is one specific laptop or board, not a generic distro kernel.

**Related:** `hardware-profiling-and-driver-mapping` (run first — you need the chip inventory), `drm-kms-and-mesa-basics` (GPU side), `input-stack-libinput-udev` (input side).

---

## 1. Pick the Starting Point

| Start from | When | What you get |
|---|---|---|
| `make defconfig` | No target hw yet, just want it to build | ~5–7k symbols, everything generic |
| Distro `/boot/config-$(uname -r)` | You have a live distro running on the target | ~10k symbols, full hw coverage, way too fat |
| `make localmodconfig` on the distro kernel | You booted a distro on the target with everything plugged in | Only what's loaded right now — best baseline |
| Hand-reduced from `localmodconfig` | Production custom OS | Smallest, longest to author |

**Rule:** Start with `localmodconfig` on a full session with every peripheral connected (external display, dock, webcam, Bluetooth paired, Wi-Fi active, USB audio). What isn't loaded in `lsmod` at that moment gets stripped. Forgotten peripherals become post-boot surprises.

---

## 2. The Reproducible Workflow

```bash
# On the target machine, booted into a full distro kernel:
lsmod > /tmp/lsmod.txt
cd linux-<version>/
cp /boot/config-$(uname -r) .config
make LSMOD=/tmp/lsmod.txt localmodconfig    # reduces modules
make oldconfig                              # answer new symbols
make menuconfig                             # (optional) hand-tune
scripts/diffconfig /boot/config-$(uname -r) .config > config.diff
```

Commit `.config` **and** `config.diff` to your OS repo. The diff is the auditable record of what you stripped from the distro baseline.

**Rule:** Every symbol you flip to `n` lands with a one-line comment in a tracked `config-notes.md`. "Why is `CONFIG_NET_IPV6=n`?" will be unanswerable in six months otherwise.

---

## 3. What to Strip (Aggressive)

Cut these first — they're the fat:

| Category | Kill | Why |
|---|---|---|
| Filesystems | `REISERFS`, `JFS`, `XFS` (unless used), `NFS` (unless used), `NTFS`, `HFS*`, `AFS`, `CODA`, `CEPH` | Modern OS needs ext4/btrfs + vfat for ESP |
| Network protocols | `DECNET`, `APPLETALK`, `X25`, `IPX`, `LAPB`, `ROSE`, `NETROM` | Legacy ham-radio-era stuff |
| Legacy hardware | `FLOPPY`, `PARPORT`, `GAMEPORT`, `ISA`, `PCMCIA`, `PCCARD` | No modern laptop has these |
| Alternate GPU drivers | Everything except your actual GPU (`DRM_I915` OR `DRM_AMDGPU`, not both) | You know what you have |
| Alternate Wi-Fi drivers | Everything except your actual chip | Same |
| Input legacy | `INPUT_JOYSTICK`, `INPUT_TABLET` (unless used), `GAMEPORT_*` | Keep only what your inventory shows |
| Sound legacy | `SND_OSSEMUL`, `SND_ISA`, `SND_PPC`, SB16/ESS/OPL3 drivers | PipeWire uses ALSA native |
| Architectures | Every `CONFIG_<arch>_*` except your target | Obvious |
| Virtualization host | `KVM`, `XEN_*` | If you're not running VMs on this box |
| Debug/tracing | `DEBUG_INFO`, `FTRACE`, `KPROBES` for production (keep for dev) | Size savings; dev build re-enables |

**Rule:** If a symbol's help text says "unless you have this specific hardware," and your inventory doesn't show it, strip it.

---

## 4. What is Load-Bearing (Never Strip)

These keep your L2/L3/L4 stack working. Miss one and something breaks subtly, post-boot.

**L2 sandbox + services (`os-system-services` skills depend on these):**
```
CONFIG_CGROUPS=y
CONFIG_CGROUP_CPU=y
CONFIG_CGROUP_MEMORY=y
CONFIG_CGROUP_PIDS=y
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_BPF=y
CONFIG_MEMCG=y
CONFIG_BLK_CGROUP=y
CONFIG_USER_NS=y
CONFIG_PID_NS=y
CONFIG_NET_NS=y
CONFIG_UTS_NS=y
CONFIG_IPC_NS=y
CONFIG_SECCOMP=y
CONFIG_SECCOMP_FILTER=y
CONFIG_SECURITY=y
CONFIG_SECURITY_LANDLOCK=y
CONFIG_BPF=y
CONFIG_BPF_SYSCALL=y
CONFIG_BPF_JIT=y
CONFIG_CGROUP_BPF=y
```

**Boot + early userspace:**
```
CONFIG_DEVTMPFS=y
CONFIG_DEVTMPFS_MOUNT=y
CONFIG_TMPFS=y
CONFIG_TMPFS_POSIX_ACL=y
CONFIG_TMPFS_XATTR=y
CONFIG_EFI=y
CONFIG_EFI_STUB=y
CONFIG_EFIVAR_FS=y
CONFIG_EFI_VARS=y
CONFIG_FB_EFI=y
CONFIG_RD_GZIP=y    # or RD_ZSTD — must match initramfs compressor
```

**Crypto (LUKS root, signed updates, TPM):**
```
CONFIG_DM_CRYPT=y
CONFIG_CRYPTO_AES=y
CONFIG_CRYPTO_AES_NI_INTEL=y   # x86_64 hw accel
CONFIG_CRYPTO_XTS=y
CONFIG_CRYPTO_SHA256=y
CONFIG_TCG_TPM=y
CONFIG_TCG_TIS=y
CONFIG_TCG_CRB=y                # modern TPM2 interface
CONFIG_HW_RANDOM=y
CONFIG_HW_RANDOM_TPM=y
```

**L3 compositor (DRM + input):**
```
CONFIG_DRM=y
CONFIG_DRM_KMS_HELPER=y
CONFIG_DRM_<VENDOR>=y           # ONE: I915 / AMDGPU / NOUVEAU
CONFIG_DRM_FBDEV_EMULATION=y    # early-boot tty on the GPU
CONFIG_INPUT_EVDEV=y
CONFIG_HID_GENERIC=y
CONFIG_HID_MULTITOUCH=y
CONFIG_I2C_HID_ACPI=y           # modern laptop touchpads
```

**Audio + capture (PipeWire side):**
```
CONFIG_SND=y
CONFIG_SND_HDA_INTEL=y
CONFIG_SND_HDA_CODEC_REALTEK=y  # or whatever your codec is
CONFIG_SND_SOC=y                # SoF on Tiger Lake+
CONFIG_SND_USB_AUDIO=y
CONFIG_MEDIA_SUPPORT=y
CONFIG_MEDIA_USB_SUPPORT=y
CONFIG_VIDEO_DEV=y
CONFIG_USB_VIDEO_CLASS=y
```

**Network:**
```
CONFIG_NET=y
CONFIG_INET=y
CONFIG_IPV6=y                   # drop only if you're sure; most cloud needs it
CONFIG_NETFILTER=y
CONFIG_NF_TABLES=y              # nftables — drop legacy iptables paths
CONFIG_WIREGUARD=y
CONFIG_CFG80211=y
CONFIG_MAC80211=y
```

**Rule:** Keep `CONFIG_MODULES=y` even if you want a smaller kernel. Modules are how you iterate on drivers without a full rebuild.

---

## 5. Size vs Features Trade-offs

Every additional built-in adds to bzImage. Rough budget for a modern laptop custom OS kernel:

| Target | Size | Notes |
|---|---|---|
| Minimal embedded | <2 MB compressed | No filesystems, no network — not your case |
| Custom laptop OS (this) | 8–14 MB compressed | Reasonable with aggressive strip |
| Full distro kernel | 50+ MB compressed | Covers every chipset — fat |

**Rule:** Measure, don't guess. Run `ls -lh arch/x86/boot/bzImage` after every meaningful config change. If a change balloons the kernel by >10%, justify it.

**Boot time** is mostly initramfs + userspace, not the kernel itself. Don't over-optimize kernel size at the cost of features — aim for "small enough" (say <15 MB) and move on.

---

## 6. Cross-Arch Preparation (HAL mindset)

You'll port this to aarch64 (Apple Silicon target). Keep the config portable:

- Use `make ARCH=arm64 defconfig` as the starting point for the aarch64 variant — DO NOT try to use the same `.config`
- Keep a per-arch config directory: `config/x86_64/.config`, `config/aarch64/.config`, shared `config/common.notes`
- Enable the same load-bearing symbols on both (cgroups, seccomp, landlock, BPF, DRM, etc.)
- Arch-specific items (iGPU driver, CPU freq scaler, IOMMU flavor) live only in their arch config

**Rule:** A symbol that is load-bearing for the L2/L3/L4 stack must be enabled on every supported arch. CI builds both configs on every commit — drift is caught immediately.

---

## 7. Verification Checklist

After building and booting, verify:

```bash
# 1. Kernel loaded all expected modules
lsmod | wc -l                                           # sanity-check count
dmesg | grep -iE 'error|fail|firmware'                  # anything wrong?

# 2. Critical subsystems up
mount | grep cgroup2                                    # cgroups v2 mounted
ls /sys/fs/bpf                                          # BPF fs mounted
cat /proc/sys/kernel/unprivileged_userns_clone          # = 1
grep -c 'seccomp' /proc/self/status                     # seccomp available

# 3. GPU up
cat /sys/class/drm/card0/device/driver/module/version   # driver bound
dmesg | grep -i drm                                     # clean init

# 4. Audio subsystem up
cat /proc/asound/cards                                  # cards detected
ls /dev/snd/                                            # device nodes exist

# 5. Input devices enumerated
ls /dev/input/event*                                    # evdev nodes
libinput list-devices                                   # libinput sees them
```

If any check fails, your config is wrong — don't move forward until it's clean.

---

## 8. Concrete Starting Point — Tiger Lake Laptop

A worked example for an Intel 11th-gen (Tiger Lake-UP3) laptop:

```bash
# Starting from Fedora's kernel config as baseline
cp /boot/config-$(uname -r) .config

# Strip alternative GPU vendors
scripts/config --disable DRM_AMDGPU --disable DRM_NOUVEAU --disable DRM_RADEON
scripts/config --disable DRM_VC4 --disable DRM_VMWGFX

# Strip alternative Wi-Fi families (keep only iwlwifi)
scripts/config --disable BRCMFMAC --disable RT2800USB --disable RTL8XXXU
scripts/config --disable ATH9K --disable ATH10K --disable ATH11K

# Strip legacy audio
scripts/config --disable SND_OSSEMUL --disable SND_ISA
scripts/config --disable SND_SOC_AMD_ACP --disable SND_SOC_QCOM

# Ensure L2 essentials (paranoid — localmodconfig may have unset them)
scripts/config --enable USER_NS --enable SECCOMP_FILTER
scripts/config --enable SECURITY_LANDLOCK --enable BPF_SYSCALL
scripts/config --enable CGROUP_BPF --enable MEMCG

# Strip debug for release kernel
scripts/config --disable DEBUG_INFO --disable DEBUG_KERNEL
scripts/config --disable FTRACE --disable KPROBES

make olddefconfig
make -j$(nproc) bzImage modules
```

Result: ~10 MB compressed bzImage, boots on the target, all peripherals working.

---

## 9. Rules of Thumb

**Rule:** Never strip a symbol you don't understand. "It sounds unrelated" is not a reason — read the help text.

**Rule:** Every new kernel release, re-run `make oldconfig` and audit the new symbols. New symbols default to maintainer-chosen values which may include unwanted behavior.

**Rule:** If you find yourself enabling an out-of-tree driver, file an issue upstream and track it. Out-of-tree = perpetual rebase tax.

**Rule:** Kernel config is a security-critical artifact. Review it in PRs like code, not as a blob.

**Rule:** Don't chase "minimal" as an end in itself. A 14 MB kernel that works on every supported machine beats a 6 MB kernel that breaks when you plug in a USB hub.
