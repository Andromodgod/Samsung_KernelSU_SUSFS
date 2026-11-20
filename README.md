<div align="center">

# ⚡ Normie-Kernel for Samsung (GKI 2.0)

[![Target](https://img.shields.io/badge/Target-SM--S928B%20(One%20UI%208)-blue?style=flat-square)](https://github.com/haroonjee99/Samsung_KernelSU_SUSFS)
[![Kernel](https://img.shields.io/badge/Linux-6.1%20GKI%202.0-lightgrey?style=flat-square)](https://source.android.com/docs/core/architecture/kernel)
[![BakaSU](https://img.shields.io/badge/Root-BakaSU-green?style=flat-square)](https://github.com/Baka-SU/BakaSU)
[![SUSFS](https://img.shields.io/badge/SUSFS-v2.4.0%20Inline-orange?style=flat-square)](https://gitlab.com/simonpunk/susfs4ksu)
[![LTO](https://img.shields.io/badge/Clang%20LTO-Full-red?style=flat-square)](https://llvm.org/)

**Custom KMI-safe Android GKI kernel built with Bazel/Kleaf, featuring BakaSU, SuSFS v2.4.0 inline hooks, NoMount, Baseband-Guard, and low-latency scheduler/I/O tuning.**

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
- **Automated `Kbuild` & `UTS_RELEASE` Patching:** Resolves host `.git` commit counts for `KSU_VERSION`, injects UAPI include paths for `preinit/runner.elf`, synchronizes Kconfig fragments across `common` and `msm-kernel`, and embeds `-Normie-Kernel` directly into `Makefile` (`filechk_kernel.release`) and Kleaf's `stamp.bzl`.
- **Artifact Extraction:** Automatically extracts built `Image`, `Image.gz`, and `Image.lz4` binaries from `bazel-bin` into `AnyKernel3` flashable zips.

---

## 🛠️ Forking & Building for Your Own Variant (One UI 6 / 7 / 8)

You can fork this repository and adapt the CI pipeline to compile custom kernels for other Galaxy models, One UI versions, and Linux kernel branches.

### Step 1: Identify Your Device, One UI Version & Kernel Matrix

Check **Settings → About phone → Software information → Kernel version** on your target firmware to identify your exact Linux GKI version and SoC platform:

| One UI Version | Example Galaxy Devices | SoC Platform | Kernel Version | SuSFS Branch | Build Type (`build_type`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **One UI 8 / 7** | S25 / S25+ / S25 Ultra (`SM-S931*`, `SM-S936*`, `SM-S938*`) | Snapdragon 8 Elite | `android15-6.6` | `gki-android15-6.6` | `Bazel` (`build_kernel_GKI.sh`) |
| **One UI 8 / 7** | Galaxy A56 (`SM-A566*`) | Exynos 1580 (`s5e8855`) | `android15-6.6` | `gki-android15-6.6` | `Bazel` (`//exynos/device/mx/mx8855:mx8855`) |
| **One UI 8 / 7 / 6.1** | S24 / S24+ / S24 Ultra (`SM-S921*`, `SM-S926*`, `SM-S928*`), Z Fold6 / Flip6 | Snapdragon 8 Gen 3 (`pineapple`) | `android14-6.1` | `gki-android14-6.1` | `Bazel` (`build_kernel_GKI.sh`) |
| **One UI 8 / 7 / 6.1** | S24 / S24+ (`SM-S921B`, `SM-S926B`), Galaxy A55 (`SM-A556*`) | Exynos 2400 / 1480 (`s5e9945` / `s5e8845`) | `android14-6.1` | `gki-android14-6.1` | `Bazel` (`//projects/s5e9945:s5e9945_user`) |
| **One UI 7 / 6.1 / 6.0** | S23 / S23+ / S23 Ultra (`SM-S911*`, `SM-S916*`, `SM-S918*`), Z Fold5 / Flip5 | Snapdragon 8 Gen 2 (`kalama`) | `android13-5.15` | `gki-android13-5.15` | `Bazel` (`build_kernel_GKI.sh`) |
| **One UI 7 / 6.1 / 6.0** | S22 Series (`SM-S908*`), Galaxy A54 (`SM-A546*`) | Snapdragon 8 Gen 1 / Exynos 1380 | `android12-5.10` | `gki-android12-5.10` | `Bazel` or `make` |
| **One UI 6.x / 7** | Legacy / Entry Devices (`SM-A055*`, `SM-A146P*`, `SM-X926B*`) | MediaTek / Older Exynos / QCOM | `4.19` / `5.4` / `5.10` (Non-GKI) | Kernel-version matched | `make` (`build_kernel.sh`) |

---

### Step 2: Fork the Repository & Create Your Device Branch

1. **Fork** this repository to your GitHub account.
2. Download the matching open-source kernel package for your exact model and firmware CSC/build from **[Samsung Open Source](https://opensource.samsung.com/)**.
3. Clone your fork locally and create a branch following the repository's naming convention (`<MODEL>-<OneUI>-<KERNEL_VERSION>`):
   ```bash
   git clone [https://github.com/](https://github.com/)<your-username>/Samsung_KernelSU_SUSFS.git
   cd Samsung_KernelSU_SUSFS

   # Example for Galaxy S24 Ultra on One UI 7 (Linux 6.1):
   git checkout -b SM-S928B-Oneui7-android14_6.1

   # Example for Galaxy S23 Ultra on One UI 6.1 (Linux 5.15):
   git checkout -b SM-S918B-Oneui6-android13_5.15
   ```
4. Extract Samsung's kernel source archive into a folder named **identically to your branch** inside the repository root (so `$CONFIG` resolves to `$BRANCH`):
   ```bash
   mkdir -p SM-S928B-Oneui7-android14_6.1
   tar -xzf Kernel.tar.gz -C SM-S928B-Oneui7-android14_6.1/
   git add .
   git commit -m "Add SM-S928B One UI 7 kernel source"
   git push -u origin SM-S928B-Oneui7-android14_6.1
   ```

---

### Step 3: Configure `.github/workflows/main.yml` for Your Variant

1. **Add Your Branch to the Workflow Dropdown:**
   In `.github/workflows/main.yml`, add your new branch name under `on.workflow_dispatch.inputs`:
   ```yaml
   branch:
     description: 'Target branch / kernel source folder'
     required: true
     default: 'SM-S928B-Oneui7-android14_6.1'
     type: choice
     options:
       - SM-S928B-Oneui8-android14_6.1
       - SM-S928B-Oneui7-android14_6.1
       - SM-S918B-Oneui6-android13_5.15
   ```
2. **Match the SuSFS Branch to Your Kernel Version:**
   In `.github/workflows/main.yml`, update the `susfs4ksu` git clone branch to match your target Linux kernel version:
   - `6.6` kernels (One UI 7 / 8 on S25, A56) → `gki-android15-6.6`
   - `6.1` kernels (One UI 6.1 / 7 / 8 on S24, Z Fold6, A55) → `gki-android14-6.1`
   - `5.15` kernels (One UI 6.0 / 6.1 / 7 on S23, Z Fold5) → `gki-android13-5.15`
   - `5.10` kernels (One UI 6.0 / 6.1 / 7 on S22, A54) → `gki-android12-5.10`
3. **Set Kernel Version & Build Type Variables:**
   Ensure `KERNEL_VER` (`6.6`, `6.1`, `5.15`, `5.10`, or `5.4`) and `build_type` (`Bazel` for GKI 2.0 or `make` for legacy non-GKI kernels) match your device row from Step 1.

---

### Step 4: Configure Kernel-Specific Rules in `.github/actions/build/action.yml`

Depending on your One UI version and chipset, verify these four areas in `.github/actions/build/action.yml`:

1. **Chipset / Bazel Target Routing:**
   - **Qualcomm GKI (`6.1` / `6.6`):** Handled automatically by `elif [[ $KERNEL_VER == 6.* ]]`, which runs `./build_kernel_GKI.sh` with `LTO=$EFFECTIVE_LTO`.
   - **Qualcomm GKI `5.15` (One UI 6 / 7 on S23 Series `SM-S911*` / `SM-S916*` / `SM-S918*`):** Add a branch condition in `start bazel build` so it invokes `build_kernel_GKI.sh`:
     ```bash
     elif [[ $BRANCH == SM-S911* || $BRANCH == SM-S916* || $BRANCH == SM-S918* ]]; then
       chmod +x build_kernel_GKI.sh
       env $SCRIPT_LTO ./build_kernel_GKI.sh || true
     ```
   - **Exynos Bazel Targets:** Match or add your SoC's Bazel target in `start bazel build`:
     - Exynos 2400 (`SM-S921B`, `SM-S926B`) → `//projects/s5e9945:s5e9945_user`
     - Exynos 1480 (`SM-A556*`) → `//projects/s5e8845:s5e8845_user`
     - Exynos 1580 (`SM-A566*`) → `//exynos/device/mx/mx8855:mx8855`
   - **Legacy Non-GKI (`4.19` / `5.4`):** Set `build_type: make` in `.github/workflows/main.yml` so CI executes the `start make build` step (`./build_kernel.sh`).

2. **Version-Specific Kconfig & KMI Compatibility (`5.10` / `5.15` vs `6.1` / `6.6`):**
   - **16ms PELT Half-Life (`kernel/sched/pelt.h` & `pelt.c`):** Works identically across `5.10`, `5.15`, `6.1`, and `6.6`.
   - **MGLRU (`CONFIG_LRU_GEN_ENABLED=y`):** Native in `6.1` and `6.6`. On `5.15` and `5.10`, MGLRU only activates if Samsung backported it into your kernel tree; otherwise Kconfig safely ignores it.
   - **KMI Symbol Whitelist (`abi_gki_aarch64`):** On GKI 2.0 devices (`5.10`–`6.6`), stock vendor `.ko` modules on `vendor_dlkm` are compiled against Google's KMI. Keep `BUILD_LTO=full` (or `thin` to match stock) and avoid modifying exported kernel structs so Wi-Fi, camera, and audio vendor modules load cleanly.
   - **`check_merged_defconfig` Strictness:** Bazel's `KernelConfig` step fails if an unknown or parent-disabled Kconfig symbol is injected into `gki_defconfig`. If an older `5.10` or `5.15` tree lacks a newer driver symbol (such as `CONFIG_HID_PLAYSTATION` or `CONFIG_F2FS_FS_ZSTD`), remove that symbol from `base_config_lines` in `.github/actions/build/action.yml`.

3. **Customize Your Kernel Branding (Optional):**
   To replace `-Normie-Kernel` with your own name in **Settings → About phone → Software information**, update all four occurrences in `.github/actions/build/action.yml`:
   - `CONFIG_LOCALVERSION="-Your-Kernel"`
   - Top-level `Makefile` (`filechk_kernel.release` suffix)
   - `scripts/setlocalversion` suffix
   - Kleaf `build/kernel/kleaf/impl/stamp.bzl` (`scmversion` suffix)

---

### Step 5: Trigger GitHub Actions & Package

1. Open the **Actions** tab in your forked GitHub repository and select the build workflow.
2. Click **Run workflow**.
3. Choose your target branch (`SM-XXXX-OneuiX-...`), select `BUILD_LTO` (`full`, `thin`, or `none`), and click **Run workflow**.
4. Once all steps finish, download the generated `*-AnyKernel3.zip` from **GitHub Releases** or the workflow run's **Artifacts** section.

---

## 🎛️ Workflow Dispatch Inputs Reference

| Input / Env Variable | Options / Example | Description |
| :--- | :--- | :--- |
| `branch` | `SM-S928B-Oneui8-android14_6.1` | Target device/One UI branch folder to build. |
| `BUILD_LTO` | `full` *(default)*, `thin`, `none` | Controls Clang Link-Time Optimization (`--lto=full` / `LTO=full`). |
| `build_type` | `Bazel`, `make` | Selects Kleaf/Bazel GKI 2.0 build vs legacy `make` build. |
| `ccache` | `true`, `false` | Enables compiler caching where supported. |

---

## 🧩 Troubleshooting & Build Notes

- **Kleaf `module_outs` Packaging Error (`tcp_bic.ko`, `tcp_westwood.ko`, `tcp_htcp.ko`):** Enabling `CONFIG_TCP_CONG_ADVANCED=y` exposes legacy TCP congestion modules whose Kconfig defaults are `=m`. Because Kleaf enforces a strict `module_outs` allowlist in `BUILD.bazel`, `.github/actions/build/action.yml` explicitly disables `# CONFIG_TCP_CONG_BIC/WESTWOOD/HTCP is not set` while keeping `CONFIG_TCP_CONG_BBR=y` built-in.
- **Custom Kernel Name Not Showing in About Phone:**
  1. Android 14/15 Kleaf builds set `--kleaf_localversion=true` by default, ignoring `CONFIG_LOCALVERSION` and `scripts/setlocalversion`. `action.yml` patches both `Makefile` (`filechk_kernel.release`) and `build/kernel/kleaf/impl/stamp.bzl` so `-Normie-Kernel` is embedded directly into `UTS_RELEASE`.
  2. If you use **SuSFS (`CONFIG_KSU_SUSFS_SPOOF_UNAME=y`)**, ensure your SuSFS userspace module configuration does not spoof `uname` back to the stock version string if you want `-Normie-Kernel` visible in **Settings → About phone**.
- **Stock Build Timestamp (`KBUILD_BUILD_TIMESTAMP`):** Samsung's `build_kernel_GKI.sh` hardcodes `KBUILD_BUILD_TIMESTAMP` to match the stock firmware build date. Seeing the stock date next to `-Normie-Kernel` is expected and does not indicate an old image.

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
```

---

## 🌟 Credits & Upstream Projects

Built on top of open-source contributions from the Android kernel community:

| Project / Contributor | Role | Link |
| :--- | :--- | :--- |
| **Wild Kernels (`Jimsterino98`)** | Upstream Samsung GKI CI & patch framework | [![GitHub](https://img.shields.io/badge/GitHub-WildKernels-blue?style=flat-square&logo=github)](https://github.com/WildKernels/Samsung_KernelSU_SUSFS) |
| **BakaSU (`Baka-SU`)** | Kernel-mode root solution | [![GitHub](https://img.shields.io/badge/GitHub-Baka--SU-blue?style=flat-square&logo=github)](https://github.com/Baka-SU/BakaSU) |
| **SUSFS (`simonpunk`)** | Kernel-level root hiding subsystem | [![GitLab](https://img.shields.io/badge/GitLab-simonpunk-orange?style=flat-square&logo=gitlab)](https://gitlab.com/simonpunk/susfs4ksu) |
| **SUSFS Module (`sidex15`)** | Userspace companion module | [![GitHub](https://img.shields.io/badge/GitHub-sidex15-blue?style=flat-square&logo=github)](https://github.com/sidex15) |
| **NoMount (`maxsteeel`)** | Mountless namespace isolation | [![GitHub](https://img.shields.io/badge/GitHub-maxsteeel-blue?style=flat-square&logo=github)](https://github.com/maxsteeel/nomount) |
| **Baseband-Guard (`vc-teahouse`)** | Baseband & partition protection | [![GitHub](https://img.shields.io/badge/GitHub-vc--teahouse-blue?style=flat-square&logo=github)](https://github.com/vc-teahouse/Baseband-guard) |
| **DroidSpaces (`ravindu644`)** | Linux container support for Android | [![GitHub](https://img.shields.io/badge/GitHub-ravindu644-blue?style=flat-square&logo=github)](https://github.com/ravindu644/Droidspaces-OSS) |
| **KernelSU (`tiann`)** | Core KernelSU architecture | [![GitHub](https://img.shields.io/badge/GitHub-tiann-blue?style=flat-square&logo=github)](https://github.com/tiann/KernelSU) |
| **Community Contributors** | `backslashxx`, `Teemo` (`liqideqq`), `kerneltoast` | Patch & GKI contributions |
