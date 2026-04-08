# Samsung Galaxy S24 FE (SM-S721B) - KernelSU v0.9.5

Custom kernel with [KernelSU](https://github.com/tiann/KernelSU) v0.9.5 built-in for the Samsung Galaxy S24 FE (Exynos).

- **Device:** Samsung Galaxy S24 FE (SM-S721B)
- **SoC:** Exynos 2500 (s5e9945)
- **Android:** 16
- **Kernel:** 6.1.138-android14-11
- **Build:** S721BXXS7CYJ5
- **KernelSU:** v0.9.5 (version code 11872)

## What's Modified

Compared to the `stock` branch:

### KernelSU Integration
- KernelSU v0.9.5 source added to `kernel/drivers/kernelsu/`
- `kernel/drivers/Makefile` and `kernel/drivers/Kconfig` updated to include KernelSU
- `CONFIG_KSU=y` added to defconfig (compiled as built-in, not a module)

### Security Modifications (required for KernelSU to function)
- **RKP/KDP disabled** in `kernel/arch/arm64/configs/s5e9945-bazel_defconfig` — Samsung's hypervisor-level kernel protection conflicts with KernelSU's privilege escalation
- **FIVE/PROCA/DEFEX disabled** — Samsung integrity verification frameworks
- **Module signature bypass** — runtime patches in:
  - `kernel/kernel/module/version.c` — `check_version()` returns 1
  - `kernel/kernel/module/main.c` — `check_modinfo()` returns 0
  - `kernel/kernel/module/signing.c` — `mod_verify_sig()` returns 0, `module_sig_check()` returns 0
  - `kernel/security/security.c` — `security_kernel_read_file()` and `security_kernel_load_data()` return 0
- `CONFIG_MODULE_FORCE_LOAD=y` added to defconfig

### Build Fixes
- `projects/s5e9945/s5e9945.bzl` — vendor ramdisk and vendor files entries adjusted for building without full platform prebuilts

## How to Build

### 1. Get the Toolchain

```bash
mkdir toolchain && cd toolchain
repo init -u https://android.googlesource.com/kernel/manifest -b common-android14-6.1 --depth=1
repo sync -c -j$(nproc) --no-tags
cd ..
```

### 2. Clone This Repo

```bash
git clone -b v0.9.5 https://github.com/Sir-MmD/s24-fe.git
cd s24-fe
```

### 3. Copy Toolchain Into Source Tree

```bash
cp -rf ../toolchain/build .
cp -rf ../toolchain/external .
cp -rf ../toolchain/prebuilts .
cp -rf ../toolchain/tools .
```

### 4. Fix Bazel Visibility

```bash
find build/kernel/kleaf/impl -name "*.bzl" -exec sed -i 's|visibility("//build/kernel/kleaf/...")|# visibility("//build/kernel/kleaf/...")|g' {} +
```

### 5. Build

```bash
tools/bazel run --nocheck_bzl_visibility --config=stamp --sandbox_debug --verbose_failures --debug_make_verbosity=I //projects/s5e9945:s5e9945_user_dist
```

Output will be in `out/s5e9945_user/dist/`.

### 6. Package Boot Image

Get the stock `boot.img` from your firmware (extract `boot.img.lz4` from the AP tar and decompress with `lz4 -d`).

```bash
magiskboot unpack stock_boot.img
cp out/s5e9945_user/dist/Image kernel
magiskboot repack stock_boot.img new_boot.img
lz4 -B6 --content-size new_boot.img boot.img.lz4
tar -cf kernelsu_boot.tar boot.img.lz4
```

### 7. Flash

Reboot to download mode and flash with Odin:

```bash
adb reboot download
sudo odin4 -a kernelsu_boot.tar
```

### 8. Install KernelSU Manager

After booting, install the matching Manager APK (v0.9.5):

```bash
# Download from: https://github.com/tiann/KernelSU/releases/tag/v0.9.5
adb install KernelSU_v0.9.5_11872-release.apk
```

## Notes

- KernelSU v0.9.5 is the last version that supports non-GKI built-in compilation with manual kernel hooks
- v0.9.5 uses LSM hooks via `security_add_hooks()` for its core functionality
- This version is no longer maintained — consider using the `v3.x` branch for the latest KernelSU
- Knox is tripped (bootloader unlocked required)
- Wi-Fi works with the module signature bypass patches applied
