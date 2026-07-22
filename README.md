# OpenWrt for Xiaomi Mi Router 4 Pro (R1350)

Device-specific OpenWrt 24.10 port for the **Xiaomi Mi Router 4 Pro (R1350 / YA105)**.

> ⚠️ **Not** "Mi Router 4" or "Mi Router Pro". Make sure your device matches the hardware listed below before flashing.

## Hardware

| Component | Details |
|-----------|---------|
| SoC | Qualcomm Atheros QCA9563 |
| 2.4 GHz | QCA9563 integrated (ath9k) |
| 5 GHz | Qualcomm QCA9888 hw2.0, PCIe `168c:0056` (ath10k) |
| Flash | 16 MiB SPI NOR |
| RAM | 128 MiB |
| Switch | AR8327, 1×WAN + 3×LAN |
| Reset button | GPIO18, active-low |
| LEDs | GPIO2 yellow:system, GPIO3 blue:system, GPIO4 yellow:function, GPIO5 blue:function (all active-low) |

## Status

| Feature | Status |
|---------|--------|
| Booting | ✅ Confirmed on real hardware |
| 2.4 GHz Wi-Fi | ✅ Working |
| 5 GHz Wi-Fi (QCA9888) | ✅ Working — calibration read from device ART partition |
| Ethernet WAN/LAN | ✅ Working |
| LEDs | ✅ Working — system/WAN/WiFi activity mapped |
| Reset button | ✅ Working |
| Software flow offload | ✅ Enabled by default (hardware offload not supported on QCA9563) |

## Pre-built Firmware

Download from [Releases](../../releases) or directly:

| File | Description |
|------|-------------|
| `R1350-*-sysupgrade.bin` | For devices already running OpenWrt |
| `R1350-*-initramfs-kernel.bin` | RAM-boot for initial testing (does not write flash) |

Latest firmware also available at:
```
https://dl.naixi.net/openwrt/r1350/
```

## Flashing

### From original firmware (first time)

Use U-Boot TFTP to RAM-boot the initramfs image first. **Do not write to flash until you have confirmed the device boots correctly.**

1. Back up the entire flash: `all.bin`, `art.bin`, `bootloader.bin`, `firmware.bin`
2. Connect a 3.3V TTL serial adapter (do **not** connect 5V)
3. TFTP-boot `R1350-*-initramfs-kernel.bin` into RAM — this does not touch flash
4. Verify: board name, Ethernet, LEDs, 2.4G, 5G
5. Only after successful RAM-boot, flash the sysupgrade via LuCI or `sysupgrade -n`

### From existing OpenWrt

```bash
sysupgrade -n openwrt-ath79-generic-xiaomi_mi-router-4-pro-r1350-squashfs-sysupgrade.bin
```

### Flash partition layout

| Partition | Offset | Size | Notes |
|-----------|--------|------|-------|
| Bootloader | `0x000000` | 192 KB | Read-only |
| Nvram | `0x030000` | 64 KB | Read-only |
| Bdata | `0x040000` | 64 KB | Read-only |
| crash | `0x050000` | 64 KB | Read-only |
| ART | `0x060000` | 64 KB | **Read-only — never overwrite** |
| cfg_bak | `0x070000` | 128 KB | Read-only |
| firmware | `0x200000` | 14 MB | OpenWrt kernel + rootfs |

## Building from Source

Tested on Debian 12 / Ubuntu 22.04:

```bash
# Install dependencies
sudo apt install -y build-essential clang flex bison g++ gawk gcc-multilib \
  gettext git libncurses-dev libssl-dev python3-distutils rsync unzip zlib1g-dev \
  file wget

# Clone OpenWrt 24.10 at the tested commit
git clone --branch openwrt-24.10 https://github.com/openwrt/openwrt.git
cd openwrt
git checkout 924f06e76cd73e24d39691a4de621bdcae80a756

# Apply patches
git apply ../patches/0001-r1350-add-dts.patch
git apply ../patches/0002-r1350-image-leds-network.patch
cp ../files/etc/uci-defaults/99-r1350-flow-offload \
   target/linux/ath79/generic/base-files/etc/uci-defaults/

# Build
cp ../patches/config.seed .config
./scripts/feeds update -a && ./scripts/feeds install -a
make defconfig
make -j"$(nproc)" download
make -j"$(nproc)"
```

Output: `bin/targets/ath79/generic/openwrt-ath79-generic-xiaomi_mi-router-4-pro-r1350-*`

## Recovery

To restore original firmware, write only the `firmware` partition (offset `0x200000`, max 14 MB) using your own device backup. **Never use another device's ART, Bdata, or Nvram.**

## What's in this repo

```
patches/
  0001-r1350-add-dts.patch          # Device Tree Source (new file)
  0002-r1350-image-leds-network.patch  # image/leds/network board definitions
  config.seed                        # minimal .config for single-device build
dts/
  qca9563_xiaomi_mi-router-4-pro-r1350.dts   # DTS for reference
files/
  etc/uci-defaults/99-r1350-flow-offload     # enables software flow offload
firmware/
  R1350-*-sysupgrade.bin            # pre-built sysupgrade
  R1350-*-initramfs-kernel.bin      # pre-built initramfs for RAM-boot testing
```

## Credits

Hardware identification and initial porting by [Nyarime](https://github.com/Nyarime) / [Naixi](https://naixi.net).  
Calibration data approach based on upstream OpenWrt ath79 conventions.  
LED GPIO mapping confirmed by real-hardware testing.

## License

Patches and DTS: GPL-2.0-or-later OR MIT (matching upstream OpenWrt).  
Pre-built firmware images are built from OpenWrt sources under the same license.
