# Print Quality Fixes - Post First Test Print

**Date:** 2025-11-26  
**Status:** [DONE] PARTIALLY COMPLETE  
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
```

**PURGE_LINE Macro Details:**
- Moves to front-left corner (X=0.5, Y=0.5)
- Draws 100mm purge line along X-axis at Z=0.4mm
- Extrudes 26.6mm of filament to fully prime nozzle
- Lifts to Z=5mm before starting print

**Verification:** Klipper restarted successfully, service active.

---

## Fixes Requiring User Action (OrcaSlicer)

These settings need to be changed in OrcaSlicer on the Windows PC.

### 2. First Layer Speed Settings [PENDING]

**Issue:** Default first layer speed too fast, causing poor bed adhesion. Required manual 50% speed reduction during first test print.

**Location:** OrcaSlicer > Printer Settings > Speed (or Process Settings)

| Setting | Current (Default) | Recommended |
|---------|-------------------|-------------|
| First layer speed | ~50 mm/s | **20-25 mm/s** |
| First layer infill speed | ~50 mm/s | **20-25 mm/s** |

**Why:** Slower first layer allows better adhesion and gives time for the filament to properly bond to the bed surface.

---

### 3. First Layer Line Width [PENDING]

**Issue:** Standard line width provides less bed contact area.

**Location:** OrcaSlicer > Quality > Line Width

| Setting | Current | Recommended |
|---------|---------|-------------|
| First layer line width | 100% (0.4mm) | **120% (0.48mm)** |

**Why:** Wider first layer lines "squish" more into the bed, improving adhesion and filling gaps better.

---

### 4. XY Hole Compensation [PENDING]

**Issue:** Threaded screw required pliers to insert into cube - tolerance too tight due to hole shrinkage.

**Location:** OrcaSlicer > Quality > Precision (or search "hole compensation")

| Setting | Current | Recommended |
|---------|---------|-------------|
| XY hole compensation | 0 mm | **0.1-0.2 mm** |

**Why:** Holes print smaller than designed due to:
- Material shrinkage during cooling
- Inner perimeter over-extrusion
- First layer squish

Start with 0.1mm and increase if needed.

---

## Future Calibration (Optional)

These are lower priority and require additional setup:

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
| Poor bed adhesion | Improved with slower speed + wider lines |
| Tight thread tolerance | Better fit with hole compensation |
| Corner blobs | Reduced (full fix needs Pressure Advance) |
| Wall rippling | Reduced (full fix needs Input Shaper) |

---

## Verification Plan

1. **Re-print calibration cube** after OrcaSlicer settings are updated
2. **Compare results** to first print photos
3. **Document improvements** in follow-up log

---

## Files Modified

| File | Change | Status |
|------|--------|--------|
| `~/printer_data/config/cfgs/misc-macros.cfg` | Added PURGE_LINE call to PRINT_START | [DONE] Applied |
| OrcaSlicer printer profile | First layer speed 20-25mm/s | [PENDING] User action |
| OrcaSlicer printer profile | First layer width 120% | [PENDING] User action |
| OrcaSlicer printer profile | XY hole compensation 0.1-0.2mm | [PENDING] User action |

---

## Current Status

- [DONE] Purge line added to PRINT_START macro
- [DONE] Klipper restarted and verified
- [PENDING] OrcaSlicer first layer speed adjustment
- [PENDING] OrcaSlicer first layer width adjustment
- [PENDING] OrcaSlicer XY hole compensation
- [PENDING] Verification print

---

**Next Action:** Update OrcaSlicer settings on Windows PC, then re-print calibration cube to verify improvements.
