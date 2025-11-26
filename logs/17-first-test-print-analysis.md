# First Test Print Analysis - Orca Calibration Cube

**Date:** 2025-11-26  
**Status:** [PENDING] IN PROGRESS  
**Print:** OrcaSlicer Calibration Cube

---

## Print Conditions

### Issues During Print
1. **No purge line** - First layer outer perimeter didn't print properly
2. **Speed too fast** - Filament wasn't sticking to bed
3. **Manual intervention** - Reduced to 50% speed factor in Mainsail device tab

---

## Image Analysis

### Set 1: X and Y Faces

#### X Face (cube_x.jpg)

**Observed Issues:**
- **Significant layer inconsistency** - visible waviness/rippling across the surface, especially on the right side of the X
- **Diagonal banding pattern** - lines are not uniform, suggesting possible mechanical issues or extrusion inconsistency
- **Corner damage** - bottom-left and top-left corners show poor adhesion/lifting (likely from the missing purge line affecting first layers)
- **Surface texture varies** - some areas smoother than others

#### Y Face (cube_y.jpg)

**Observed Issues:**
- **Similar diagonal banding/rippling** - consistent with X face, suggesting a systemic issue
- **Better overall appearance** than X face but still has visible layer lines
- **Top surface** looks relatively clean
- **Bottom corner damage** - again showing first layer adhesion problems
- **Slight bulging** on lower right area

#### Set 1 Summary

| Issue | Severity | Likely Cause |
|-------|----------|--------------|
| Missing first layer perimeter | High | No purge line in start G-code |
| Diagonal rippling/banding | Medium | Speed too fast, possible ringing/ghosting |
| Corner lifting | Medium | First layer adhesion + no purge |
| Inconsistent extrusion | Medium | Speed issues, possibly needs Pressure Advance tuning |

---

### Set 2: Z Face and Bottom (PENDING)

*Analysis pending - images: cube_z.jpg, cube_bottom.jpg*

---

### Set 3: Orca Logo and Text (PENDING)

*Analysis pending - images: cube_orca_logo.jpg, cube_orcaslicer_text.jpg*

---

## Identified Problems

### 1. No Purge Line
**Impact:** High  
**Description:** Without a purge line, the nozzle starts printing the actual model with inconsistent extrusion. The first few centimeters of filament may be under-extruded or have air gaps.

**Fix Required:** Add purge line to PRINT_START macro or OrcaSlicer start G-code

### 2. First Layer Speed Too Fast
**Impact:** High  
**Description:** Default speeds caused poor bed adhesion, requiring manual 50% speed reduction during print.

**Fix Required:** 
- Reduce first layer speed in OrcaSlicer (suggest 20-25mm/s)
- Increase first layer line width (120%) for better adhesion

### 3. Diagonal Banding/Rippling
**Impact:** Medium  
**Description:** Visible on both X and Y faces, suggesting mechanical or extrusion issues.

**Possible Causes:**
- Print speed too fast (ringing/ghosting)
- Pressure Advance not tuned
- Belt tension issues
- Input shaper not configured

**Fix Required:** 
- Tune Pressure Advance
- Consider Input Shaper calibration (requires accelerometer)
- Check belt tension

---

## Recommended Fixes

### Immediate (Before Next Print)

1. **Add purge line to start G-code**
   - Either in PRINT_START macro in printer.cfg
   - Or in OrcaSlicer Machine Start G-code

2. **Adjust OrcaSlicer first layer settings:**
   - First layer speed: 20-25 mm/s
   - First layer line width: 120% (0.48mm for 0.4mm nozzle)
   - First layer height: 0.2-0.25mm

3. **Reduce overall print speeds** until calibration complete

### Future Calibration

1. **Pressure Advance tuning** - Will improve extrusion consistency
2. **Input Shaper** - Requires ADXL345 accelerometer, will reduce ringing
3. **Flow rate calibration** - Fine-tune extrusion multiplier

---

## Files to Modify

### Option A: Add Purge Line to PRINT_START Macro

Location: `~/printer_data/config/printer.cfg` or macros config file

```gcode
[gcode_macro PRINT_START]
gcode:
    # ... existing preheat code ...
    
    # Purge line
    G1 Z5 F3000                    ; Move Z up
    G1 X5 Y20 F5000                ; Move to purge start position
    G1 Z0.3 F1000                  ; Lower to first layer height
    G1 X5 Y150 E15 F1500           ; Draw purge line
    G1 X5.4 Y150 F5000             ; Small move
    G1 X5.4 Y20 E15 F1500          ; Draw second purge line
    G1 E-0.5 F3000                 ; Small retract
    G1 Z5 F3000                    ; Lift Z
    G92 E0                         ; Reset extruder
    
    # ... continue with print ...
```

### Option B: Add Purge Line to OrcaSlicer Start G-code

In OrcaSlicer: Printer Settings > Machine G-code > Machine start G-code

Add after heating commands, before print starts.

---

## Current Status

- [DONE] X face analyzed
- [DONE] Y face analyzed
- [PENDING] Z face analysis
- [PENDING] Bottom face analysis
- [PENDING] Orca logo face analysis
- [PENDING] OrcaSlicer text face analysis
- [PENDING] Implement purge line fix
- [PENDING] Adjust first layer settings
- [PENDING] Re-print calibration cube

---

## Related Images

| Image | Description | Status |
|-------|-------------|--------|
| cube_x.jpg | X face of calibration cube | Analyzed |
| cube_y.jpg | Y face of calibration cube | Analyzed |
| cube_z.jpg | Z face of calibration cube | Pending |
| cube_bottom.jpg | Bottom of calibration cube | Pending |
| cube_orca_logo.jpg | Orca logo face | Pending |
| cube_orcaslicer_text.jpg | OrcaSlicer text face | Pending |

---

**Next Session:** Continue analysis with remaining cube images, then implement fixes.
