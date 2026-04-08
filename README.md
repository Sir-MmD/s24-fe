# Samsung Galaxy S24 FE (SM-S721B) - KernelSU v3.x

Custom kernel with the latest [KernelSU](https://github.com/tiann/KernelSU) (v3.2.4) built-in for the Samsung Galaxy S24 FE (Exynos).

- **Device:** Samsung Galaxy S24 FE (SM-S721B)
- **SoC:** Exynos 2500 (s5e9945)
- **Android:** 16
- **Kernel:** 6.1.138-android14-11
- **Build:** S721BXXS7CYJ5
- **KernelSU:** v3.2.4 (version code 32457)

## What's Modified

Compared to the `stock` branch:

### KernelSU Integration
- KernelSU v3.x source added to `kernel/drivers/kernelsu/`
- `kernel/drivers/Makefile` and `kernel/drivers/Kconfig` updated to include KernelSU
- `CONFIG_KSU=y` added to defconfig (compiled as built-in)
- `CONFIG_KPROBES=y` added to defconfig (required dependency for v3.x)
- Version code hardcoded to 32457 in `kernel/drivers/kernelsu/Kbuild` (bazel sandbox doesn't preserve git history)

### How v3.x Differs from v0.9.5
- **No manual kernel source patches needed** — v3.x hooks everything at runtime via syscall table patching, LSM hook patching, and tracepoints
- **Runtime symbol resolution** via `kallsyms_lookup_name()` — no need to export symbols manually
- **Syscall tracepoints** for per-process interception via `TIF_SYSCALL_TRACEPOINT`
- **Active development** — receives security fixes and new features
- Manager shows "Working (GKI)" — this is correct, it means "built into kernel" (as opposed to LKM mode)

### Security Modifications (required for KernelSU to function)
- **RKP/KDP disabled** in defconfig — Samsung's hypervisor-level kernel protection conflicts with KernelSU's syscall table and LSM hook patching
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
git clone -b v3.x https://github.com/Sir-MmD/s24-fe.git
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

After booting, install the latest Manager APK:

```bash
# Download from: https://github.com/tiann/KernelSU/releases/latest
adb install KernelSU_v3.2.4_32457-release.apk
```

## Notes

- KernelSU v3.x uses runtime hooking (syscall table + LSM patches) — no manual kernel source hooks required
- Despite KernelSU officially "dropping non-GKI support" in v1.0+, the kernel code still fully supports `CONFIG_KSU=y` (built-in) via `tristate` Kconfig and proper `#ifdef MODULE` guards
- The Manager displays "Working (GKI)" which means "built into kernel" — this is correct
- Knox is tripped (bootloader unlocked required)
- Wi-Fi works with the module signature bypass patches applied
- This is the recommended branch for most users
