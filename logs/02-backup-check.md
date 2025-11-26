# Step 2: Backup Check

**Date:** 2025-11-16

## Purpose
Check existing Klipper installation for any configs worth backing up before complete removal.

## Commands to Run

```bash
# Check for printer configs
ls -la ~/printer_data/config/ 2>/dev/null || echo "No printer_data directory"

# If it exists, show the printer.cfg
cat ~/printer_data/config/printer.cfg 2>/dev/null || echo "No printer.cfg found"

# Check klipper logs
ls -la ~/klipper_logs/ 2>/dev/null || echo "No klipper_logs"

# Check systemd services
systemctl list-units --type=service | grep -E 'klipper|moonraker'
```

## Output

### Config Directory Listing
```
total 276
drwxrwxr-x  3 user user  4096 Sep  3 16:40 .
drwxrwxr-x 12 user user  4096 Jun 16  2024 ..
-rw-r--r--  1 user user  2109 Jun 15  2024 added_macros.cfg
-rw-r--r--  1 user user 88918 Dec 15  2024 config-20241215-133326.zip
drwxrwxr-x  5 user user  4096 Jun 15  2024 klipper-macros
lrwxrwxrwx  1 user user    37 Jun 15  2024 mainsail.cfg -> /home/user/mainsail-config/client.cfg
-rw-r--r--  1 user user  2037 Dec 12  2024 moonraker.conf
-rw-r--r--  1 user user  3100 Sep  3 16:40 printer.cfg
lrwxrwxrwx  1 user user    58 Nov 10  2024 timelapse.cfg -> /home/user/moonraker-timelapse/klipper_macro/timelapse.cfg
(Plus 38 backup printer.cfg files from June 2024 to September 2025)
```

### Current printer.cfg (SV06 Configuration)
- **MCU Serial:** /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0 (CH340 converter)
- **Kinematics:** Cartesian
- **Stepper drivers:** TMC2209 with sensorless homing on X/Y
- **Probe:** BLTouch style (pin PB1, offset X:27 Y:-23)
- **Z-offset:** 2.060 (calibrated)
- **Bed mesh:** 10x10 bicubic
- **PID tuned:** Extruder and bed both tuned

### Services Running
```
klipper.service   - loaded active running Klipper 3D Printer Firmware SV1
moonraker.service - loaded active running API Server for Klipper SV1
```

### Klipper Logs
```
No klipper_logs directory found
```

---

## Analysis

[DONE] **Good existing config** - The printer.cfg is well-configured for SV06 with proper TMC2209 settings
[DONE] **PID tuned** - Both hotend and bed have calibrated PID values
[DONE] **Z-offset calibrated** - Z-offset is set to 2.060
[DONE] **38 backup configs** - Good history of changes
[WARN] **OctoEverywhere installed** - Extra service we may or may not need

## Files Worth Saving

Before removal, we should backup:
1. **printer.cfg** - Current working configuration
2. **added_macros.cfg** - Custom macros
3. **moonraker.conf** - Moonraker configuration
4. Latest backup: **config-20241215-133326.zip**

## Decision

[DONE] **Proceed with complete removal and fresh installation**
We'll backup the important configs to this repository first, then completely remove everything.
