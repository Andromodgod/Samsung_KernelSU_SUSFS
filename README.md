<div align="center">

# ⚡ Normie-Kernel for Samsung (GKI 2.0)

[![Target](https://img.shields.io/badge/Target-SM--S928B%20(One%20UI%208)-blue?style=flat-square)](https://github.com/haroonjee99/Samsung_KernelSU_SUSFS)
[![Kernel](https://img.shields.io/badge/Linux-6.1%20GKI%202.0-lightgrey?style=flat-square)](https://source.android.com/docs/core/architecture/kernel)
[![BakaSU](https://img.shields.io/badge/Root-BakaSU-green?style=flat-square)](https://github.com/Baka-SU/BakaSU)
[![SUSFS](https://img.shields.io/badge/SUSFS-v2.4.0%20Inline-orange?style=flat-square)](https://gitlab.com/simonpunk/susfs4ksu)
[![LTO](https://img.shields.io/badge/Clang%20LTO-Full-red?style=flat-square)](https://llvm.org/)

**Custom KMI-safe Android 14 (`6.1`) GKI kernel built with Bazel/Kleaf, featuring BakaSU, SuSFS v2.4.0 inline hooks, NoMount, Baseband-Guard, and low-latency scheduler/I/O tuning.**

</div>

---

<div align="center">

## ⚠️ Disclaimer & Warranty Notice

**Your warranty is no longer valid once you unlock the bootloader or flash custom kernels.**

This software is provided for testing and educational purposes only. The maintainers are **not responsible** for bricked devices, tripped Knox (`0x1`), damaged hardware, or data loss. Back up your `boot.img` and critical data before flashing.

**🚨 Proceed at your own risk!**

</div>

---

## ✨ Key Features

### 🔐 Root, Security & Stealth Hardening
- **BakaSU (`CONFIG_KSU`):** Kernel-mode root solution supporting **KernelSU Official**, **KOWSU**, and **BakaSU** managers out of the box.
- **SuSFS v2.4.0 (Inline Hooks):** Latest root-hiding subsystem using inline hooks (`setresuid`, `execveat`, `faccessat`, `sys_read`, `stat`, `vfs_fstat`, `sys_reboot`, `input_handle_event`) and `selinux_hide` manual hooks.
- **NoMount:** Mountless namespace isolation for module overlays without leaving detectable mount table artifacts.
- **Baseband-Guard (BBG):** Hardware-level baseband/partition write protection integrated into `CONFIG_LSM`.
- **Kernel Patch Module (KPM) Support:** Dynamic runtime kernel patching support.
- **Unicode Path Traversal Fix:** Blocks detection vectors relying on non-printable Unicode codepoints in paths.
- **`/proc/config.gz` Stealth Masking:** Filters build-time `ikconfig` output so `CONFIG_KSU`, `CONFIG_KPM`, `CONFIG_BBG`, `CONFIG_TMPFS_XATTR`, `CONFIG_IP6_NF_NAT`, `CONFIG_WIREGUARD`, and `CONFIG_DEFAULT_BBR` report as `# ... is not set` while preserving a valid gzip stream.

### 🚀 Compiler, Scheduler & Storage Optimizations (KMI-Safe)
- **Full Clang LTO (`LTO=full`):** Cross-translation-unit link-time optimization enabled by default for improved IPC and smaller binary footprint without breaking vendor `.ko` KMI compatibility.
- **300Hz Kernel Timer (`CONFIG_HZ_300=y`):** Aligns scheduler timer ticks evenly with `60Hz` / `120Hz` LTPO display refresh intervals for tighter touch and frame pacing.
- **16ms PELT Half-Life:** Halves Per-Entity Load Tracking decay windows (`LOAD_AVG_PERIOD=16`) with recalculated decay tables so WALT/EAS ramps CPU frequencies twice as fast during UI and gaming bursts.
- **MGLRU Enabled at Boot (`CONFIG_LRU_GEN_ENABLED=y`):** Multi-Gen LRU active immediately without userspace scripts, cutting `kswapd` overhead and improving background app retention.
- **F2FS Zstd & LZ4/LZ4HC Compression:** Built-in `CONFIG_F2FS_FS_ZSTD` and `CONFIG_CRYPTO_ZSTD` for higher UFS 4.0 random-read throughput and reduced NAND write amplification.

### 🎮 Gaming, Containers & USB Hardware
- **NTSYNC (`CONFIG_NTSYNC`):** In-kernel Windows NT synchronization primitives for low-latency Wine / Winlator gaming.
- **DroidSpaces Support:** Full Linux container and namespace isolation support ([Droidspaces-OSS](https://github.com/ravindu644/Droidspaces-OSS)).
- **Built-in Gamepad & Controller Drivers (`=y`):** Native plug-and-play USB/Bluetooth support with force feedback and LEDs for **Xbox (`xpad`)**, **PlayStation DualSense / DualShock 4 (`hid-playstation`, `hid-sony`)**, **Nintendo Switch Pro / Joy-Con (`hid-nintendo`)**, **Steam Controller (`hid-steam`)**, and **Microsoft HID** controllers.
- **USB Gadget HID & Mass Storage (`=y`):** Built-in `usb_f_hid` and `usb_f_mass_storage` for USB keyboard/mouse emulation and bootable `.iso` mounting over USB-C.

### 🌐 Networking & Firewall Stack
- **TCP BBRv3 (Default at Boot):** Active immediately via `CONFIG_DEFAULT_BBR=y` for higher throughput and lower latency on mobile and Wi-Fi links.
- **FQ & CAKE Packet Schedulers:** `fq` enabled as default pacing qdisc (`CONFIG_DEFAULT_FQ=y`) alongside built-in `sch_cake` and `sch_pie` for bufferbloat control.
- **Built-in WireGuard (`CONFIG_WIREGUARD=y`):** Zero module-loading overhead for VPN tunnels.
- **IPv6 NAT, IP Set & TTL Target Support:** Full netfilter support for advanced firewall rules and packet TTL/HL modification.

---

## ⚙️ Automated Bazel / Kleaf CI Pipeline

This repository includes custom GitHub Actions automation (`.github/actions/build/action.yml` and `.github/workflows/main.yml`) tailored for Samsung's Bazel/Kleaf GKI 2.0 build system:
- **Sandbox Materialization:** Converts out-of-tree symlinks (`drivers/kernelsu`, `fs/nomount`, `security/baseband-guard`) and UAPI headers into real files compatible with Bazel's `processwrapper-sandbox`.
- **Automated `Kbuild` & `UTS_RELEASE` Patching:** Resolves host `.git` commit counts for `KSU_VERSION`, injects UAPI include paths for `preinit/runner.elf`, synchronizes Kconfig fragments across `common` and `msm-kernel`, and appends `-Normie-Kernel` to the kernel version string.
- **Artifact Extraction:** Automatically extracts built `Image`, `Image.gz`, and `Image.lz4` binaries from `bazel-bin` into `AnyKernel3` flashable zips.

---

## 📋 Installation & Post-Flash Verification

### Flashing
1. Download the latest `*-AnyKernel3.zip` from **GitHub Releases** (or GitHub Actions workflow artifacts).
2. Flash the zip using a compatible kernel flasher such as **[KernelFlasher](https://github.com/fatalcoder524/KernelFlasher)** or **[Horizon Kernel Flasher](https://github.com/libxzr/HorizonKernelFlasher)**.
3. Reboot and verify the active kernel version under **Settings → About phone → Software information → Kernel version** (`...-Normie-Kernel`).

### Quick `adb shell` Verification
```bash
# Confirm Normie-Kernel branding
uname -r

# Confirm BBR congestion control & FQ default qdisc
sysctl net.ipv4.tcp_congestion_control net.core.default_qdisc

# Confirm MGLRU is active at boot (0x0007)
cat /sys/kernel/mm/lru_gen/enabled

# Confirm /proc/config.gz stealth masking and 300Hz timer
zcat /proc/config.gz | grep -E "CONFIG_HZ=|CONFIG_KSU|CONFIG_KPM|CONFIG_BBG"
