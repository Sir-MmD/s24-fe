# Samsung Galaxy S24 FE (SM-S721B) - Stock Kernel Source

Unmodified Samsung kernel source for SM-S721B (Exynos 2500 / s5e9945).

- **Device:** Samsung Galaxy S24 FE (SM-S721B)
- **SoC:** Exynos 2500 (s5e9945)
- **Android:** 16
- **Kernel:** 6.1.138-android14-11
- **Build:** S721BXXS7CYJ5
- **Source:** [Samsung Open Source](https://opensource.samsung.com)

## How to Build

### 1. Get the Toolchain

```bash
mkdir toolchain && cd toolchain
repo init -u https://android.googlesource.com/kernel/manifest -b common-android14-6.1 --depth=1
repo sync -c -j$(nproc) --no-tags
cd ..
```

### 2. Extract and Prepare

Clone this repo, then copy toolchain directories into it (overwriting existing files):

```bash
cp -rf toolchain/build .
cp -rf toolchain/external .
cp -rf toolchain/prebuilts .
cp -rf toolchain/tools .
```

### 3. Fix Bazel Build

Create missing prebuilt boot-artifacts (if not already present):

```bash
mkdir -p prebuilts/boot-artifacts/arm64/exynos
echo 'exports_files(glob(["**/*"]))' > prebuilts/boot-artifacts/arm64/exynos/BUILD
touch prebuilts/boot-artifacts/arm64/exynos/file_contexts
```

Fix Bazel visibility errors:

```bash
find build/kernel/kleaf/impl -name "*.bzl" -exec sed -i 's|visibility("//build/kernel/kleaf/...")|# visibility("//build/kernel/kleaf/...")|g' {} +
```

Edit `projects/s5e9945/s5e9945.bzl`:

- Comment out all entries in `_vendor_files` srcs
- Swap vendor ramdisk: uncomment `_custom_vendor_ramdisk`, comment out the `prebuilts` ramdisk line

### 4. Build

```bash
tools/bazel run --nocheck_bzl_visibility --config=stamp --sandbox_debug --verbose_failures --debug_make_verbosity=I //projects/s5e9945:s5e9945_user_dist
```

### 5. Output

Output files are in `out/s5e9945_user/dist/`. The compiled kernel `Image` can be found there along with `boot.img`.

### 6. Flash

Use `magiskboot` to replace the kernel in your stock `boot.img`:

```bash
magiskboot unpack stock_boot.img
cp out/s5e9945_user/dist/Image kernel
magiskboot repack stock_boot.img new_boot.img
```

Compress and flash via Odin:

```bash
lz4 -B6 --content-size new_boot.img boot.img.lz4
tar -cf custom_boot.tar boot.img.lz4
sudo odin4 -a custom_boot.tar
```
