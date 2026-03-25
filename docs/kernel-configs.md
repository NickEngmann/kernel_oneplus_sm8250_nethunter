# Kernel Configuration Changes

All configs added on top of the Nameless AOSP `vendor/kona-perf_defconfig`.

## NetHunter Wireless
| Config | Value | Purpose |
|--------|-------|---------|
| CONFIG_MAC80211 | y | Wireless stack for USB WiFi adapters |
| CONFIG_MAC80211_MESH | y | Mesh networking support |
| CONFIG_MAC80211_LEDS | y | LED trigger support |
| CONFIG_WIRELESS_EXT | y | Wireless extensions (legacy tools) |
| CONFIG_WEXT_CORE | y | Wext core |
| CONFIG_WEXT_PROC | y | Wext /proc interface |
| CONFIG_WEXT_PRIV | y | Wext private ioctls |
| CONFIG_CFG80211_WEXT | y | cfg80211 wext compatibility |
| CONFIG_88XXAU | y | rtl8812au driver (Kconfig entry, build as module separately) |

## Security/VPN
| Config | Value | Purpose |
|--------|-------|---------|
| CONFIG_WIREGUARD | y | WireGuard VPN (kernel-level, used by Tailscale) |
| CONFIG_NF_TABLES | y | nftables firewall framework |

## USB Adapters
| Config | Value | Purpose |
|--------|-------|---------|
| CONFIG_BT_HCIBTUSB | y | USB Bluetooth adapters |
| CONFIG_USB_SERIAL_CH341 | y | CH341 USB-to-serial (hardware hacking) |
| CONFIG_USB_SERIAL_PL2303 | y | PL2303 USB-to-serial |
| CONFIG_USB_SERIAL_OPTION | m | USB modem (4G/LTE dongles) |
| CONFIG_USB_RTL8150 | y | USB ethernet adapter |
| CONFIG_USB_NET_CDC_EEM | y | USB CDC EEM networking |

## Networking
| Config | Value | Purpose |
|--------|-------|---------|
| CONFIG_NETLINK_DIAG | y | Socket diagnostics (ss command) |
| CONFIG_PPP_ASYNC | y | Async PPP (USB modem dial-up) |
| CONFIG_PACKET_DIAG | y | Packet socket diagnostics |

## Module Loading (disabled for flexibility)
| Config | Value | Purpose |
|--------|-------|---------|
| CONFIG_MODULE_SIG | not set | Allows unsigned modules |
| CONFIG_MODVERSIONS | not set | Allows modules from different builds |

## Already Present (no changes needed)
CONFIG_CFG80211=y, CONFIG_TUN=y, CONFIG_BRIDGE=y, CONFIG_VETH=y,
CONFIG_NF_NAT=y, CONFIG_IP_NF_NAT=y, CONFIG_NETFILTER_XT_MATCH_CONNTRACK=y,
CONFIG_USB_ACM=y, CONFIG_USB_SERIAL=y, CONFIG_USB_SERIAL_CP210X=y,
CONFIG_USB_CONFIGFS_F_HID=y, CONFIG_HID_GENERIC=y, CONFIG_SND_USB_AUDIO=y,
CONFIG_QCOM_KGSL=y (GPU), CONFIG_QCA_CLD_WLAN=y (internal WiFi)
