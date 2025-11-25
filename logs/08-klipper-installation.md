# Klipper Installation Guide - SV06

**Date:** 2025-11-17  
**Status:** In Progress

## Overview

Setting up Klipper firmware on Sovol SV06 from scratch. This document guides through the complete process.

## Prerequisites Completed

- [x] Debian 13 system installed and updated
- [x] SSH access configured (10.0.0.139)
- [x] Build dependencies installed (git, python3, gcc, etc.)
- [x] Klipper repository cloned to `~/klipper`
- [x] Python virtual environment created at `~/klippy-env`
- [x] Klipper Python dependencies installed
- [x] Directory structure created at `~/printer_data/`

## Installation Steps

### Step 1: Build Klipper MCU Firmware

The SV06 uses an STM32F103 microcontroller. We need to compile Klipper firmware for it.

**Configure firmware build:**
```bash
cd ~/klipper
make menuconfig
```

**Settings for SV06:**
- Enable extra low-level configuration options: YES
- Micro-controller Architecture: STMicroelectronics STM32
- Processor model: STM32F103
- Bootloader offset: 28KiB bootloader
- Clock Reference: 8 MHz crystal
- Communication interface: Serial (on USART1 PA10/PA9)

**Build the firmware:**
```bash
make
```

This creates `out/klipper.bin` file.

### Step 2: Flash Firmware to Printer

**Prepare SD card:**
1. Format SD card as FAT32
2. Copy `~/klipper/out/klipper.bin` to SD card
3. Rename file on SD card to `firmware.bin`

**Flash process:**
1. Power off printer
2. Insert SD card into printer
3. Power on printer
4. Wait 10-20 seconds (printer will flash firmware)
5. Power off, remove SD card, power on

**Verify:**
- SD card file should be renamed to `firmware.CUR` (indicates successful flash)

### Step 3: Find Printer Serial Port

**With printer connected via USB:**
```bash
ls /dev/serial/by-id/
```

Look for device like: `usb-1a86_USB_Serial-if00-port0`

### Step 4: Create printer.cfg

Create minimal configuration file at `~/printer_data/config/printer.cfg`

We'll build this step-by-step, starting with MCU connection.

### Step 5: Setup Klipper Service

Create systemd service to run Klipper on boot.

**Service file:** `/etc/systemd/system/klipper.service`

**Enable and start:**
```bash
sudo systemctl enable klipper
sudo systemctl start klipper
```

### Step 6: Install Moonraker (API)

Moonraker provides the API interface for web frontends.

```bash
cd ~
git clone https://github.com/Arksine/moonraker
cd moonraker
./scripts/install-moonraker.sh
```

### Step 7: Install Web Interface

Choose either:
- **Mainsail** (recommended): Modern, clean interface
- **Fluidd**: Alternative with different UI

## Current Progress

- [x] Prerequisites installed
- [x] Firmware configured and built
- [x] Static IP configured (10.0.0.139)
- [x] SD card prepared with firmware.bin
- [ ] Firmware flashed to printer
- [ ] Serial port identified
- [ ] printer.cfg created
- [ ] Klipper service configured
- [ ] Moonraker installed
- [ ] Web interface installed

## Firmware Configuration Details

**MCU Settings Used:**
- Architecture: STMicroelectronics STM32
- Processor: STM32F103
- Bootloader offset: 28KiB
- Clock: 8 MHz crystal
- Communication: Serial (USART1 PA10/PA9)

**Build command:** `make`
**Output file:** `~/klipper/out/klipper.bin`

## Next Steps

1. Configure and build MCU firmware
2. Flash firmware to SV06
3. Create basic printer.cfg
4. Test Klipper connection
5. Install Moonraker and web interface

## Session: 2025-11-25 - Network Configuration and Firmware Preparation

### Issue: Dynamic IP Changed
**Problem:** Laptop IP changed from 10.0.0.139 to 10.0.0.143, breaking SSH access.

**Root cause:** NetworkManager was configured for DHCP, causing IP to change on each connection.

**Solution: Configure Static IP**

1. **Identify network connection:**
   ```bash
   nmcli connection show
   # Connection: "86 Roomies" (WiFi)
   # Interface: wlo1
   # Gateway: 10.0.0.1
   ```

2. **Configure static IP:**
   ```bash
   sudo nmcli connection modify '86 Roomies' \
     ipv4.addresses 10.0.0.139/24 \
     ipv4.gateway 10.0.0.1 \
     ipv4.dns '64.71.255.204,64.71.255.198' \
     ipv4.method manual
   ```

3. **Apply changes:**
   ```bash
   sudo nmcli connection down '86 Roomies'
   sudo nmcli connection up '86 Roomies'
   ```

4. **Verification:**
   ```bash
   ip addr show wlo1 | grep 'inet 10'
   # Output: inet 10.0.0.139/24 brd 10.0.0.255 scope global noprefixroute wlo1
   ```

**Result:** ✅ Static IP 10.0.0.139 configured successfully. IP will no longer change.

### SD Card Firmware Preparation

**SD card detected:**
- Device: `/dev/sdb1` (7.5GB FAT32)
- Mount: `/media/pri/12AE-E2DE`

**Firmware preparation steps:**

1. **Backup old firmware:**
   ```bash
   mv /media/pri/12AE-E2DE/klipper.bin /media/pri/12AE-E2DE/klipper-old-backup.bin
   ```
   - Old firmware: 31KB (from June 2024)

2. **Copy new firmware:**
   ```bash
   cp ~/klipper/out/klipper.bin /media/pri/12AE-E2DE/firmware.bin
   ```
   - New firmware: 36KB (built Nov 17, 2025)
   - MD5: `0d71c31abc45eb6156ae406d2b4793f8`

3. **Verify integrity:**
   ```bash
   md5sum ~/klipper/out/klipper.bin /media/pri/12AE-E2DE/firmware.bin
   # Both files match: 0d71c31abc45eb6156ae406d2b4793f8
   ```

4. **Safely unmount:**
   ```bash
   sync
   sudo umount /media/pri/12AE-E2DE
   ```

**Status:** ✅ SD card ready with fresh Klipper firmware (firmware.bin)

### Next Steps

**Ready to flash firmware to SV06:**

1. Remove SD card from laptop
2. Power OFF printer
3. Insert SD card into printer
4. Power ON printer (wait 10-20 seconds for automatic flash)
5. Power OFF printer
6. Remove SD card and verify `firmware.bin` renamed to `firmware.CUR`
7. Power ON printer with new firmware

After successful flash:
- Connect printer via USB to laptop
- Identify serial port (`ls /dev/serial/by-id/`)
- Configure Klipper systemd service
- Create printer.cfg
- Install Moonraker and Mainsail

## Notes

- Starting fresh without using backup config
- Will build printer.cfg step-by-step for better understanding
- SV06 uses CH340 USB-to-serial chip (driver usually included in Linux)
- Static IP ensures consistent SSH access at 10.0.0.139
