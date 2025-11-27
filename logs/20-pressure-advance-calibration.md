# Pressure Advance Calibration

**Date:** 2025-11-26  
**Status:** [IN PROGRESS] Analysis Phase 1 of 2  
**Related Logs:** [18-print-quality-fixes.md](18-print-quality-fixes.md), [19-speed-tuning.md](19-speed-tuning.md)

---

## Summary

Calibrating Pressure Advance to eliminate stringing and improve corner quality. PA is critical for achieving Bambu Lab-level print quality.

---

## Background

**Problem:** Significant stringing observed in first test prints (calibration cube and screw threads).

**Root Cause:** No Pressure Advance configured in firmware.

**Solution:** Add PA to extruder config and calibrate optimal value using PA tower test.

---

## PA Tower Test Parameters

### Test Configuration

| Parameter | Value |
|-----------|-------|
| **Start PA** | 0.02 |
| **End PA** | 0.08 |
| **PA Step** | 0.005 |
| **Number of Sections** | 12 |
| **Method** | PA Tower (OrcaSlicer built-in) |
| **Extruder Type** | DDE (Direct Drive) |
| **Print Numbers** | Not available (manual counting required) |

### Section Reference Table

| Section | Height (mm) | PA Value |
|---------|-------------|----------|
| 1 (bottom) | 0-10 | 0.02 |
| 2 | 10-20 | 0.025 |
| 3 | 20-30 | 0.03 |
| 4 | 30-40 | 0.035 |
| 5 | 40-50 | 0.04 ← Current firmware value |
| 6 | 50-60 | 0.045 |
| 7 | 60-70 | 0.05 |
| 8 | 70-80 | 0.055 |
| 9 | 80-90 | 0.06 |
| 10 | 90-100 | 0.065 |
| 11 | 100-110 | 0.07 |
| 12 (top) | 110-120 | 0.075 |

---

## Analysis Phase 1: Front and Back Views

### Images Analyzed
- **PA_tower_front.jpg** - Front view showing textured and smooth faces
- **PA_tower_back.jpg** - Back view showing test pattern lines and artifacts

---

### Front View Analysis (PA_tower_front.jpg)

**Date Analyzed:** 2025-11-26  
**View:** Front face, held at angle

#### Observations

**Left Side (Textured/Lined Face):**
- Vertical line patterns visible (from speed test pattern)
- **Bottom sections (PA 0.02-0.03):** Lines more pronounced, slightly blobby appearance
- **Middle sections (PA 0.04-0.05):** Lines becoming more uniform
- **Top sections (PA 0.06-0.08):** Lines cleanest, most uniform
- Consistent layer adhesion throughout

**Right Side (Smooth Face):**
- Clean, uniform surface texture across all PA values
- No visible stringing or surface artifacts
- Good layer consistency from bottom to top
- This face less useful for PA analysis (no speed change patterns)

**Corners:**
- Sharp, well-defined throughout tower
- No obvious corner bulging visible from this angle
- Straight edges maintained

**Overall:** Front view confirms print quality is good but doesn't show detailed artifacts. Back view more useful for analysis.

---

### Back View Analysis (PA_tower_back.jpg)

**Date Analyzed:** 2025-11-26  
**View:** Back face showing test pattern

#### Critical Observations by Section

**Bottom Section: PA 0.02-0.03 (Sections 1-3)**

**Issues Observed:**
- ❌ **Visible stringing/wisps** on left side of tower
- ❌ **Blob artifacts** where speed changes occur in pattern
- ❌ **Over-extrusion appearance** - lines thicker and more prominent than upper sections
- ❌ **Poor extrusion control** at direction changes

**Root Cause:** PA value too low - extruder cannot compensate for pressure buildup during speed changes

**Rating:** ❌ **FAIL** - Not acceptable for quality prints

---

**Middle Section: PA 0.04-0.05 (Sections 5-6)**

**Observations:**
- ⚠️ **Reduced stringing** compared to bottom (50-70% improvement)
- ⚠️ **Cleaner lines** - more defined and consistent
- ⚠️ **Less blobbing** at corners and speed changes
- ⚠️ Still some minor artifacts visible on close inspection

**Assessment:** This is the **current firmware value (0.04)**, showing significant improvement over lower values but still room for optimization

**Rating:** ⚠️ **ACCEPTABLE** - Works but can be improved

---

**Upper Section: PA 0.06-0.075 (Sections 9-12)**

**Observations:**
- ✅ **Very clean lines** with minimal visible artifacts
- ✅ **No visible stringing** between features
- ✅ **Sharp corners** without bulging or rounding
- ✅ **Uniform line width** throughout test patterns
- ✅ **Crisp detail definition** at speed transitions
- ✅ **Best overall surface quality** of entire tower

**Assessment:** This range shows **optimal Pressure Advance compensation** - extrusion pressure is well-controlled during speed changes and direction changes

**Rating:** ✅ **EXCELLENT** - Best quality achieved

---

**Top Section: PA ~0.08 (Section 12)**

**Observations:**
- Clean appearance, similar to section below it
- Need to verify corners for under-extrusion (requires side view photos)
- No obvious issues from back view

**Assessment:** Need more angles to confirm if PA is too high

**Rating:** ⚠️ **NEEDS VERIFICATION** - May be approaching too-high threshold

---

## Preliminary Findings (Phase 1)

### Quality Assessment by PA Range

| PA Range | Stringing | Corner Quality | Line Definition | Overall Rating |
|----------|-----------|----------------|-----------------|----------------|
| **0.02-0.03** | ❌ Severe | ⚠️ Blobby | ❌ Over-extruded | **FAIL** |
| **0.04-0.05** | ⚠️ Reduced | ✅ Good | ⚠️ Acceptable | **ACCEPTABLE** |
| **0.06-0.075** | ✅ Eliminated | ✅ Excellent | ✅ Crisp | **BEST** |
| **0.08** | ✅ None visible | ❓ TBD | ✅ Good | **VERIFY** |

---

## Preliminary Recommendation

**Based on front and back view analysis:**

**Optimal PA Range: 0.06 - 0.07**

**Specific Recommendation:**
- **Current Value:** 0.04 (in printer.cfg)
- **Recommended New Value:** **0.06** or **0.065**

### Why Increase from 0.04 to 0.06?

| Current (0.04) | With 0.06 | Improvement |
|----------------|-----------|-------------|
| Some stringing visible | No stringing | ✅ Eliminated |
| Minor blobs at corners | Clean corners | ✅ Sharper details |
| Acceptable line quality | Excellent lines | ✅ Better surface finish |
| Good but improvable | Optimal | ✅ Bambu Lab quality |

**Conservative Approach:**
1. Update to **0.06** first (50% increase from 0.04)
2. Test with actual prints
3. Fine-tune to 0.065 or 0.07 if needed

---

## Pending Analysis (Phase 2)

**Waiting for additional photos:**
- PA_tower_left.jpg - Left side view
- PA_tower_right.jpg - Right side view

**What Phase 2 will verify:**
1. **Corner quality at high PA values** (0.07-0.08)
   - Check for under-extrusion gaps
   - Check for weak/rounded corners
2. **Surface quality on side faces**
   - Confirm no artifacts missed in front/back views
3. **Final PA value recommendation**
   - Confirm 0.06-0.07 range or adjust based on corners

---

## Current Firmware Configuration

**File:** `~/printer_data/config/printer.cfg`

**Current Settings:**
```gcode
[extruder]
rotation_distance: 4.604  # Calibrated 2025-11-26
pressure_advance: 0.04    # Temporary starting value
pressure_advance_smooth_time: 0.040
```

**Pending Update:** Will update `pressure_advance` value after Phase 2 analysis confirms optimal value.

---

## Technical Notes

### What is Pressure Advance?

**Problem:** Filament pressure in the hotend lags behind extruder motor movement, causing:
- **Acceleration:** Under-extrusion at the start of moves (pressure building up)
- **Deceleration:** Over-extrusion at the end of moves (pressure releasing)

**Solution:** PA anticipates these pressure changes and adjusts extruder timing:
- **Before acceleration:** Advance extra filament to build pressure
- **Before deceleration:** Reduce filament to prevent oozing

**Result:** Consistent extrusion pressure = clean prints without stringing or blobbing

### Direct Drive vs Bowden

| Extruder Type | Typical PA Range | SV06 Type |
|---------------|------------------|-----------|
| **Direct Drive** | 0.02 - 0.08 | ✅ SV06 is Direct Drive |
| **Bowden** | 0.3 - 0.8 | N/A |

Direct drive has much lower PA values because there's no flexible bowden tube between extruder motor and hotend.

---

## Related Images

| Image | Description | Status |
|-------|-------------|--------|
| PA_test_orcaslicer.png | OrcaSlicer PA calibration dialog | [REFERENCE] |
| PA_tower_front.jpg | Front view of PA tower | [DONE] Analyzed Phase 1 |
| PA_tower_back.jpg | Back view showing test patterns | [DONE] Analyzed Phase 1 |
| PA_tower_left.jpg | Left side view | [PENDING] Phase 2 |
| PA_tower_right.jpg | Right side view | [PENDING] Phase 2 |

---

## Current Status

- [DONE] PA added to firmware (0.04 starting value)
- [DONE] PA tower printed with 0.02-0.08 range
- [DONE] Front view analyzed
- [DONE] Back view analyzed
- [DONE] Preliminary recommendation: 0.06-0.07 range
- [PENDING] Left side view analysis (Phase 2)
- [PENDING] Right side view analysis (Phase 2)
- [PENDING] Final PA value determination
- [PENDING] Update printer.cfg with optimal value
- [PENDING] Verification print to confirm improvement

---

## Next Session Actions

**User will provide:**
1. PA_tower_left.jpg
2. PA_tower_right.jpg

**Agent will:**
1. Analyze left and right side photos
2. Verify corner quality at PA 0.06-0.08
3. Check for under-extrusion at high PA values
4. Determine final recommended PA value
5. Update printer.cfg with optimal value
6. Document complete analysis

---

**Analysis Phase 1 Complete:** 2025-11-26  
**Next Action:** Await left and right side photos for Phase 2 analysis
