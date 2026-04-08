# Samsung Galaxy S24 FE - Custom Kernel

Custom kernel builds for the Samsung Galaxy S24 FE (Exynos 2500) with KernelSU root support.

- **SoC:** Exynos 2500 (s5e9945)
- **Android:** 16
- **Kernel:** 6.1.138-android14-11
- **Firmware:** S721BXXS7CYJ5

## Compatible Models

Per [Samsung Open Source](https://opensource.samsung.com/uploadSearch?searchValue=s721b), this kernel is compatible with all S24 FE variants:

| Model | Region |
|-------|--------|
| SM-S721B | Global |
| SM-S721U | USA |
| SM-S721U1 | USA (Unlocked) |
| SM-S7210 | China |
| SM-S721W | Canada |
| SM-S721Q | Latin America |
| SM-S721J | Japan |

## Branches

| Branch | Description | KernelSU Version | Status |
|--------|-------------|------------------|--------|
| [`stock`](../../tree/stock) | Unmodified Samsung kernel source | None | Baseline |
| [`v0.9.5`](../../tree/v0.9.5) | KernelSU v0.9.5 built into kernel | 11872 | Unmaintained |
| [`v3.x`](../../tree/v3.x) | Latest KernelSU (v3.2.4) built into kernel | 32457 | **Recommended** |

### `stock`

Clean Samsung kernel source extracted from the official open source release. No modifications. Use this as a reference or starting point for your own patches.

### `v0.9.5`

KernelSU v0.9.5 — the last version that supported non-GKI built-in integration with manual VFS hooks. Uses LSM hooks via `security_add_hooks()`. This version is no longer maintained upstream but is stable and functional.

### `v3.x`

Latest KernelSU (v3.2.4) compiled as a built-in kernel component. Despite KernelSU officially "dropping non-GKI support" in v1.0+, the kernel code still fully supports `CONFIG_KSU=y` (built-in compilation) via `tristate` Kconfig. v3.x uses runtime syscall table + LSM hook patching — no manual kernel source hooks are required. This is the recommended branch.

## Common Modifications (v0.9.5 and v3.x)

Both KernelSU branches include:

- **RKP/KDP disabled** — Samsung's hypervisor-level kernel protection
- **FIVE/PROCA/DEFEX disabled** — Samsung integrity verification
- **Module signature bypass** — runtime patches to allow unsigned kernel modules (required for Wi-Fi)
- **Bazel build fixes** — adjusted for building without full platform prebuilts

## Quick Start

See the README in each branch for detailed build instructions. The general flow is:

1. Sync the AOSP kernel toolchain (`common-android14-6.1`)
2. Clone the desired branch
3. Copy toolchain directories into the source tree
4. Build with bazel
5. Replace the kernel in your stock `boot.img` using `magiskboot`
6. Flash via Odin

## Related

- [KernelSU](https://github.com/tiann/KernelSU)
- [Samsung Open Source](https://opensource.samsung.com)
- [XDA Build Guide](https://xdaforums.com/t/compile-s24-fe-stock-kernel-s721b.4772301/)
