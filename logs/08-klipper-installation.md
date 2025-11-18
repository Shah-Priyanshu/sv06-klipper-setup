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

## Notes

- Starting fresh without using backup config
- Will build printer.cfg step-by-step for better understanding
- SV06 uses CH340 USB-to-serial chip (driver usually included in Linux)
