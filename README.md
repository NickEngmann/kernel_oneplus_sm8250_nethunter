# NetHunter Kernel for OnePlus 8T (Nameless AOSP)

Custom kernel with MAC80211 wireless stack, WireGuard, and pentest tools support for running [Kali NetHunter](https://www.kali.org/docs/nethunter/) on a OnePlus 8T with Nameless AOSP.

## What This Is

A patched build of the [Nameless AOSP kernel](https://github.com/Nameless-AOSP-OSS/kernel_oneplus_sm8250) (branch `thirteen`, kernel 4.19.297) with:

- **MAC80211** — wireless stack for external USB WiFi adapters (monitor mode, packet injection)
- **WireGuard** — kernel-level VPN
- **nftables** — modern netfilter framework
- **USB Bluetooth** — external BT adapter support
- **USB serial** — CH341, PL2303, CP210X adapters for hardware hacking
- **Module signing & MODVERSIONS disabled** — load any `.ko` module without hassle

All stock vendor modules preserved: internal WiFi (qcacld), GPU (KGSL/Adreno 650), camera, display, audio, USB.

## Device Support

| Device | SoC | Codename | Status |
|--------|-----|----------|--------|
| OnePlus 8T | Snapdragon 865 | kebab | **Tested, boots** |
| OnePlus 8 | Snapdragon 865 | instantnoodle | Should work (same SoC, untested) |
| OnePlus 8 Pro | Snapdragon 865 | instantnoodlep | Should work (same SoC, untested) |

**ROM:** Nameless AOSP Android 12. Will NOT boot on crDroid, OxygenOS, or other ROMs — kernel vendor modules are ROM-specific.

## Quick Start

### Pre-built (flash and go)
Download the latest release zip and flash via ADB sideload in recovery:
```bash
adb sideload nethunter-nameless-v4.zip
```

### Build from source
```bash
# Install deps
sudo apt install gcc-aarch64-linux-gnu build-essential libssl-dev bc flex bison libelf-dev

# Get Clang 12.0.7 (must be this exact version)
wget https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/+archive/refs/heads/android12-release/clang-r416183b1.tar.gz
mkdir clang && tar xf clang-r416183b1.tar.gz -C clang

# Build
export ARCH=arm64 PATH=$(pwd)/clang/bin:$PATH
make CC=clang LLVM=1 LLVM_IAS=1 CROSS_COMPILE=aarch64-linux-gnu- vendor/kona-perf_defconfig
make -j$(nproc) CC=clang LLVM=1 LLVM_IAS=1 CROSS_COMPILE=aarch64-linux-gnu- \
    AR=llvm-ar LD=aarch64-linux-gnu-ld OBJDUMP=llvm-objdump STRIP=llvm-strip \
    Image.gz-dtb
```

Output: `arch/arm64/boot/Image.gz-dtb` — package with [AnyKernel3](https://github.com/osm0sis/AnyKernel3) and flash.

## USB WiFi Adapters

| Adapter | Chipset | Driver | Status |
|---------|---------|--------|--------|
| Edimax N150 (7392:b811) | RTL8188EUS | 8188eu.ko | **Working** — monitor mode confirmed |
| Edimax AC600 (7392:d811) | RTL8821CU? | 8821cu.ko | Testing — interface creates, radio investigation ongoing |
| Ralink (148f:5370) | RT5370 | rt2800usb | Not yet built |

Build out-of-tree modules:
```bash
# Example: rtl8188eus
git clone --depth 1 https://github.com/aircrack-ng/rtl8188eus.git
cd rtl8188eus
make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=clang \
    KSRC=/path/to/this/kernel modules
# Push to phone: adb push 8188eu.ko /sdcard/
# Load: insmod /sdcard/8188eu.ko
```

## Build Notes

**Must use Clang 12.0.7** (`r416183b1`) — the exact compiler Nameless AOSP uses. GCC won't work. Newer Clang versions may also have issues.

**Must use GNU ld** (`LD=aarch64-linux-gnu-ld`) — the LLVM linker (`ld.lld`) has MODVERSIONS bugs on 4.19 kernels.

See [`docs/kernel-build-debrief.md`](docs/kernel-build-debrief.md) for the full story of what we tried and why things failed.

## Documentation

- [`CLAUDE.md`](CLAUDE.md) — Build reference for AI assistants
- [`docs/kernel-configs.md`](docs/kernel-configs.md) — Every config change with purpose
- [`docs/kernel-build-debrief.md`](docs/kernel-build-debrief.md) — Build history and lessons learned
- [`docs/usb-wifi-adapters.md`](docs/usb-wifi-adapters.md) — Adapter compatibility
- [`docs/recovery.md`](docs/recovery.md) — How to recover from bootloops

## Recovery

If the phone bootloops after flashing, use Linux (not Windows) with fastboot:
```bash
sudo fastboot flash boot_b boot-backup.img
sudo fastboot reboot
```

See [`docs/recovery.md`](docs/recovery.md) for full instructions.

## Related Projects

- [Nightcrawler](https://github.com/NickEngmann/nightcrawler) — Autonomous pentest agent running on this phone
- [Nameless AOSP kernel](https://github.com/Nameless-AOSP-OSS/kernel_oneplus_sm8250) — Upstream kernel source

## License

GPL-2.0 (same as upstream Linux kernel)
