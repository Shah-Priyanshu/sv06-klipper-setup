# Print Quality Fixes - Post First Test Print

**Date:** 2025-11-26  
**Status:** [DONE] FIRMWARE FIXES COMPLETE  
**Related Log:** [17-first-test-print-analysis.md](17-first-test-print-analysis.md)

---

## Summary

This log documents the fixes implemented to address issues identified in the first test print (OrcaSlicer calibration cube).

---

## Fixes Implemented

### 1. Add Purge Line to PRINT_START Macro [DONE]

**Issue:** No purge line was being drawn before printing, causing:
- Inconsistent first layer extrusion
- Stringing at print start
- Rough bottom surfaces
- Poor first layer adhesion

**Root Cause:** The `PRINT_START` macro ended with `G1 Z20 F3000` (lift nozzle) without priming the nozzle first. A `PURGE_LINE` macro existed in the config but was never called.

**Fix Applied:**

Modified `~/printer_data/config/cfgs/misc-macros.cfg`:

```gcode
# Before (end of PRINT_START):
    M109 S{hotendtemp}                                   ; set & wait for hotend temp

    G1 Z20 F3000                                         ; move nozzle away from bed

# After:
    M109 S{hotendtemp}                                   ; set & wait for hotend temp

    PURGE_LINE                                           ; draw purge line to prime nozzle
    _FIRST_LAYER_SETTINGS                                ; apply first layer speed limits
```

**PURGE_LINE Macro Details:**
- Moves to front-left corner (X=0.5, Y=0.5)
- Draws 100mm purge line along X-axis at Z=0.4mm
- Extrudes 26.6mm of filament to fully prime nozzle
- Lifts to Z=5mm before starting print

**Verification:** Klipper restarted successfully, service active.

---

### 2. Firmware-Enforced First Layer Speed Limiting [DONE]

**Issue:** Default first layer speed too fast, causing poor bed adhesion. Required manual 50% speed reduction during first test print. Slicer settings alone are fragile - changing slicers or profiles could reintroduce the problem.

**Solution:** Implement firmware-level speed limiting that:
1. Automatically applies slow speeds at print start
2. Automatically restores normal speeds after first layer
3. Works regardless of slicer settings

**Macros Added to `misc-macros.cfg`:**

```gcode
[gcode_macro _FIRST_LAYER_SETTINGS]
description: Apply conservative settings for first layer
gcode:
    {% set FIRST_LAYER_VELOCITY = 25 %}      ; mm/s - max speed for first layer
    {% set FIRST_LAYER_ACCEL = 1000 %}       ; mm/s^2 - max acceleration for first layer
    
    SET_VELOCITY_LIMIT VELOCITY={FIRST_LAYER_VELOCITY} ACCEL={FIRST_LAYER_ACCEL}

[gcode_macro _NORMAL_SETTINGS]
description: Restore normal printer settings after first layer
gcode:
    {% set max_velocity = printer.configfile.settings.printer.max_velocity|default(200)|int %}
    {% set max_accel = printer.configfile.settings.printer.max_accel|default(3000)|int %}
    
    SET_VELOCITY_LIMIT VELOCITY={max_velocity} ACCEL={max_accel}

[gcode_macro SET_PRINT_STATS_INFO]
rename_existing: _SET_PRINT_STATS_INFO_BASE
description: Intercept layer changes to restore speed after first layer
gcode:
    _SET_PRINT_STATS_INFO_BASE {rawparams}
    
    {% if params.CURRENT_LAYER is defined %}
        {% set layer = params.CURRENT_LAYER|int %}
        {% if layer == 2 %}
            _NORMAL_SETTINGS
        {% endif %}
    {% endif %}
```

**How It Works:**
1. `PRINT_START` calls `_FIRST_LAYER_SETTINGS` → limits speed to 25mm/s, accel to 1000mm/s²
2. OrcaSlicer sends `SET_PRINT_STATS_INFO CURRENT_LAYER=2` when layer 2 starts
3. Our intercepted macro detects layer 2 and calls `_NORMAL_SETTINGS` → restores 200mm/s, 3000mm/s²
4. `PRINT_END` also calls `_NORMAL_SETTINGS` as a safety cleanup

**First Layer Limits:**
| Parameter | First Layer | Normal |
|-----------|-------------|--------|
| Max Velocity | 25 mm/s | 200 mm/s |
| Max Acceleration | 1000 mm/s² | 3000 mm/s² |

**Benefits:**
- Slicer-independent - works with any slicer or profile
- Automatic - no manual speed adjustment needed
- Self-restoring - normal speeds resume automatically after layer 1
- Safe - even if slicer sends faster speeds, firmware caps them

**Verification:** Klipper restarted successfully, service active.

---

## Optional OrcaSlicer Settings (User Action)

These settings are now **optional** since firmware handles speed limiting. However, they can still improve print quality:

### 3. First Layer Line Width [OPTIONAL]

**Location:** OrcaSlicer > Quality > Line Width

| Setting | Current | Recommended |
|---------|---------|-------------|
| First layer line width | 100% (0.4mm) | **120% (0.48mm)** |

**Why:** Wider first layer lines "squish" more into the bed, improving adhesion.

---

### 4. XY Hole Compensation [RECOMMENDED]

**Issue:** Threaded screw required pliers to insert into cube - tolerance too tight due to hole shrinkage.

**Location:** OrcaSlicer > Quality > Precision (or search "hole compensation")

| Setting | Current | Recommended |
|---------|---------|-------------|
| XY hole compensation | 0 mm | **0.1-0.2 mm** |

**Why:** Holes print smaller than designed due to material shrinkage. This cannot be fixed in firmware.

---

## Future Calibration (Optional)

### Pressure Advance Tuning

**Purpose:** Improves extrusion consistency at corners and direction changes, reducing:
- Corner bulging/blobs
- Diagonal banding on walls

**How to Calibrate:**
1. Print Pressure Advance test pattern
2. Measure optimal PA value
3. Add to printer.cfg: `pressure_advance: 0.XXX`

**Documentation:** [Klipper Pressure Advance Guide](https://www.klipper3d.org/Pressure_Advance.html)

---

### Input Shaper Calibration

**Purpose:** Reduces ringing/ghosting artifacts on walls

**Requirements:** ADXL345 accelerometer connected to printer

**How to Calibrate:**
1. Connect accelerometer to MCU
2. Run `SHAPER_CALIBRATE`
3. Apply recommended shaper settings

**Documentation:** [Klipper Input Shaper Guide](https://www.klipper3d.org/Resonance_Compensation.html)

---

## Expected Improvements

After implementing all fixes:

| Issue | Expected Result |
|-------|-----------------|
| Rough first layer | Smooth, consistent first layer |
| Stringing at print start | Eliminated by purge line |
| Poor bed adhesion | Improved with firmware speed limiting |
| Tight thread tolerance | Better fit with hole compensation (slicer) |
| Corner blobs | Reduced (full fix needs Pressure Advance) |
| Wall rippling | Reduced (full fix needs Input Shaper) |

---

## Files Modified

| File | Change | Status |
|------|--------|--------|
| `~/printer_data/config/cfgs/misc-macros.cfg` | Added PURGE_LINE call to PRINT_START | [DONE] |
| `~/printer_data/config/cfgs/misc-macros.cfg` | Added _FIRST_LAYER_SETTINGS call to PRINT_START | [DONE] |
| `~/printer_data/config/cfgs/misc-macros.cfg` | Added _NORMAL_SETTINGS call to PRINT_END | [DONE] |
| `~/printer_data/config/cfgs/misc-macros.cfg` | Added _FIRST_LAYER_SETTINGS macro | [DONE] |
| `~/printer_data/config/cfgs/misc-macros.cfg` | Added _NORMAL_SETTINGS macro | [DONE] |
| `~/printer_data/config/cfgs/misc-macros.cfg` | Added SET_PRINT_STATS_INFO interceptor | [DONE] |
| OrcaSlicer printer profile | XY hole compensation 0.1-0.2mm | [OPTIONAL] User action |

---

## Current Status

- [DONE] Purge line added to PRINT_START macro
- [DONE] First layer speed limiting (firmware-enforced)
- [DONE] Automatic speed restoration after layer 1
- [DONE] Klipper restarted and verified
- [OPTIONAL] OrcaSlicer XY hole compensation (for thread tolerance)
- [PENDING] Verification print

---

**Next Action:** Re-print calibration cube to verify improvements. Optionally set XY hole compensation in OrcaSlicer for better thread fit.
