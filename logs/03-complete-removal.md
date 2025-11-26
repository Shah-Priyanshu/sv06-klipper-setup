# Step 3: Complete Removal of Old Installation

**Date:** 2025-11-16

## Purpose
Completely remove the existing Klipper installation to start fresh with proper documentation.

## Pre-removal Backup
[DONE] Saved current printer.cfg to `configs/backup-old-printer.cfg`

## Important Notes
- **Serial ID preserved:** `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`
- **Z-offset to remember:** 2.060
- **PID values saved** in backup config

---

## Removal Commands

Run these commands on the laptop to completely remove Klipper:

```bash
# Stop all services first
sudo systemctl stop klipper
sudo systemctl stop moonraker
sudo systemctl stop nginx  # If using reverse proxy

# Disable services from autostart
sudo systemctl disable klipper
sudo systemctl disable moonraker

# Remove systemd service files
sudo rm -f /etc/systemd/system/klipper.service
sudo rm -f /etc/systemd/system/moonraker.service
sudo systemctl daemon-reload

# Remove Klipper directories
rm -rf ~/klipper
rm -rf ~/moonraker
rm -rf ~/mainsail
rm -rf ~/fluidd
rm -rf ~/klipper_logs
rm -rf ~/printer_data
rm -rf ~/mainsail-config
rm -rf ~/moonraker-timelapse

# Remove OctoEverywhere (if you don't want it)
rm -rf ~/octoeverywhere
rm -rf ~/octoeverywhere-system

# Remove any KIAUH installation
rm -rf ~/kiauh

# Clean up nginx configs (if any)
sudo rm -f /etc/nginx/sites-enabled/mainsail
sudo rm -f /etc/nginx/sites-enabled/fluidd
sudo rm -f /etc/nginx/sites-available/mainsail
sudo rm -f /etc/nginx/sites-available/fluidd

# Remove klippy environment
rm -rf ~/klippy-env

# Check what's left
ls -la ~ | grep -E 'klipper|moonraker|mainsail|fluidd|octo'
```

---

## Verification

After running the removal commands, verify everything is gone:

```bash
# Check services are not running
systemctl list-units --type=service | grep -E 'klipper|moonraker'

# Should return nothing
ls -la ~/klipper ~/moonraker ~/mainsail 2>/dev/null
```

---

## Output

### Services Stopped and Disabled
```
sudo systemctl stop klipper moonraker
sudo systemctl disable klipper moonraker
Removed /etc/systemd/system/multi-user.target.wants/klipper.service.
Removed /etc/systemd/system/multi-user.target.wants/moonraker.service.
```

### Main Directories Removed
```
[DONE] ~/klipper
[DONE] ~/moonraker
[DONE] ~/mainsail
[DONE] ~/fluidd
[DONE] ~/klipper_logs
[DONE] ~/printer_data
[DONE] ~/mainsail-config
[DONE] ~/moonraker-timelapse
[DONE] ~/octoeverywhere
[DONE] ~/octoeverywhere-system
[DONE] ~/kiauh
[DONE] ~/klippy-env
```

### Remaining Files Found
```
drwxrwxr-x  4 user user     4096 Jun 15  2024 moonraker-env
drwxrwxr-x  4 user user     4096 Jun 16  2024 octoeverywhere-env
-rw-r--r--  1 user user    34374 Jun 16  2024 octoeverywhere-installer.log
```

### Services Check
```
[DONE] No klipper or moonraker services running
```

## Final Cleanup

```bash
rm -rf ~/moonraker-env
rm -rf ~/octoeverywhere-env
rm -f ~/octoeverywhere-installer.log
```

### Final Verification
```
=== Final check for Klipper-related files ===
All clean!

=== System ready for fresh installation ===
Python 3.10.12
git version 2.34.1
```

---

## Status

[DONE] **Complete removal successful!**
[DONE] **System clean and ready for fresh installation**
[DONE] **Python 3.10.12 available**
[DONE] **Git 2.34.1 available**

Ready to proceed with KIAUH installation.
