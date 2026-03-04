# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a hardware security research project — a TP-Link router firmware extraction and analysis. There is no buildable source code; the repo contains binary firmware dumps, an extracted embedded Linux root filesystem, and documentation.

## Target Device

- **SoC:** MediaTek MT7628AN (MIPS 24Kc @ 580 MHz)
- **Flash:** 8 MB SPI NOR (EN25QX64A), only 4 MB used
- **OS:** Linux 2.6.36, SquashFS 4.0 rootfs, uClibc 0.9.33.2, BusyBox
- **Main daemon:** `cos` (TP-Link's Configuration and Operation System in `/usr/bin/cos`)

## Flash Partition Layout

| Offset | Size | Name | File |
|--------|------|------|------|
| 0x000000 | 64 KB | boot | `uboot.bin` |
| 0x010000 | 960 KB | kernel | `kernel.bin` |
| 0x100000 | ~2.9 MB | rootfs | `rootfs.squashfs` |
| 0x3E0000 | 64 KB | config | `config.bin` |
| 0x3F0000 | 64 KB | radio | `radio.bin` |

## Key Paths in rootfs/

- `etc/init.d/rcS` — Boot script (module loading, network init, launches `cos`)
- `etc/inittab` — Init config (UART on ttyS1, askfirst login)
- `etc/reduced_data_model.xml` — TR-069 style device data model
- `etc/default_config.xml` — Factory default configuration
- `etc/passwd.bak` — Default credentials (admin is UID 0)
- `usr/bin/` — All main daemons: cos, httpd, dropbear, dhcpd, dnsProxy, tddp, tdpd
- `lib/libcmm.so` — Core configuration management library (largest proprietary binary)
- `web/` — Web management interface HTML/JS files

## Architecture Notes

- All binaries are **MIPS32 rel2, little-endian, dynamically linked against uClibc**
- `cos` is the central daemon; it parses XML configs, manages all network interfaces (VLANs, bridge, WiFi), and spawns sub-services
- Network uses VLAN tagging: WAN=eth0.2, LAN ports=eth0.3-6, all LAN+WiFi bridged into br0
- The UART console (`ttyS1` at 115200 8N1) is present but requires login — `askfirst` in inittab
