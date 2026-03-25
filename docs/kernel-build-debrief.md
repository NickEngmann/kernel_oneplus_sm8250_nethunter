# Kernel Build Debrief

## What Worked
- **v4 kernel** from Nameless AOSP `thirteen` branch (4.19.297) boots on OnePlus 8T with Nameless AOSP Android 12
- All vendor modules work: WiFi (qcacld), GPU (KGSL/Adreno 650), camera, display, audio, USB
- MAC80211, WireGuard, nftables built into the kernel
- 8188eu.ko module loads and works (Edimax N150 USB WiFi adapter confirmed working with monitor mode)

## What Failed and Why

### Pre-built NetHunter kernels (a11, a12 zips from flypatriot) — BOOTLOOP
Built against crDroid ROM. The kernel code's vendor module implementations don't match Nameless AOSP's vendor partition. DTB wasn't the issue — the kernel code itself is incompatible.

### DTB swap (crDroid kernel + Nameless AOSP DTB) — BOOTLOOP
Confirmed the problem is kernel code, not DTB. The vendor HAL bindings in crDroid kernel don't match Nameless AOSP.

### Building crDroid kernel from source — BOOTS but wrong ROM
Successfully compiled flypatriot's crDroid kernel. Proved we could build a kernel, but it still doesn't boot on Nameless AOSP.

### Building Nameless AOSP `fifteen` branch — BUILD FAILED
Too new (4.19.322). Missing trace headers, broken techpack includes. The `fifteen` branch has diverged too far.

### GCC 13 — BUILD FAILED
Kernel requires Clang. GCC 13 has format string incompatibilities and the Makefile hardcodes `CC = clang`.

### ld.lld (LLVM linker) — BUILD FAILED
MODVERSIONS `.tmp_*.ver` files not found. Use GNU ld (`aarch64-linux-gnu-ld`) instead.

## Build Fixes Required

### 1. Trace headers (75+ Makefiles)
**Problem:** `define_trace.h:89: fatal error: ./trace.h: No such file or directory`
**Cause:** Clang resolves relative includes differently from GCC
**Fix:** `ccflags-y += -I$(src)` in every directory containing a trace header with `TRACE_INCLUDE_PATH = .`

### 2. techpack/display/pll/Makefile (trailing backslash)
**Problem:** Last `obj-y` line had `\` continuation, our `ccflags-y` append corrupted the object list
**Fix:** Rewrote Makefile without trailing backslash on last line

### 3. Camera techpack include paths
**Problem:** `cam_cci_dev.h`, `cam_trace.h`, `cam_context.h` not found across subdirectories
**Fix:** Added comprehensive `ccflags-y += -I$(srctree)/techpack/camera/drivers/...` to all camera Makefiles

### 4. cam_ois/onsemi_fw parent directory include
**Problem:** `onsemi_fw/fw_download_interface.h` includes `"cam_ois_dev.h"` from parent dir, Clang can't find it
**Fix:** `KBUILD_CPPFLAGS += -iquote $(srctree)/techpack/camera/drivers/cam_sensor_module/cam_ois`

### 5. -ftrivial-auto-var-init (Clang 12 incompatible)
**Problem:** `-ftrivial-auto-var-init=zero` and its enabler flag not supported by Clang 12.0.7
**Fix:** Removed all `trivial-auto-var` lines from Makefile

### 6. MODULE_SIG_FORCE
**Problem:** Out-of-tree modules rejected without signing
**Fix:** Disabled CONFIG_MODULE_SIG entirely

### 7. MODVERSIONS CRC mismatch
**Problem:** Modules built against different config state had mismatched symbol CRCs
**Fix:** Disabled CONFIG_MODVERSIONS

## Module Build Gotchas

### rtl8812au
- USB ID 7392:d811 (Edimax AC600) must be added manually to `os_dep/linux/usb_intf.c`
- Remove GCC-only flags: `-Wno-cast-function-type`, `-Wno-stringop-overread`, `-Wno-stringop-overflow`
- Must pass `CONFIG_88XXAU=m` when building as module (defconfig has `=y` which means built-in, not module)

### rtl8821cu
- Also needs 7392:d811 USB ID added
- **CRITICAL:** Remove `EXTRA_CFLAGS += $(ccflags-y)` from Makefile — creates infinite recursion with `scripts/Makefile.lib` line 4 (`ccflags-y += $(EXTRA_CFLAGS)`)
- Remove `-Wno-enum-int-mismatch` (Clang doesn't have it)

### rtl8188eus
- Builds cleanly with no modifications needed

### rtw88 (lwfinger)
- **DOES NOT WORK on kernel 4.19** — requires `RX_FLAG_NO_PSDU` and other newer mac80211 APIs
- Also needs `__nonstring` attribute removed for Clang 12

## Recovery
- **Always keep boot-backup.img** on computer AND phone
- **Use Linux for fastboot** — Windows USB drivers don't work for OnePlus 8T fastboot
- From Kali chroot if SSH is up: `echo b > /proc/sysrq-trigger` forces kernel reboot
- Fastboot recovery: `sudo fastboot flash boot_b boot-backup.img && sudo fastboot reboot`

## Key Learnings
1. Kernel source MUST match the ROM — crDroid kernel won't boot on Nameless AOSP
2. Use Clang r416183b1 (12.0.7) — the exact compiler Nameless AOSP uses
3. Use GNU ld, not ld.lld — LLVM linker broken with MODVERSIONS on 4.19
4. `scripts/config + olddefconfig` is the right way to add configs
5. Out-of-tree drivers need `CONFIG_XX=m` (not =y) when building as modules
6. Disable MODULE_SIG and MODVERSIONS for flexible module loading
7. Never append to a Makefile line with trailing backslash
8. The Edimax AC600 (7392:d811) might be RTL8821CU, not RTL8811AU
