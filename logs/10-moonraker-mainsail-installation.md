# Moonraker and Mainsail Installation

**Date:** 2025-11-25  
**Status:** ✅ COMPLETED  
**Prerequisites:** ✅ Klipper firmware verified and communicating  
**Installation Method:** KIAUH (Klipper Installation And Update Helper)

---

## Installation Summary

✅ **Successfully Installed:**
- **Moonraker v0.9.3-128-g960e933** - API server for Klipper
- **Mainsail** - Web interface (latest stable)
- **Mainsail-Config** - Recommended macros and configuration
- **nginx** - Web server serving Mainsail on port 80

✅ **All Services Running:**
- Klipper: Active, connected to MCU
- Moonraker: Active, connected to Klipper
- nginx: Active, serving web interface

✅ **Web Interface Accessible:**
- URL: http://10.0.0.139
- Status: Fully functional
- Printer controls verified (homing successful)

---

## Current Status

✅ **Completed:**
- Debian 13 system installed and configured
- Static IP configured: 10.0.0.139
- Klipper firmware functional and communicating with MCU
- MCU communication verified (temp sensors working)
- Official SV06 configuration downloaded and active
- KIAUH installed at `~/kiauh`
- **Moonraker installed and configured**
- **Mainsail web interface installed**
- **nginx web server installed and configured**
- **Configuration path issues fixed**
- **Web interface verified working**

⏳ **Next Steps:**
- Initial printer calibration (PID tuning, bed mesh, etc.)

---

## Installation Method: KIAUH

**KIAUH** (Klipper Installation And Update Helper) is the recommended tool for managing Klipper ecosystem components. It handles:
- Installation of Klipper, Moonraker, web interfaces
- Updates and version management
- Service configuration
- Dependency management
- Safe uninstallation

**KIAUH Location:** `~/kiauh`  
**Launch Command:** `~/kiauh/kiauh.sh`

### Why KIAUH?

- **Official Recommendation:** Recommended by Klipper documentation
- **Consistent Setup:** Ensures proper directory structure and permissions
- **Easy Updates:** Simplifies updating components
- **Multiple UIs:** Can install Mainsail, Fluidd, or Octoprint
- **Safe Management:** Handles dependencies and service configurations

---

## Installation Plan

### Step 1: Verify Current Installation Status

Check what's already installed via KIAUH to avoid conflicts.

### Step 2: Install Moonraker via KIAUH

Moonraker is the API server that provides web access to Klipper.
- Option 2 in KIAUH main menu: "Install"
- Select Moonraker from the install menu

### Step 3: Install Mainsail via KIAUH

Mainsail is the web-based user interface for controlling the printer.
- Option 2 in KIAUH main menu: "Install"
- Select Mainsail from the install menu
- KIAUH will automatically install and configure nginx

### Step 4: Verify Services

Ensure all services are running:
```bash
systemctl status klipper
systemctl status moonraker
systemctl status nginx
```

### Step 5: Test Web Access

Access the printer via web browser at http://10.0.0.139

---

## KIAUH Installation Steps (Detailed)

### Launching KIAUH

```bash
cd ~/kiauh
./kiauh.sh
```

KIAUH presents a text-based menu interface with options:
1. **[Install]** - Install Klipper ecosystem components
2. **[Update]** - Update installed components
3. **[Remove]** - Remove components
4. **[Advanced]** - Advanced options
5. **[Backup]** - Backup configurations
Q. **[Quit]** - Exit KIAUH

### Installation Process

**For Moonraker:**
1. Launch KIAUH
2. Select `1) [Install]`
3. Select `2) [Moonraker]`
4. Follow prompts (accept defaults for standard setup)
5. Wait for installation to complete
6. Return to main menu

**For Mainsail:**
1. From KIAUH main menu, select `1) [Install]`
2. Select `3) [Mainsail]` (or appropriate number)
3. KIAUH will:
   - Download Mainsail
   - Install nginx if not present
   - Configure nginx for Mainsail and Moonraker
   - Set up proper permissions
4. Wait for installation to complete

### Post-Installation Verification

KIAUH will show installation status. Verify with:
```bash
systemctl status klipper
systemctl status moonraker
systemctl status nginx
```

All services should show "active (running)" in green.

---

---

## Installation Process (Actual Steps Performed)

### Step 1: Verify KIAUH Status
✅ Confirmed KIAUH installed at `~/kiauh`

### Step 2: Remove Old Moonraker Installation
✅ Used KIAUH Remove menu to clean up incomplete Moonraker installation
- Removed service, repository, Python environment, and policykit rules

### Step 3: Install Moonraker via KIAUH
✅ KIAUH Menu: `1) [Install]` → `2) [Moonraker]`
- Created example moonraker.conf: Yes
- Cloned from: https://github.com/Arksine/moonraker
- Created Python virtual environment at `~/moonraker-env`
- Installed Python dependencies and speedups
- Installed policykit rules
- Created systemd service
- **Result:** Moonraker v0.9.3-128-g960e933 installed successfully

### Step 4: Install Mainsail via KIAUH
✅ KIAUH Menu: `1) [Install]` → `3) [Mainsail]` → `1) Reinstall Mainsail`
- Downloaded recommended Mainsail-Config: Yes
- Downloaded latest Mainsail release
- Created config backup: `~/kiauh_backups/printer_data/config_20251125-192139`
- Cloned Mainsail-Config repository
- Created symlink for mainsail.cfg
- Configured nginx automatically
- **Result:** Mainsail installed and accessible

### Step 5: Fix Configuration Path Issues

**Issue 1: Moonraker Connection Error**
- **Symptom:** "Moonraker can't connect to Klipper"
- **Cause:** Socket path in moonraker.conf used `/home/pi/` instead of `/home/pri/`
- **Fix:** Updated `klippy_uds_address` in moonraker.conf
  ```bash
  sed -i 's|/home/pi/|/home/pri/|g' ~/printer_data/config/moonraker.conf
  sudo systemctl restart moonraker
  ```

**Issue 2: Virtual SD Card Path Warning**
- **Symptom:** Moonraker warning about gcode path mismatch
- **Cause:** Tilde expansion in mainsail.cfg not working as expected
- **Fix:** Updated to explicit full path
  ```bash
  sed -i 's|path: ~/|path: /home/pri/|g' ~/printer_data/config/mainsail.cfg
  sudo systemctl restart klipper
  ```

### Step 6: Verify Installation
✅ All services active and connected:
- Klipper: Connected to MCU, temps reading normally
- Moonraker: Connected to Klipper (status: "ready")
- nginx: Serving Mainsail on port 80

### Step 7: Test Web Interface
✅ Accessed http://10.0.0.139
✅ Mainsail loaded successfully
✅ Printer controls functional - **verified by homing printer**

---

## Installed Component Details

### Moonraker
- **Version:** v0.9.3-128-g960e933
- **Repository:** Arksine/moonraker
- **Location:** `/home/pri/moonraker`
- **Python Env:** `/home/pri/moonraker-env`
- **Port:** 7125
- **Config:** `~/printer_data/config/moonraker.conf`

### Mainsail
- **Repository:** mainsail-crew/mainsail
- **Location:** `/home/pri/mainsail`
- **Access:** http://10.0.0.139

### Mainsail-Config
- **Repository:** mainsail-crew/mainsail-config
- **Location:** `/home/pri/mainsail-config`
- **Config:** `~/printer_data/config/mainsail.cfg` (symlink)

### nginx
- **Config:** `/etc/nginx/sites-available/mainsail`
- **Port:** 80
- **Logs:** Linked to `~/printer_data/logs/`

---

## Configuration Files Modified

1. **`~/printer_data/config/moonraker.conf`**
   - Fixed: `klippy_uds_address: /home/pri/printer_data/comms/klippy.sock`

2. **`~/printer_data/config/mainsail.cfg`**
   - Fixed: `path: /home/pri/printer_data/gcodes` in `[virtual_sdcard]`

3. **`~/printer_data/config/printer.cfg`**
   - Added: `[include mainsail.cfg]` (by KIAUH)

---

## Troubleshooting Note: KIAUH Path Issue

**Common Issue:** KIAUH uses `/home/pi/` as default in config templates (Raspberry Pi standard), causing connection issues on systems with different usernames.

**Solution:** After KIAUH installation, verify and fix paths:
```bash
# Check for pi paths in configs
grep -r "/home/pi" ~/printer_data/config/

# Fix moonraker.conf
sed -i 's|/home/pi/|/home/USERNAME/|g' ~/printer_data/config/moonraker.conf

# Fix mainsail.cfg
sed -i 's|path: ~/|path: /home/USERNAME/|g' ~/printer_data/config/mainsail.cfg

# Restart services
sudo systemctl restart moonraker klipper
```

---

## Next Steps

1. ✅ Verify KIAUH installation
2. ✅ Install Moonraker via KIAUH
3. ✅ Install Mainsail via KIAUH
4. ✅ Fix configuration path issues
5. ✅ Verify all services running
6. ✅ Test web interface and printer controls
7. ⏳ **Begin printer calibration** (PID tuning, bed mesh, input shaper, etc.)

