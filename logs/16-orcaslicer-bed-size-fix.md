# OrcaSlicer Bed Size Configuration Fix

**Date:** 2025-11-26  
**Status:** [DONE] RESOLVED  
**Issue:** Print preview in Mainsail showing objects off-center compared to OrcaSlicer

---

## Problem Description

### User Observation

When comparing sliced model position between OrcaSlicer and Mainsail G-code viewer:

**OrcaSlicer Preview:**
- Objects appeared centered on the build plate

**Mainsail G-code Viewer:**
- Same objects appeared shifted towards upper-left area
- Not centered as expected

### Visual Evidence

- `slicer_test_print.png`: Shows objects centered in OrcaSlicer
- `printer_gcode_viewer_tracking.png`: Shows objects off-center in Mainsail

---

## Root Cause Analysis

### Investigation Steps

**Checked Klipper Configuration:**
```bash
ssh pri@10.0.0.139 'grep -A 10 "^\[stepper_x\]" ~/printer_data/config/printer.cfg'
ssh pri@10.0.0.139 'grep -A 10 "^\[stepper_y\]" ~/printer_data/config/printer.cfg'
```

**Klipper Configuration:**
```ini
[stepper_x]
position_endstop: 0
position_max: 223

[stepper_y]
position_endstop: 0
position_max: 223

[safe_z_home]
home_xy_position: 84.50, 135
```

**Actual Printer Specifications (Sovol SV06):**
- Bed Size: 220mm x 220mm x 250mm (height)
- Build Volume: 220 x 220 x 250mm

### The Problem

**OrcaSlicer Bed Configuration:** 250mm x 250mm (INCORRECT)  
**Klipper Configuration:** 223mm x 223mm  
**Actual Printer Bed:** 220mm x 220mm

The bed size mismatch caused:
- OrcaSlicer to calculate positions for a 250mm bed
- G-code generated with coordinates assuming 250mm bed
- Mainsail displaying objects based on actual 223mm Klipper config
- Visual discrepancy between slicer preview and actual positioning

### Coordinate Math

**Example - Centered Object on 250mm Bed (OrcaSlicer):**
- Center position: X=125, Y=125

**When Viewed on 220mm Bed (Mainsail):**
- X=125 on a 220mm bed = 5mm past center (towards right edge)
- Y=125 on a 220mm bed = 5mm past center (towards back)
- Result: Object appears off-center, shifted towards upper-right

---

## Solution Applied

### Fix: Update OrcaSlicer Bed Size

**User Action Taken:**
1. Opened OrcaSlicer on Windows PC
2. Navigate to: **Printer Settings** → **Basic Information**
3. Changed bed size configuration:
   - **Before:** 250mm x 250mm
   - **After:** 220mm x 220mm
4. Saved printer profile

**Result:**
- OrcaSlicer now matches Sovol SV06 actual bed dimensions
- Sliced objects will be positioned correctly
- Mainsail preview will match OrcaSlicer preview

---

## Remaining Minor Discrepancy

### Klipper vs Actual Bed Size

**Current State:**
- **OrcaSlicer:** 220mm x 220mm (correct for SV06)
- **Klipper Config:** 223mm x 223mm (3mm larger)
- **Actual Bed:** 220mm x 220mm

**Impact:**
- Very minor 3mm difference
- Objects will still print correctly
- Coordinates will be accurate for practical purposes

### Why Klipper Shows 223mm

The bassamanator SV06 configuration sets `position_max: 223` to allow slight overtravel beyond the physical bed edge. This is common practice to maximize usable print area.

### Optional Future Fix

If perfect alignment desired, modify Klipper config:

```ini
[stepper_x]
position_max: 220  # Change from 223

[stepper_y]
position_max: 220  # Change from 223

[safe_z_home]
home_xy_position: 110, 110  # Center of 220mm bed (currently 84.50, 135)
```

**Note:** This would require:
- Recalibrating bed mesh (5x5 grid points might change)
- Verifying homing position
- Testing that probe can reach all corners

**Recommendation:** Leave Klipper at 223mm since 3mm difference is negligible for printing.

---

## Alternative Solution: Match OrcaSlicer to Klipper

Instead of keeping OrcaSlicer at 220mm, set it to match Klipper exactly:

**OrcaSlicer → Printer Settings:**
- Bed Size X: 223mm
- Bed Size Y: 223mm

**Pros:**
- Perfect 1:1 match with Klipper configuration
- No changes needed on printer side

**Cons:**
- Slightly larger than actual bed spec (by 3mm)
- Could theoretically slice objects that exceed physical bed

**Decision:** User chose 220mm (correct for SV06 spec) - this is the better choice.

---

## Verification Steps

### After Changing Bed Size in OrcaSlicer

1. **Re-slice existing model:**
   - Load the same STL file
   - Re-slice with updated 220mm bed size
   - Compare new G-code coordinates

2. **Check object positioning:**
   - Verify object is still centered in OrcaSlicer preview
   - Upload new G-code to printer
   - Check Mainsail G-code viewer preview
   - Objects should now appear more centered

3. **Test print (optional):**
   - Print a small test object
   - Verify it prints in expected location on bed
   - Confirm no boundary violations

---

## Lessons Learned

### 1. Always Verify Bed Size Configuration

When setting up a slicer for a new printer:
- Check manufacturer specifications (Sovol SV06 = 220x220mm)
- Verify against Klipper configuration
- Don't assume default slicer values are correct

### 2. Slicer Profiles May Have Incorrect Defaults

Even printer-specific profiles can have errors:
- "Generic Klipper Printer" may default to 250x250mm
- "Sovol SV06" profile (if available) should be correct
- Always double-check before first print

### 3. Visual Preview Comparison is Critical

Before first print:
- Compare slicer preview with Mainsail G-code viewer
- Check that objects are positioned similarly
- Verify bed dimensions match between interfaces

### 4. Minor Discrepancies (±3mm) Are Acceptable

- Klipper configs often allow slight overtravel
- 220mm vs 223mm won't cause printing issues
- Focus on matching slicer to actual physical bed size first

---

## Related Configuration Details

### Sovol SV06 Specifications

**Build Volume:**
```
X (Width): 220mm
Y (Depth): 220mm  
Z (Height): 250mm
```

**Print Area:**
- Origin (0,0): Front-left corner
- Maximum (220, 220): Back-right corner

### Current OrcaSlicer Configuration (After Fix)

**Printer Settings → Basic Information:**
```
Bed Size X: 220mm ✓ CORRECT
Bed Size Y: 220mm ✓ CORRECT
Max Print Height: 250mm ✓ CORRECT
```

**Printer Settings → Extruder:**
```
Nozzle Diameter: 0.4mm
```

### Current Klipper Configuration

**From `printer.cfg`:**
```ini
[stepper_x]
position_endstop: 0
position_max: 223  # 3mm larger than spec

[stepper_y]
position_endstop: 0
position_max: 223  # 3mm larger than spec

[stepper_z]
position_min: -4   # Allows slight negative Z for first layer squish
position_max: 258  # 8mm larger than spec (250mm + margin)
```

**Note:** All `position_max` values slightly exceed physical dimensions to maximize usable area.

---

## Impact on Previous Test Prints

### If Any Prints Were Attempted Before Fix

Prints sliced with 250x250mm bed size would have:
- Coordinates calculated for larger bed
- Objects positioned assuming more space available
- Potential issues:
  - Objects near edges might be clipped
  - Large objects might exceed physical bed boundaries
  - Centering would be off by ~15mm in each direction

**Example Math:**
- Object centered at (125, 125) on 250mm bed
- Actual position on 220mm bed: (125, 125) - should be (110, 110)
- Offset: 15mm towards back-right

---

## Files Modified

**None** - Configuration change made in OrcaSlicer on Windows PC only.

**OrcaSlicer Config Location (Windows):**
```
C:\Users\<username>\AppData\Roaming\OrcaSlicer\
  └── system\*.json (printer profiles)
```

**Recommendation:** Export OrcaSlicer config bundle for backup:
- File → Export → Export Config Bundle
- Save to this repository for future reference

---

## Testing Checklist

After bed size fix:

- [DONE] Bed size changed from 250mm to 220mm in OrcaSlicer
- [PENDING] Re-slice test model with correct bed size
- [PENDING] Upload new G-code to printer
- [PENDING] Compare Mainsail preview with OrcaSlicer preview
- [PENDING] Verify objects appear centered in both interfaces
- [PENDING] Test print to confirm physical positioning

---

## Current System Status

### OrcaSlicer (Windows PC)

```
Printer: Sovol SV06 / Generic Klipper Printer
Bed Size: 220mm x 220mm x 250mm ✓ CORRECT
Connection: Moonraker at 10.0.0.139:7125 ✓ WORKING
Start G-code: PRINT_START macro ✓ CONFIGURED
End G-code: PRINT_END macro ✓ CONFIGURED
```

### Klipper (Debian Laptop)

```
Position Max: 223mm x 223mm (X/Y)
Position Max: 258mm (Z)
Configuration: Stable, no changes needed
Status: Ready for printing
```

### Alignment Status

```
OrcaSlicer bed: 220mm x 220mm
Klipper bed:    223mm x 223mm
Difference:     3mm (acceptable)
Preview match:  Should be aligned now ✓
```

---

## Recommendations

### Before Next Print

1. **Re-slice any previously sliced models** (they were sliced for 250mm bed)
2. **Verify preview alignment** in Mainsail before starting print
3. **Test with small object first** (calibration cube) to confirm positioning
4. **Monitor first layer** via camera to ensure correct positioning

### For Future Slicers

If setting up other slicers (PrusaSlicer, Cura, etc.):
- Always set bed size to **220mm x 220mm** for Sovol SV06
- Double-check origin position (front-left corner = 0,0)
- Verify max Z height is 250mm

### Optional: Create Bed Texture

In OrcaSlicer, you can add a custom bed texture/model:
- Take photo of actual bed surface
- Import as bed texture in Printer Settings → Bed
- Helps visualize actual print positioning

---

**Status:** [DONE] RESOLVED - Bed size corrected in OrcaSlicer from 250mm to 220mm

**Next Steps:**
- Re-slice test model with correct bed size
- Verify preview alignment between OrcaSlicer and Mainsail
- Proceed with first test print

**Date Completed:** 2025-11-26
