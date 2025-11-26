# Firmware Flash Troubleshooting - Detailed Log

**Date:** 2025-11-25  
**Issue:** Printer had corrupted firmware (blank screen, USB boot loop)  
**Resolution:** Klipper firmware was functional despite SD card flash indicators

---

## Initial Problem

### Symptoms
- Printer screen completely blank (no display output)
- Printer continuously rebooted when connected via USB
- USB enumeration errors in dmesg logs
- Printer not responding to any commands

### USB Error Pattern
```
usb 1-2: device descriptor read/64, error -71
usb 1-2: Device not responding to setup address
usb 1-2: device not accepting address, error -71
usb usb1-port2: unable to enumerate USB device
```

This pattern indicated corrupted or missing firmware on the printer's STM32F103 microcontroller.

---

## Attempted Solutions

### Attempt 1: SD Card Flash with Existing Firmware (Old Backup)
**Method:** Copy old working Klipper firmware (31KB from June 2024) to SD card as `firmware.bin`

**Steps:**
1. Backed up old firmware from SD card to `klipper-old-backup.bin`
2. Copied as `firmware.bin` to root of SD card
3. Power cycled printer with SD card inserted
4. Waited 30 seconds

**Result:** ❌ FAILED
- File remained as `firmware.bin` (not renamed to `firmware.CUR`)
- SD card bootloader did not process the file
- Printer continued to boot loop

**Why it failed:** SD card bootloader on SV06 was not responding to firmware files.

---

### Attempt 2: SD Card Flash with Newly Built Firmware
**Method:** Use freshly compiled Klipper firmware (36KB built Nov 17, 2025)

**Firmware Build Settings:**
- Architecture: STMicroelectronics STM32
- Processor: STM32F103
- Bootloader offset: 28KiB (0x7000)
- Clock: 8 MHz crystal
- Communication: Serial (USART1 PA10/PA9)

**Steps:**
1. Built firmware using `make` in ~/klipper
2. Copied `~/klipper/out/klipper.bin` to SD card as `firmware.bin`
3. Power cycled printer

**Result:** ❌ FAILED
- File remained as `firmware.bin`
- No evidence of bootloader processing the file

**Why it failed:** Same bootloader issue as Attempt 1.

---

### Attempt 3: Alternative Firmware Filenames
**Method:** Try different firmware filenames that some SV06 variants might expect

**Files Created on SD Card:**
- `firmware.bin`
- `Robin_nano.bin` (alternative name for some boards)
- `FIRMWARE.BIN` (uppercase variant)

**Steps:**
1. Created multiple copies with different names
2. Power cycled printer multiple times
3. Waited 30+ seconds each time

**Result:** ❌ FAILED
- No files were renamed to `.CUR`
- Bootloader did not process any filename variant

**Why it failed:** The SV06's SD card bootloader was either:
- Corrupted
- Looking for a different file format
- Disabled/non-functional

---

### Attempt 4: Direct USB Flash via stm32flash
**Method:** Attempt to flash firmware directly via USB while printer briefly appears as ttyUSB0

**Steps:**
1. Waited for printer to appear as `/dev/ttyUSB0`
2. Ran: `stm32flash -g 0x7000 -b 115200 -w klipper.bin /dev/ttyUSB0`

**Result:** ❌ FAILED
```
Error probing interface "serial_posix"
Cannot handle device "/dev/ttyUSB0"
Failed to open port: /dev/ttyUSB0
```

**Why it failed:** The printer was not in bootloader mode. STM32 bootloader requires BOOT0 pin to be high during power-on to enter DFU mode. The printer was booting into corrupted application firmware instead.

---

### Attempt 5: Official SV06 Pre-compiled Firmware
**Method:** Download and use official pre-compiled firmware from bassamanator/Sovol-SV06-firmware repository

**Firmware Details:**
- File: `klipper-v0.13.0-371-g7a723bdc.bin`
- Size: 37KB
- Source: https://github.com/bassamanator/Sovol-SV06-firmware/tree/master/misc

**Steps:**
1. Cloned the official repository
2. Copied pre-compiled firmware to SD card
3. Power cycled printer

**Result:** ❌ FAILED (at SD card level)
- File remained as `firmware.bin`
- SD card bootloader did not process it

**BUT:** This firmware may have been loaded during a previous successful boot cycle or the old firmware was still functional.

---

### Attempt 6: SD Card Reformatting with Exact Specifications
**Method:** Reformat SD card with EXACT specifications from official guide

**Official Requirements:**
- Size: 16GB maximum
- File system: FAT32
- Allocation unit size: 4096 bytes
- Must not contain any files except firmware file

**Steps Taken:**
1. Unmounted SD card
2. Wiped partition table: `wipefs -a /dev/sdb`
3. Created new partition: `parted /dev/sdb --script mklabel msdos mkpart primary fat32 1MiB 100%`
4. Formatted with specific cluster size: `mkfs.vfat -F 32 -s 8 -n KLIPPER /dev/sdb1`
   - `-s 8` = 8 sectors × 512 bytes = 4096 bytes per cluster
5. Verified: `fsck.vfat -v /dev/sdb1` showed "4096 bytes per cluster"
6. Copied ONLY firmware.bin (37KB official firmware)
7. Verified SD card contained no other files
8. Power cycled printer with 60 second wait

**Result:** ❌ FAILED (at SD card level)
- File remained as `firmware.bin`
- SD card bootloader still did not process it

**However:** This attempt led to the breakthrough...

---

## The Breakthrough

### Following the Official Guide's Advice

The official guide states:
> "At this point, it's not possible to tell with certainty whether your flash was successful, continue on with the guide."

**Decision:** Proceed with Klipper configuration despite SD card not showing success.

### Configuration Steps

1. **Downloaded Official SV06 Klipper Configuration**
   ```bash
   cd ~/printer_data/config
   rm -rf * .git*
   git clone -b master --single-branch https://github.com/bassamanator/Sovol-SV06-firmware.git .
   ```

2. **Configured MCU Serial Path**
   - Detected printer at: `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`
   - Updated `printer.cfg`:
     ```yaml
     [mcu]
     serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
     restart_method: command
     ```

3. **Fixed Permissions**
   ```bash
   sudo usermod -a -G dialout pri
   sudo usermod -a -G tty pri
   ```

4. **Restarted Klipper**
   ```bash
   sudo systemctl restart klipper
   ```

### Success!

**Klipper Logs Showed Communication:**
```
Stats 692410.6: gcodein=0 mcu: mcu_awake=0.000 mcu_task_avg=0.000000 
bytes_write=5489 bytes_read=6894 bytes_retransmit=9 bytes_invalid=0 
send_seq=275 receive_seq=275 retransmit_seq=2 srtt=0.004 rttvar=0.001 
rto=0.025 ready_bytes=0 upcoming_bytes=0 freq=72005522 
heater_bed: target=0 temp=23.8 pwm=0.000 
extruder: target=0 temp=24.9 pwm=0.000
```

**Key Indicators:**
- ✅ MCU frequency: `freq=72005522` (STM32 is running at ~72MHz)
- ✅ Temperature readings: Bed = 23.8°C, Hotend = 24.9°C
- ✅ Serial communication: Bytes read/write successful
- ✅ No error messages in logs

---

## Root Cause Analysis

### Why SD Card Method Failed

The SD card bootloader method failed for one or more of these reasons:

1. **Bootloader Corruption**
   - The STM32's SD card bootloader may have been corrupted
   - The bootloader looks for specific file formats/signatures that we didn't meet

2. **Bootloader Not Installed**
   - Some SV06 printers may not have the SD card bootloader installed
   - The 28KiB bootloader offset assumes a bootloader exists at address 0x0000-0x7000

3. **Incorrect Bootloader Offset**
   - We used 28KiB (0x7000) offset based on common SV06 configurations
   - The actual printer might use a different offset or no offset

4. **Hardware Issue**
   - SD card slot hardware issue
   - Timing issues with SD card detection

### Why Klipper Still Worked

**Possible Explanations:**

1. **Old Firmware Was Still Functional**
   - The original Klipper installation (from June 2024) was still on the printer
   - The blank screen was due to:
     - Missing/corrupted display firmware component
     - Display configuration issue
     - Hardware display issue
   - The core MCU firmware was intact

2. **One of Our Flash Attempts Succeeded**
   - Despite SD card not showing `.CUR` rename, the firmware may have been loaded
   - Some bootloaders don't rename the file but still process it
   - The printer may have successfully flashed during one of our many attempts

3. **USB Serial Bootloader Active**
   - The printer may have a USB-based bootloader that accepted our firmware
   - The brief USB connections during boot loops may have been flash opportunities

---

## Alternative Solutions (Not Attempted)

### Method 1: Physical BOOT0 Button Access

**Requirements:**
- Open printer's electronics enclosure
- Locate mainboard (usually at bottom/back)
- Find BOOT0 button or jumper on STM32 chip

**Procedure:**
1. Power OFF printer
2. Hold BOOT0 button (or set jumper to HIGH/1 position)
3. Connect USB cable to laptop
4. Power ON printer while holding BOOT0
5. Release BOOT0 after 2-3 seconds
6. Printer enters DFU mode
7. Verify with: `sudo dfu-util -l`
8. Flash with: `sudo dfu-util -a 0 -s 0x08000000:leave -D klipper.bin`

**Pros:**
- Most reliable method for corrupted firmware
- Direct access to STM32 bootloader
- Works even if application firmware is completely corrupted

**Cons:**
- Requires disassembly
- Risk of ESD damage to electronics
- BOOT0 button may be difficult to locate/access

---

### Method 2: ST-Link V2 Programmer

**Requirements:**
- ST-Link V2 USB programmer (~$10-20)
- Connection to SWD pins on mainboard (SWDIO, SWCLK, GND, 3.3V)

**Procedure:**
1. Open electronics enclosure
2. Locate SWD programming header on mainboard
3. Connect ST-Link V2 to SWD pins
4. Use `st-flash` utility:
   ```bash
   st-flash write klipper.bin 0x08000000
   ```

**Pros:**
- Most professional/reliable method
- Can completely erase and reprogram chip
- Can read existing firmware for backup
- Can verify programming success

**Cons:**
- Requires additional hardware ($)
- Requires disassembly and pin identification
- Risk of bricking if done incorrectly

---

### Method 3: Serial Bootloader (if BOOT0 accessible)

**Requirements:**
- Physical access to mainboard
- BOOT0 pin accessible

**Procedure:**
1. Enter bootloader mode (BOOT0 method above)
2. Use `stm32flash` over serial:
   ```bash
   stm32flash -w klipper.bin -v -g 0x0 /dev/ttyUSB0
   ```

**Note:** We attempted this but printer wasn't in bootloader mode.

---

### Method 4: ISP/JTAG Programming

**Requirements:**
- JTAG programmer
- JTAG pins accessible on mainboard

**Procedure:**
1. Connect JTAG programmer
2. Use OpenOCD to flash firmware

**Pros:**
- Most low-level access
- Can recover from any software brick

**Cons:**
- Expensive hardware required
- Complex setup
- Overkill for this situation

---

## Lessons Learned

### Key Takeaways

1. **Trust the Official Guide**
   - The guide explicitly states you can't tell if flash succeeded
   - Continue with configuration even if SD card shows no change
   - The firmware may be loaded despite no visible confirmation

2. **SD Card Bootloader Unreliable**
   - Many users report issues with SD card flashing on SV06
   - SD card method should not be the only option attempted
   - Always verify with USB serial connection afterwards

3. **USB Serial Connection is Diagnostic**
   - Even if screen is blank, USB serial connection can confirm firmware
   - Temperature readings are strong indicators of working firmware
   - MCU frequency indicates communication is established

4. **Multiple Firmware Sources Exist**
   - Old working firmware may persist despite blank screen
   - Screen issues don't always indicate firmware issues
   - Test communication before assuming firmware corruption

5. **Permissions Matter**
   - Adding user to `dialout` and `tty` groups is essential
   - Permission errors can look like communication errors
   - Always check logs for permission denied messages

### Best Practices for Future Flashing

1. **Always try USB serial configuration first** before assuming firmware flash failed
2. **Check Klipper logs** for actual communication errors vs permission errors
3. **Have physical access plan** ready (BOOT0 button location known)
4. **Keep backup firmware files** from working configurations
5. **Document firmware build settings** for reproducibility
6. **Test with multiple SD cards** if SD method is required

---

## Successful Configuration Summary

### What Actually Worked

**Method:** USB Serial Configuration (skipped SD card bootloader entirely)

**Steps:**
1. Connected printer via USB
2. Detected serial device: `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`
3. Downloaded official configuration from bassamanator/Sovol-SV06-firmware
4. Updated `printer.cfg` with correct serial path
5. Fixed user permissions (dialout/tty groups)
6. Restarted Klipper service
7. Verified communication via Klipper logs

**Result:** ✅ SUCCESS
- Klipper communicating with MCU
- Temperature sensors working
- Ready for Moonraker/Mainsail installation

### Firmware Version Information

**Active Firmware:** Unknown exact version (likely either old June 2024 firmware or one of our flashed versions)

**Evidence of Working Firmware:**
```
MCU: STM32F103 running at ~72MHz
Serial: CH340 USB-to-Serial converter
Communication: Active, bi-directional
Sensors: Bed thermistor, Hotend thermistor responding
```

---

## Technical Details

### Hardware Configuration

**Printer:** Sovol SV06  
**Mainboard:** Stock SV06 board with STM32F103 MCU  
**USB Interface:** CH340 Serial Converter (1a86:7523)  
**Serial Path:** `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`  

**MCU Specifications:**
- Chip: STM32F103
- Clock: 72MHz
- Bootloader: 28KiB (addresses 0x0000-0x7000)
- Application: Starts at 0x7000
- Flash: 64KB-128KB total
- UART: USART1 on PA10/PA9

### Network Configuration

**Static IP Configuration:**
- IP Address: 10.0.0.139/24
- Gateway: 10.0.0.1
- DNS: 64.71.255.204, 64.71.255.198
- Interface: wlo1 (WiFi)
- Method: NetworkManager static configuration

**Laptop Access:**
- SSH: `ssh pri@10.0.0.139`
- User: pri
- Groups: pri, dialout, tty, sudo, etc.

### Software Versions

**Host System:**
- OS: Debian 13 (Trixie)
- Kernel: (check with `uname -r`)
- Python: 3.13.5

**Klipper:**
- Location: ~/klipper
- Python Environment: ~/klippy-env
- Configuration: ~/printer_data/config
- Logs: ~/printer_data/logs

**Firmware Build Tools:**
- gcc: (GNU compiler for ARM)
- make: Standard build system
- stm32flash: 0.7
- dfu-util: 0.11

---

## Files and Locations

### Important Files

**Firmware Files:**
- Official firmware: `/tmp/Sovol-SV06-firmware/misc/klipper-v0.13.0-371-g7a723bdc.bin` (37KB)
- Built firmware: `~/klipper/out/klipper.bin` (36KB)
- Old backup: `/mnt/sdcard/klipper-old-backup.bin` (31KB, June 2024)

**Configuration Files:**
- Main config: `~/printer_data/config/printer.cfg`
- Moonraker config: `~/printer_data/config/moonraker.conf`
- Macro configs: `~/printer_data/config/cfgs/`

**System Files:**
- Klipper service: `/etc/systemd/system/klipper.service`
- Klipper logs: `~/printer_data/logs/klippy.log`
- USB rules: `/etc/udev/rules.d/` (if any)

### SD Card Details

**Device:** `/dev/sdb1`  
**Capacity:** 7.5GB  
**Format:** FAT32  
**Cluster Size:** 4096 bytes  
**Label:** KLIPPER  

---

## Timeline

**18:08** - Initial SD card detection, old firmware found  
**18:16** - First firmware copy to SD card  
**18:26** - Multiple flash attempts with different firmware files  
**18:34** - Attempted flash with old backup firmware  
**18:40** - Downloaded official bassamanator firmware  
**18:46** - Multiple firmware filename variants created  
**18:50** - Official firmware (37KB) copied to SD card  
**18:52** - SD card properly reformatted with 4096-byte clusters  
**18:54** - Printer connected via USB, detected as ttyUSB0  
**18:55** - Official Klipper configuration downloaded  
**18:56** - Klipper configured and restarted  
**18:56** - **SUCCESS**: MCU communication established  

---

## References

- [Official Guide: bassamanator/Sovol-SV06-firmware](https://github.com/bassamanator/Sovol-SV06-firmware)
- [Klipper Documentation](https://www.klipper3d.org/)
- [STM32 Flash Bootloader Protocol](https://www.st.com/resource/en/application_note/an3155-usart-protocol-used-in-the-stm32-bootloader-stmicroelectronics.pdf)
- [CH340 USB Serial Driver](https://github.com/torvalds/linux/blob/master/drivers/usb/serial/ch341.c)

---

## Status

✅ **RESOLVED**: Klipper firmware functional and communicating  
⏭️ **NEXT**: Install Moonraker and Mainsail web interface  

