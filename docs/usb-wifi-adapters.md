# USB WiFi Adapter Support

## Tested Adapters

| Adapter | USB ID | Chipset | Driver | Module | Status |
|---------|--------|---------|--------|--------|--------|
| Edimax N150 | 7392:b811 | RTL8188CUS/EUS | rtl8188eus | 8188eu.ko | **WORKING** — monitor mode, 28 networks scanned |
| Edimax AC600 | 7392:d811 | RTL8811AU or RTL8821CU | rtl8812au OR rtl8821cu | 88XXau.ko / 8821cu.ko | Interface created but radio silent (wrong driver?) |
| Edimax EW-7811Un | 7392:7811 | RTL8188CUS | rtl8xxxu / rtl8188eus | 8188eu.ko | Untested (should work with 8188eu) |
| Ralink | 148f:5370 | RT5370 | rt2800usb | in-tree (needs CONFIG) | Not built — rt2x00 dependencies unresolved |

## The Edimax AC600 Problem
The AC600 (7392:d811) is listed as RTL8811AU on Edimax's site, but:
- rtw88 project maps 7392:d811 → RTL8821**CU** (not AU)
- rtl8812au driver creates interface + monitor mode, but radio receives 0 packets
- rtl8821cu driver is the likely correct one

Both driver modules are available. Test with 8821cu.ko first.

## Module Loading
```bash
# Load
insmod /path/to/module.ko

# Verify
lsmod | grep <name>
dmesg | tail -20
ip link show

# Monitor mode
airmon-ng start wlan1
airodump-ng wlan1mon

# Unload
rmmod <name>
```
