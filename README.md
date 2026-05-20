# ImmortalWrt for Davolink DV01-901H (KT-708)

**User:** Mrtx1  
**Email:** Mrtx460@gmail.com  
**Device:** Davolink DV01-901H / KT GiGA WiFi Wave 2  
**SoC:** Qualcomm IPQ4019  
**Branch:** `master` (latest ImmortalWrt)

## 📁 Files Structure

```
.
├── .github/workflows/build-dv01-901h.yml   # GitHub Actions workflow
├── .config                                   # Build configuration + packages
├── dts/
│   └── qcom-ipq4019-dv01-901h.dts           # Device Tree Source (from your DTB!)
├── base-files/etc/board.d/
│   └── 02_network                           # Network config (LAN/WAN/LEDs)
├── image/
│   └── generic.mk.patch                     # Device profile for Makefile
└── README.md                                # This file
```

## 🚀 Build (Zero Config)

1. **Create new repo** on GitHub (name: `immortalwrt-dv01-901h`)
2. **Upload all files** maintaining the folder structure above
3. Go to **Actions** tab → **Build DV01-901H**
4. Click **Run workflow** → select `master` → **Run**
5. Wait 1-2 hours
6. Download firmware from **Artifacts** or **Releases**

## 📦 Packages Included (Same as your script)

| Category | Packages |
|----------|----------|
| **Proxy** | xray-core, sing-box, luci-app-passwall2 |
| **AdBlock** | adblock, luci-app-adblock |
| **Torrent** | transmission-daemon + web control |
| **NAS** | samba4-server, USB storage (ext4/ntfs/exfat) |
| **DNS** | mosdns, smartdns, dnsmasq-full |
| **VPN** | WireGuard, OpenVPN |
| **QoS** | nft-qos, sqm-scripts |
| **Monitor** | nlbwmon, collectd, tcpdump, iperf3, htop |
| **Tools** | ttyd, ddns-go, WOL, nmap, vim, nano |
| **Theme** | luci-theme-argon |
| **Firewall** | firewall4 (nftables) |

## 🔥 Flashing Guide

### ⚠️ CRITICAL: Backup ART First!

ART partition contains **unique WiFi calibration data** per device. Without it, WiFi is dead forever!

```bash
# In U-Boot, backup ART to TFTP:
tftpboot 0x80000000 ART
# OR via serial console in stock firmware:
cat /dev/mtd5 > /tmp/art_backup.bin
```

### Method 1: TFTP Recovery (Recommended)

1. **Open router case**, connect UART (TX, RX, GND) — **3.3V only!**
2. Set PC ethernet to `192.168.1.2`, run TFTP server
3. Copy `*-initramfs-fit-uImage.itb` to TFTP root
4. Power on router, press key in serial console to stop U-Boot
5. Run:
```
setenv ipaddr 192.168.1.1
setenv serverip 192.168.1.2
tftpboot 0x82000000 immortalwrt-ipq40xx-davolink_dv01-901h-initramfs-fit-uImage.itb
bootm 0x82000000
```
6. If it boots successfully, flash permanently:
```
sysupgrade -n immortalwrt-ipq40xx-davolink_dv01-901h-squashfs-nand-sysupgrade.bin
```

### Method 2: Stock Web UI (If Supported)

Try uploading `*-squashfs-nand-factory.ubi` through stock firmware web interface. May need to rename file.

## 🔧 Hardware Details (from your DTB file)

| Component | Spec |
|-----------|------|
| **SoC** | Qualcomm IPQ4019 (4x ARM Cortex-A7 @ 717MHz) |
| **RAM** | 256MB DDR3 |
| **Flash NOR** | 16MB (MX25L1606E / N25Q128A11) — U-Boot + ART |
| **Flash NAND** | 128MB — Kernel + Rootfs (UBI) |
| **WiFi** | QCA4019 (2.4GHz + 5GHz, 2x2 MIMO) |
| **Ethernet** | QCA8075 (4x LAN + 1x WAN Gigabit) |
| **USB** | USB 3.0 (DWC3) |
| **Buttons** | Reset (GPIO 63), WPS (GPIO 4) |
| **LEDs** | Status (GPIO 58), WAN, LAN1-4 |
| **UART** | 115200 baud, 3.3V (NOT 5V!) |

## ⚠️ Important Notes

- **NEVER erase ART partition (0x170000)** — WiFi dies without it!
- **NEVER erase APPSBL (0x0f0000)** — U-Boot bootloader!
- Test with **initramfs first** before permanent flash
- Default IP after flash: `192.168.1.1`
- Default login: root / (no password, set on first boot)

## 🆘 Troubleshooting

**Build fails?**
- Check Actions logs for errors
- Try `make -j1 V=s` for verbose single-thread build

**WiFi not working?**
- ART partition missing or corrupted
- Restore ART backup from stock firmware

**Ethernet not working?**
- Check MAC addresses in vmacs partition
- Verify switch configuration in 02_network

## 📚 References

- ImmortalWrt: https://github.com/immortalwrt/immortalwrt
- IPQ40xx Devices: https://openwrt.org/toh/views/toh_extended_all
- Your MAC Address (from label): `60:29:D5:0F:E1:4B`
