# Recovery Guide

## If the Phone Bootloops

### Method 1: Fastboot (from Linux — Windows doesn't work)
1. Hold **Power + Volume Down** for 10-15 seconds → FASTBOOT MODE
2. From a Linux machine:
```bash
sudo apt install android-tools-fastboot
sudo fastboot flash boot_b boot-backup-20260324.img
sudo fastboot reboot
```

### Method 2: From Kali chroot (if SSH is still up)
```bash
echo b > /proc/sysrq-trigger
```
This forces a kernel reboot. If the old kernel is on a different slot, it may boot.

### Method 3: Stock recovery ADB sideload
1. Boot to recovery (Power + Volume Down → navigate to Recovery)
2. Select "Apply update from ADB"
3. Flash a known-good kernel zip

## Backup Image
- Location: `/sdcard/nightcrawler-kernels/boot-backup-20260324.img`
- Also keep a copy on your computer at all times
- This is the original Nameless AOSP 4.19.157-perf+ kernel

## Boot Slots
OnePlus 8T is an A/B device. The kernel was flashed to boot_b.
If boot_b is broken, you can switch slots:
```bash
sudo fastboot --set-active=a
sudo fastboot reboot
```

## "Problems with internal hardware" Warning
This is Android verified boot detecting a non-stock kernel. It's cosmetic and harmless. Dismiss it.
