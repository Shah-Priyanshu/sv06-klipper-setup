# Image Catalog

This document catalogs all screenshots and images in the printer configuration repository, including their purpose, context, and problem resolution.

---

## Image Index

| Filename | Date | Related Log | Issue | Status |
|----------|------|-------------|-------|--------|
| `start_end_gcode.png` | 2025-11-26 | 15-gcode-path-fix.md | PRINT_START macro parameters | [DONE] RESOLVED |
| `slicer_test_print.png` | 2025-11-26 | 16-orcaslicer-bed-size-fix.md | Bed size mismatch | [DONE] RESOLVED |
| `printer_gcode_viewer_tracking.png` | 2025-11-26 | 16-orcaslicer-bed-size-fix.md | Bed size mismatch | [DONE] RESOLVED |
| `screw_top.jpg` | 2025-11-26 | 17-first-test-print-analysis.md | Calibration cube screw analysis | [DONE] REFERENCE |
| `screw_side.jpg` | 2025-11-26 | 17-first-test-print-analysis.md | Calibration cube screw analysis | [DONE] REFERENCE |

---

## Image Details

### 1. start_end_gcode.png

**Date Taken:** 2025-11-26  
**Source:** OrcaSlicer - Machine G-code settings  
**Related Log:** [15-gcode-path-fix.md](../logs/15-gcode-path-fix.md)

**Description:**
Screenshot of OrcaSlicer's Machine G-code configuration panel showing the corrected start and end G-code settings.

**What It Shows:**
- **Machine start G-code:** `PRINT_START BED=[bed_temperature_initial_layer_single] HOTEND=[nozzle_temperature_initial_layer]`
- **Machine end G-code:** `PRINT_END`
- OrcaSlicer printer settings interface

**Problem Context:**
Initial file upload from OrcaSlicer failed with error:
```
Error evaluating 'gcode_macro PRINT_START:gcode': 
jinja2.exceptions.UndefinedError: 'dict object' has no attribute 'HOTEND'
```

**Root Cause:**
- OrcaSlicer start G-code was using standard G-code commands (M104, M140, G28, etc.)
- The `PRINT_START` macro in `misc-macros.cfg` expected `BED` and `HOTEND` parameters
- Parameters were not being passed, causing Jinja2 template error

**Resolution:**
- Changed OrcaSlicer start G-code to call `PRINT_START` macro with proper parameters
- Changed OrcaSlicer end G-code to call `PRINT_END` macro
- Macros now handle all heating, homing, bed mesh loading, and purge operations

**Status:** [DONE] RESOLVED - Macro parameters correctly configured

**Technical Details:**
- **Issue Type:** Configuration mismatch between slicer and Klipper macros
- **Fix Type:** Update slicer G-code to use Klipper macros
- **Files Modified:** OrcaSlicer printer profile (Windows PC)

---

### 2. slicer_test_print.png

**Date Taken:** 2025-11-26  
**Source:** OrcaSlicer - Slicer preview window  
**Related Log:** [16-orcaslicer-bed-size-fix.md](../logs/16-orcaslicer-bed-size-fix.md)

**Description:**
Screenshot of OrcaSlicer's build plate preview showing two test objects (square and circle) positioned on the virtual build plate.

**What It Shows:**
- OrcaSlicer build plate view with grid overlay
- Two objects: rectangular block and cylindrical shape
- Objects appear centered on the build plate
- Build plate grid showing bed dimensions
- OrcaSlicer branding visible at bottom

**Problem Context:**
Objects appeared centered in OrcaSlicer but showed off-center when G-code was loaded in Mainsail's G-code viewer.

**Root Cause:**
- OrcaSlicer bed size configured as **250mm x 250mm** (incorrect)
- Actual Sovol SV06 bed size: **220mm x 220mm**
- Objects sliced with coordinates assuming larger bed
- When viewed in Mainsail (which knows actual bed size), objects appeared shifted

**Resolution:**
- Changed OrcaSlicer bed size from 250mm to 220mm
- Objects now position correctly relative to actual bed dimensions
- Slicer preview matches Mainsail G-code viewer preview

**Status:** [DONE] RESOLVED - Bed size corrected to 220mm x 220mm

**Technical Details:**
- **Issue Type:** Bed dimension misconfiguration
- **Fix Type:** Update OrcaSlicer printer profile bed size
- **Before:** 250mm x 250mm
- **After:** 220mm x 220mm
- **Files Modified:** OrcaSlicer printer profile (Windows PC)

**Coordinate Impact:**
- Objects centered on 250mm bed: X=125, Y=125
- Should be centered on 220mm bed: X=110, Y=110
- Offset: 15mm in each direction

---

### 3. printer_gcode_viewer_tracking.png

**Date Taken:** 2025-11-26  
**Source:** Mainsail - G-code viewer/preview window  
**Related Log:** [16-orcaslicer-bed-size-fix.md](../logs/16-orcaslicer-bed-size-fix.md)

**Description:**
Screenshot of Mainsail's G-code viewer showing the toolpath preview of the same test objects from OrcaSlicer, but displayed with Klipper's coordinate system.

**What It Shows:**
- Mainsail G-code preview window with dark background
- Grid overlay showing bed dimensions
- Two objects (orange/red outlines): rectangular and circular
- Objects appear shifted towards upper-left quadrant
- Coordinate axes (red/green) at bottom-left origin (0,0)

**Problem Context:**
Same objects that appeared centered in OrcaSlicer (see `slicer_test_print.png`) now appear off-center in Mainsail's viewer.

**Root Cause:**
Same as `slicer_test_print.png` - bed size mismatch:
- OrcaSlicer generated G-code for 250mm bed
- Mainsail displays G-code using Klipper's configured bed size (223mm)
- Visual discrepancy revealed the configuration error

**Resolution:**
- Fixed OrcaSlicer bed size to 220mm
- Re-slicing with correct bed size produces G-code with proper coordinates
- Mainsail preview now matches OrcaSlicer preview

**Status:** [DONE] RESOLVED - Bed size corrected to 220mm x 220mm

**Technical Details:**
- **Issue Type:** Bed dimension misconfiguration causing preview mismatch
- **Fix Type:** Update OrcaSlicer printer profile bed size
- **Klipper Config:** position_max: 223mm (X/Y)
- **Corrected Slicer:** 220mm x 220mm bed size
- **Remaining Difference:** 3mm (acceptable margin)

**Visual Analysis:**
- Grid pattern shows Klipper's coordinate system
- Objects rendered in orange/red showing toolpath
- Position clearly shows offset from center
- Comparison with OrcaSlicer preview revealed the 30mm total discrepancy (15mm per axis)

---

### 4. screw_top.jpg

**Date Taken:** 2025-11-26  
**Source:** Physical photograph - calibration cube screw test piece  
**Related Log:** [17-first-test-print-analysis.md](../logs/17-first-test-print-analysis.md)

**Description:**
Top-down photograph of the screw test piece from the OrcaSlicer calibration cube, showing the screw head surface quality.

**What It Shows:**
- Flat top surface of screw head with visible layer lines
- Circular perimeter of screw head
- Minor stringing trailing off the left side
- Overall surface quality assessment

**Problem Context:**
Part of the first test print analysis to evaluate print quality and identify calibration issues.

**Analysis Results:**
- **Surface quality:** Good - flat and level with consistent layer lines
- **Circular geometry:** Maintained with slight roughness on perimeter
- **Stringing:** Minor, related to lack of purge line at print start
- **No major defects** on top surface

**Status:** [DONE] REFERENCE - Print quality documentation

**Technical Details:**
- **Issue Type:** Print quality analysis
- **Category:** Calibration/test print
- **Print Settings:** OrcaSlicer defaults, 50% speed factor applied mid-print

---

### 5. screw_side.jpg

**Date Taken:** 2025-11-26  
**Source:** Physical photograph - calibration cube screw test piece  
**Related Log:** [17-first-test-print-analysis.md](../logs/17-first-test-print-analysis.md)

**Description:**
Side profile photograph of the screw test piece showing thread geometry and layer quality.

**What It Shows:**
- Thread profile with multiple turns visible
- Thread grooves and peaks
- Base/flange area with first layer quality
- Stringing at bottom of screw
- Overall structural integrity

**Problem Context:**
Part of the first test print analysis to evaluate thread printing capability and identify tolerance issues.

**Analysis Results:**
- **Thread geometry:** Functional but not sharp - peaks are rounded
- **Stringing at base:** Significant, caused by no purge line at print start
- **First layer flange:** Rough and ragged edges
- **Thread depth:** Slightly shallow, contributes to tight tolerance
- **Layer adhesion:** Good throughout threaded section
- **Structural integrity:** Solid, screw is functional

**Root Cause:**
1. No purge line - caused stringing and rough first layer
2. Over-extrusion or insufficient cooling - caused rounded thread peaks
3. Material shrinkage - contributes to tight thread tolerance

**Recommended Fixes:**
1. Add purge line to start G-code
2. Increase part cooling for sharper details
3. Enable XY Hole Compensation (0.1-0.2mm)

**Status:** [DONE] REFERENCE - Print quality documentation

**Technical Details:**
- **Issue Type:** Print quality analysis
- **Category:** Calibration/test print (thread tolerance test)
- **Print Settings:** OrcaSlicer defaults, 50% speed factor applied mid-print
- **Thread tolerance:** Too tight, required pliers to insert into cube

---

## Image Usage Guidelines

### Referencing Images in Logs

When referencing these images in documentation:

```markdown
![Description](../images/filename.png)
```

**Example:**
```markdown
See the corrected G-code configuration:
![OrcaSlicer Start/End G-code](../images/start_end_gcode.png)
```

### Adding New Images

When adding new images to this folder:

1. **Use descriptive filenames:**
   - Good: `klipper_mcu_connection_error.png`
   - Bad: `screenshot1.png`

2. **Add entry to this catalog:**
   - Update the Image Index table
   - Add detailed section with all fields

3. **Required Information:**
   - Date taken
   - Source application/interface
   - Related log file
   - Problem context
   - What the image shows
   - Root cause (if applicable)
   - Resolution (if applicable)
   - Status (pending, resolved, reference)

4. **Commit with descriptive message:**
   ```bash
   git add images/new_image.png images/IMAGE_CATALOG.md
   git commit -m "Add screenshot showing [specific issue/feature]"
   ```

---

## Image Organization

### Current Structure

```
printer-config/
├── images/
│   ├── IMAGE_CATALOG.md (this file)
│   ├── start_end_gcode.png
│   ├── slicer_test_print.png
│   ├── printer_gcode_viewer_tracking.png
│   ├── screw_top.jpg
│   └── screw_side.jpg
├── logs/
│   ├── 15-gcode-path-fix.md
│   ├── 16-orcaslicer-bed-size-fix.md
│   └── 17-first-test-print-analysis.md
└── configs/
```

### Categories

Images are categorized by:

**Configuration Screenshots:**
- `start_end_gcode.png` - Slicer G-code settings

**Problem Diagnosis:**
- `slicer_test_print.png` - Shows expected positioning
- `printer_gcode_viewer_tracking.png` - Shows actual positioning discrepancy

**Print Quality Analysis:**
- `screw_top.jpg` - Screw head top surface quality
- `screw_side.jpg` - Thread geometry and layer quality

**Future Categories:**
- Calibration results (bed mesh visualizations, etc.)
- Hardware modifications
- Web interface configurations
- Error messages and logs

---

## Technical Specifications

### Image Formats

All images currently in PNG format for:
- Lossless quality
- Screenshot compatibility
- Git-friendly (binary files tracked with LFS if needed)

### Resolution Guidelines

- **Screenshots:** Native resolution (as captured)
- **Minimum readable text:** 1280x720
- **Maximum file size:** Keep under 5MB per image

### Naming Conventions

**Pattern:** `source_description_context.png`

**Examples:**
- `orcaslicer_bed_size_settings.png`
- `mainsail_gcode_preview_offset.png`
- `klipper_console_error_mcu_disconnect.png`

---

## Related Documentation

- [Log 15: G-Code Upload Fix](../logs/15-gcode-path-fix.md) - PRINT_START macro configuration
- [Log 16: Bed Size Fix](../logs/16-orcaslicer-bed-size-fix.md) - OrcaSlicer bed dimension correction
- [Log 17: First Test Print Analysis](../logs/17-first-test-print-analysis.md) - Calibration cube quality analysis
- [Log 13: OrcaSlicer Setup](../logs/13-orcaslicer-setup.md) - Initial OrcaSlicer configuration

---

## Change Log

### 2025-11-26
- Created IMAGE_CATALOG.md
- Organized existing images into dedicated folder
- Cataloged 3 images related to OrcaSlicer configuration issues
- Documented bed size fix (250mm → 220mm)
- Documented PRINT_START macro parameter fix
- Added screw_top.jpg and screw_side.jpg from calibration cube analysis
- Added "Print Quality Analysis" category

---

**Last Updated:** 2025-11-26  
**Total Images:** 5  
**Issues Documented:** 3  
**All Issues Status:** [DONE] RESOLVED/REFERENCE
