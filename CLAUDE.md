# NetHunter Kernel for OnePlus 8T (Nameless AOSP)

## Device
- OnePlus 8T (kebab), Snapdragon 865 (sm8250/kona)
- ROM: Nameless AOSP, Android 12
- Kernel: 4.19.297 (branch: thirteen)

## Source
- Upstream: https://github.com/Nameless-AOSP-OSS/kernel_oneplus_sm8250 (branch: thirteen)
- Fork: https://github.com/NickEngmann/kernel_oneplus_sm8250_nethunter

## Build Requirements
- Android Clang r416183b1 (12.0.7) — exact version used by Nameless AOSP
  - Download: `wget https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/+archive/refs/heads/android12-release/clang-r416183b1.tar.gz`
  - Extract to: `/home/cyeng/kernel-build/clang/`
- GCC cross-compiler: `sudo apt install gcc-aarch64-linux-gnu`
- Build tools: `sudo apt install build-essential libssl-dev bc flex bison libelf-dev python3 cpio zip`

## Quick Build
```bash
export ARCH=arm64
export PATH=/home/cyeng/kernel-build/clang/bin:$PATH

make CC=clang LLVM=1 LLVM_IAS=1 CROSS_COMPILE=aarch64-linux-gnu- vendor/kona-perf_defconfig

# Enable NetHunter configs
scripts/config --file .config \
    -e CONFIG_MAC80211 -e CONFIG_MAC80211_MESH -e CONFIG_MAC80211_LEDS \
    -e CONFIG_WIRELESS_EXT -e CONFIG_WEXT_CORE -e CONFIG_WEXT_PROC -e CONFIG_WEXT_PRIV \
    -e CONFIG_CFG80211_WEXT -e CONFIG_88XXAU \
    -e CONFIG_BT_HCIBTUSB -e CONFIG_USB_SERIAL_CH341 \
    -e CONFIG_USB_RTL8150 -e CONFIG_USB_NET_CDC_EEM -e CONFIG_PACKET_DIAG \
    -e CONFIG_WIREGUARD -e CONFIG_NF_TABLES \
    -e CONFIG_USB_SERIAL_PL2303 -e CONFIG_NETLINK_DIAG -e CONFIG_PPP_ASYNC \
    -m CONFIG_USB_SERIAL_OPTION \
    -d CONFIG_MODULE_SIG -d CONFIG_MODVERSIONS

make CC=clang LLVM=1 LLVM_IAS=1 CROSS_COMPILE=aarch64-linux-gnu- olddefconfig

make -j$(nproc) CC=clang LLVM=1 LLVM_IAS=1 CROSS_COMPILE=aarch64-linux-gnu- \
    AR=llvm-ar LD=aarch64-linux-gnu-ld OBJDUMP=llvm-objdump STRIP=llvm-strip \
    Image.gz-dtb
```

Output: `arch/arm64/boot/Image.gz-dtb`

## Building Kernel Modules (out-of-tree)

### rtl8812au (Edimax AC600 with monitor mode)
```bash
git clone --depth 1 https://github.com/aircrack-ng/rtl8812au.git
cd rtl8812au

# Add Edimax AC600 USB ID
sed -i '/0x7392, 0xB611/a\\t{USB_DEVICE(0x7392, 0xd811), .driver_info = RTL8821}, /* Edimax AC600 */' os_dep/linux/usb_intf.c

# Fix Clang-incompatible flags
sed -i 's/-Wno-cast-function-type//g; s/-Wno-stringop-overread//g; s/-Wno-stringop-overflow//g' Makefile

make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=clang \
    KSRC=/path/to/kernel_source CONFIG_88XXAU=m modules
# Output: 88XXau.ko
```

### rtl8821cu (alternative driver for Edimax AC600 if it's actually RTL8821CU)
```bash
git clone --depth 1 https://github.com/morrownr/8821cu-20210916.git rtl8821cu
cd rtl8821cu

# Add Edimax AC600 USB ID
sed -i '/0x7392, 0xa811/a\\t{USB_DEVICE_AND_INTERFACE_INFO(0x7392, 0xd811, 0xff, 0xff, 0xff), .driver_info = RTL8821C}, /* Edimax AC600 */' os_dep/linux/usb_intf.c

# Fix Clang flags and recursive ccflags
sed -i 's/-Wno-cast-function-type//g; s/-Wno-stringop-overread//g; s/-Wno-stringop-overflow//g; s/-Wno-enum-int-mismatch//g' Makefile
sed -i 's/^EXTRA_CFLAGS += \$(ccflags-y)/# EXTRA_CFLAGS += $(ccflags-y)/' Makefile

make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=clang \
    KSRC=/path/to/kernel_source modules
# Output: 8821cu.ko
```

### rtl8188eus (Edimax N150)
```bash
git clone --depth 1 https://github.com/aircrack-ng/rtl8188eus.git
cd rtl8188eus
make -j$(nproc) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- CC=clang \
    KSRC=/path/to/kernel_source modules
# Output: 8188eu.ko
```

## Packaging as AnyKernel3 Zip
Use the NetHunter AnyKernel3 template. Replace `Image.gz-dtb` with your build output.
```bash
cp arch/arm64/boot/Image.gz-dtb anykernel3_template/
cd anykernel3_template
zip -r nethunter-kernel.zip . -x "*.git*"
```
Flash via: `adb sideload nethunter-kernel.zip` in recovery mode.

## Critical Notes
- Use `LD=aarch64-linux-gnu-ld` (GNU ld), NOT `ld.lld` — LLVM linker has MODVERSIONS issues with 4.19
- MODULE_SIG and MODVERSIONS must be disabled for out-of-tree module loading
- The Edimax AC600 (7392:d811) may be RTL8821CU not RTL8811AU — try both drivers
- Windows fastboot doesn't work for this device — always use Linux for recovery
- Recovery backup: `fastboot flash boot_b boot-backup.img`

## Configs Added (vs upstream)
See `docs/kernel-configs.md` for full list.

## Build History
See `docs/kernel-build-debrief.md` for what we tried and learned.
