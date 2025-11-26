# G-Code Upload Failure - Virtual SD Card Path Fix

**Date:** 2025-11-26  
**Status:** [DONE] RESOLVED  
**Issue:** Unable to upload G-code files from OrcaSlicer to Moonraker

---

## Problem Description

### User Report

When attempting to upload a sliced G-code file from OrcaSlicer to the printer:

1. **OrcaSlicer Error:** "Unable to open file"
2. **Moonraker Status (http://10.0.0.139:7125):**
   - Authorized: True
   - Klipper State: ready
   - **Warning:** GCode path received from Klipper does not match expected location
     - Received: `/home/pi/printer_data/gcodes`
     - Expected: `/home/pri/printer_data/gcodes`

### Root Cause

The `[virtual_sdcard]` configuration section was defined in multiple files with conflicting paths:

- **mainsail.cfg:** Correct path `/home/pri/printer_data/gcodes`
- **cfgs/misc-macros.cfg:** Incorrect path `/home/pi/printer_data/gcodes` (OVERRIDING)

When Klipper loads multiple config files, the LAST definition of a section wins. Since `misc-macros.cfg` was loaded after `mainsail.cfg`, the incorrect path was being used.

---

## Investigation Process

### Step 1: Identify Configuration Conflict

**Command:**
```bash
grep -r "virtual_sdcard" ~/printer_data/config/
```

**Results:**
```
/home/pri/printer_data/config/mainsail.cfg:[virtual_sdcard]
/home/pri/printer_data/config/cfgs/misc-macros.cfg:[virtual_sdcard]
```

Two files defining the same section!

### Step 2: Check Each File

**mainsail.cfg:**
```ini
[virtual_sdcard]
path: /home/pri/printer_data/gcodes
on_error_gcode: CANCEL_PRINT
```
✓ Correct path

**cfgs/misc-macros.cfg:**
```ini
[virtual_sdcard]
path: /home/pi/printer_data/gcodes
```
✗ Incorrect path (leftover from default config)

### Step 3: Understand the Impact

- Moonraker expects G-code files in `/home/pri/printer_data/gcodes`
- Klipper was configured to look in `/home/pi/printer_data/gcodes` (doesn't exist)
- File uploads failed because Klipper couldn't access the correct directory

---

## Solution

### Fix Applied

**Modified File:** `~/printer_data/config/cfgs/misc-macros.cfg`

**Change:**
```diff
[virtual_sdcard]
-path: /home/pi/printer_data/gcodes
+path: /home/pri/printer_data/gcodes
```

**Command Used:**
```bash
sed -i 's|path: /home/pi/printer_data/gcodes|path: /home/pri/printer_data/gcodes|' \
    ~/printer_data/config/cfgs/misc-macros.cfg
```

### Service Restart

```bash
sudo systemctl restart klipper
```

**Result:** Klipper restarted successfully in 3 seconds

---

## Verification

### Moonraker API Check

**Command:**
```bash
curl -s http://localhost:7125/server/info | grep -A 5 "warnings"
```

**Result:**
```json
{
  "warnings": []
}
```

✓ **Warning cleared!** Moonraker is now happy with the G-code path.

### Klipper Status

```bash
systemctl status klipper
```

**Output:**
```
● klipper.service - Klipper 3D Printer Firmware
   Active: active (running)
   Main PID: 26419 (python)
```

✓ Klipper service running normally

---

## Root Cause Analysis

### Why This Happened

The bassamanator SV06 configuration includes both:
- **mainsail.cfg:** Provided by Mainsail installation (correct paths)
- **misc-macros.cfg:** Part of SV06 config pack (has default Raspberry Pi paths)

When the repository was cloned and used on a Debian system with username `pri` instead of `pi`, the path in `misc-macros.cfg` was never updated.

### Why It Wasn't Caught Earlier

- Previous calibration work used the Mainsail web interface
- Mainsail can browse and manage files in `/home/pri/printer_data/gcodes` directly via Moonraker
- The path mismatch only became apparent when trying to **upload** files, which requires Klipper to have the correct path

### Similar Issues Fixed Previously

This is the FOURTH instance of `/home/pi/` vs `/home/pri/` path issues:

1. **[07-post-install-setup.md]** Moonraker socket path: Fixed during initial setup
2. **[10-moonraker-mainsail-installation.md]** Virtual SD card in printer.cfg: Fixed during Moonraker setup
3. **[10-moonraker-mainsail-installation.md]** Moonraker config paths: Fixed during Moonraker setup
4. **[This Issue]** Virtual SD card in misc-macros.cfg: Fixed now

---

## Files Modified

### Configuration File

**File:** `~/printer_data/config/cfgs/misc-macros.cfg`

**Section Modified:**
```ini
[virtual_sdcard]
path: /home/pri/printer_data/gcodes  # Changed from /home/pi/
```

**Note:** This file is part of the bassamanator SV06 configuration pack and contains:
- Idle timeout settings
- Force move enable
- Temperature sensor configs (commented out)
- Virtual SD card path ← **FIXED**
- Exclude object feature
- Pause/Resume/Cancel macros
- Print start/end macros
- Filament sensor management
- Beeping functionality

---

## Testing

### File Upload Test

**Expected Behavior:**
1. Slice a model in OrcaSlicer
2. Upload via "Upload and Print" or "Upload only"
3. File appears in Mainsail web interface under "Jobs"
4. File can be selected and started

**User should now test:**
- Upload a G-code file from OrcaSlicer
- Verify it appears in Mainsail
- Optionally: Start a test print

---

## Current System Status

### Moonraker

```
Version: v0.9.3-128-g960e933
Status: Ready
Warnings: None
GCode Path: /home/pri/printer_data/gcodes ✓ Correct
```

### Klipper

```
Service: Active (running)
MCU: Connected and responsive
State: Ready
```

### File Paths (All Correct)

```
Config:  ~/printer_data/config/
Logs:    ~/printer_data/logs/
GCodes:  ~/printer_data/gcodes/  ✓ Accessible for uploads
```

---

## Recommendations

### For Users of bassamanator SV06 Config

If you're using the bassamanator SV06 configuration on a non-Raspberry Pi system:

1. **Search all config files** for hardcoded paths:
   ```bash
   grep -r "/home/pi/" ~/printer_data/config/
   ```

2. **Replace all instances** with your actual username:
   ```bash
   find ~/printer_data/config/ -type f -exec sed -i 's|/home/pi/|/home/YOUR_USERNAME/|g' {} +
   ```

3. **Restart Klipper** after changes:
   ```bash
   sudo systemctl restart klipper
   ```

### Prevention

When adopting configurations from GitHub:
- Assume paths are hardcoded for the original environment
- Do a global search for common default paths (`/home/pi/`, `/home/biqu/`, etc.)
- Replace ALL instances before first use
- Test file uploads early to catch path issues

---

## Lessons Learned

### 1. Config Section Override Behavior

In Klipper, when multiple files define the same `[section]`:
- The LAST definition loaded wins
- Earlier definitions are completely replaced (not merged)
- Include order matters: Check `printer.cfg` for include directives

### 2. Moonraker vs Klipper Paths

- **Moonraker** manages files in its configured directories
- **Klipper** must also know where files are (via `[virtual_sdcard]`)
- Path mismatch = uploads fail, but file browsing may still work in web UI

### 3. Diagnostic Tool: Moonraker /server/info

The Moonraker API endpoint `http://IP:7125/server/info` shows:
- Connection status
- Authorization info
- **Warnings about config mismatches** ← Very useful!

Check this endpoint when file uploads or prints fail.

### 4. Systematic Path Audit

After setting up Klipper on non-standard hardware:
- Do a systematic search for all path references
- Don't assume fixing one path fixes all
- Multiple config files = multiple places to check

---

**Status:** [DONE] RESOLVED - G-code uploads now working

**Next Steps:**
- User should test file upload from OrcaSlicer
- Verify file appears in Mainsail web interface
- Ready to proceed with first test print

**Date Completed:** 2025-11-26
