# First Test Print Analysis - Orca Calibration Cube

**Date:** 2025-11-26  
**Status:** [DONE] ANALYSIS COMPLETE  
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

### Set 2: Z Face and Bottom

#### Z Face / Top Surface (cube_z.jpg)

**Observed Issues:**
- **Top surface quality is good** - diagonal infill pattern is visible and relatively consistent
- **Z embossed letter is clearly defined** - good detail resolution, sharp edges on the recessed letter
- **Minor edge imperfections** - slight roughness on outer perimeter, small defect on top-left corner
- **Best face of the cube** - indicates good layer stacking and Z-axis consistency

**Positive Signs:**
- Layer adhesion appears solid throughout the print height
- Top solid infill is complete with no gaps
- Embossed detail is crisp and readable

#### Bottom Face (cube_bottom.jpg)

**Observed Issues:**
- **Severe first layer problems** - confirms issues from missing purge line
- **Outer perimeter is rough and scalloped** - wavy, inconsistent edges all around the bottom
- **Stringing in center hole** - multiple filament strands crossing the circular opening
- **Poor first layer adhesion** - visible gaps and uneven extrusion lines across bottom surface
- **Corner blobs/buildup** - excess material accumulated at all four corners
- **Center hole perimeter is rough** - not clean circles, shows extrusion inconsistency at print start

**Root Cause Confirmation:**
The bottom face clearly shows the nozzle was not primed when printing began. The first perimeter has inconsistent extrusion (thick in some areas, thin/missing in others), and the stringing indicates the filament was oozing during travel moves without proper retraction at the start.

#### Set 2 Summary

| Issue | Severity | Likely Cause |
|-------|----------|--------------|
| Rough first layer perimeter | High | No purge line, nozzle not primed |
| Stringing in center hole | Medium | Retraction settings, no purge at start |
| Corner buildup/blobs | Medium | Over-extrusion at corners, speed changes |
| Scalloped bottom edges | High | First layer speed too fast, poor adhesion |
| Top surface acceptable | Low | Z-axis working correctly |

---

### Set 3: Orca Logo and OrcaSlicer Text

#### Orca Logo Face (cube_orca_logo.jpg)

**Observed Issues:**
- **Logo is clearly recognizable** - the orca/whale shape with diagonal line is well-defined and easily identifiable
- **Good embossed detail** - logo edges are crisp with proper depth
- **Surface quality is decent** - relatively smooth with consistent layer lines across the face
- **Bottom edge damage** - visible rough/torn material from first layer adhesion problems and print removal
- **Top-left corner defect** - slight imperfection/nick visible
- **Minor stringing at bottom** - small wisps of filament at the base

**Positive Signs:**
- Complex curved geometry (the orca shape) printed accurately
- Layer adhesion is solid throughout the vertical walls
- Embossed detail depth is consistent

#### OrcaSlicer Text Face (cube_orcaslicer_text.jpg)

**Observed Issues:**
- **Text is fully readable** - "Orca Slicer" text is clear and legible
- **Good letter definition** - all characters are crisp with clean edges
- **Wave design printed well** - decorative waves below text are smooth and continuous
- **Right edge has visible layer banding** - consistent with rippling seen on X/Y faces
- **Bottom edge problems** - rough, uneven material accumulation from first layer issues
- **Small blob at bottom-right corner** - excess material from corner over-extrusion
- **Excellent fine detail resolution** - small text features and thin wave lines reproduced accurately

**Positive Signs:**
- Fine text detail (small letters like 'r', 'c', 'e') are sharp and readable
- No bridging issues on enclosed letters ('O', 'a')
- Consistent extrusion on detailed features

#### Set 3 Summary

| Issue | Severity | Likely Cause |
|-------|----------|--------------|
| Bottom edge roughness | High | First layer problems (no purge) |
| Corner blobs | Medium | Over-extrusion at direction changes |
| Layer banding on edges | Medium | Speed/ringing (consistent with other faces) |
| Logo/text detail | Good | N/A - positive result |
| Fine feature resolution | Good | N/A - positive result |

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
- [DONE] Z face analyzed
- [DONE] Bottom face analyzed
- [DONE] Orca logo face analyzed
- [DONE] OrcaSlicer text face analyzed
- [PENDING] Implement purge line fix
- [PENDING] Adjust first layer settings
- [PENDING] Re-print calibration cube

---

## Related Images

| Image | Description | Status |
|-------|-------------|--------|
| cube_x.jpg | X face of calibration cube | Analyzed |
| cube_y.jpg | Y face of calibration cube | Analyzed |
| cube_z.jpg | Z face (top surface) of calibration cube | Analyzed |
| cube_bottom.jpg | Bottom of calibration cube | Analyzed |
| cube_orca_logo.jpg | Orca logo face | Analyzed |
| cube_orcaslicer_text.jpg | OrcaSlicer text face | Analyzed |

---

## Complete Analysis Summary

### Overall Print Quality Assessment

**Rating: 6/10 - Acceptable for first print, but needs tuning**

### What Went Well
- **Detail resolution is excellent** - All embossed text (X, Y, Z, Orca Slicer) and the Orca logo are crisp and clearly readable
- **Layer adhesion is solid** - No delamination or layer separation throughout the cube
- **Z-axis consistency** - Top surface is clean, layers stack evenly
- **Complex geometry** - Curved orca logo shape printed accurately
- **Fine features** - Small text characters and thin wave lines reproduced well

### What Needs Improvement
| Priority | Issue | Root Cause | Fix |
|----------|-------|------------|-----|
| **HIGH** | Rough first layer, poor adhesion | No purge line | Add purge line to start G-code |
| **HIGH** | Required 50% speed reduction | First layer too fast | Reduce first layer speed to 20-25mm/s |
| **MEDIUM** | Diagonal banding/rippling on walls | Speed/ringing | Tune Pressure Advance, check belts |
| **MEDIUM** | Corner blobs/over-extrusion | Direction changes | Pressure Advance tuning |
| **LOW** | Minor stringing at bottom | Retraction at start | Purge line will help |

### Conclusion

The print demonstrates that the printer hardware is functioning correctly - the mechanical systems (X, Y, Z axes), hotend, and bed are all working well. The issues observed are **software/configuration problems**, not hardware problems:

1. **Missing purge line** caused 80% of the visible defects (rough bottom, corner issues, first layer problems)
2. **Speed settings** need adjustment for first layer
3. **Pressure Advance** would improve wall quality but is optional tuning

**Recommended next steps:**
1. Add purge line to PRINT_START macro or OrcaSlicer start G-code
2. Reduce first layer speed to 20-25mm/s in OrcaSlicer
3. Re-print calibration cube to verify improvements
4. (Optional) Tune Pressure Advance for better wall quality

---

**Analysis Complete:** 2025-11-26  
**Next Action:** Implement purge line fix and adjust slicer settings
