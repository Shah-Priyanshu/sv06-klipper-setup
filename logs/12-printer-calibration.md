# Printer Calibration

**Date:** 2025-11-25  
**Status:** ✅ COMPLETED (Essential Calibrations)  
**Prerequisites:** ✅ Klipper, Moonraker, Mainsail, and camera streaming functional

---

## Calibration Summary

✅ **Essential Calibrations Completed:**
- **PID Tune Bed** - Temperature control optimized
- **PID Tune Hotend** - Nozzle temperature control optimized
- **Bed Mesh** - Surface mapping for auto-leveling
- **Z-Offset** - First layer height calibrated

⏸️ **Advanced Calibrations (Deferred):**
- Input Shaper (requires ADXL345 accelerometer)
- Pressure Advance tuning
- Flow rate calibration
- Temperature towers

---

## Calibration Results

### 1. Bed PID Tuning

**Target Temperature:** 60°C (PLA baseline)  
**Duration:** ~15 minutes

**Command Used:**
```
PID_CALIBRATE HEATER=heater_bed TARGET=60
```

**Results:**
- `pid_Kp = 69.167`
- `pid_Ki = 1.210`
- `pid_Kd = 988.229`

**Status:** ✅ Saved to printer.cfg

**What This Does:**
Calibrates the PID control algorithm for the heated bed, ensuring stable temperature without overshooting or oscillating. These values tell Klipper how aggressively to heat and how to maintain temperature.

---

### 2. Hotend PID Tuning

**Target Temperature:** 200°C (PLA standard)  
**Duration:** ~5 minutes

**Command Used:**
```
PID_CALIBRATE HEATER=extruder TARGET=200
```

**Results:**
- `pid_Kp = 23.358`
- `pid_Ki = 1.455`
- `pid_Kd = 93.723`

**Status:** ✅ Saved to printer.cfg

**What This Does:**
Calibrates temperature control for the hotend/nozzle. Ensures accurate and stable extrusion temperature, critical for consistent print quality.

---

### 3. Bed Mesh Calibration

**Probe Points:** 5x5 grid (25 points)  
**Duration:** ~5 minutes  
**Probe Type:** CR-Touch (inductive probe)

**Command Used:**
```
BED_MESH_CALIBRATE
```

**Mesh Data:**
```
[bed_mesh default]
version = 1
points =
    0.042500, 0.020469, 0.105781, 0.101719, 0.026094
    -0.054063, -0.022031, 0.101875, 0.135469, 0.086094
    -0.217969, -0.158438, 0.007500, 0.117969, 0.150156
    -0.387344, -0.265000, -0.044531, 0.123281, 0.250312
    -0.564531, -0.345469, -0.070156, 0.164219, 0.400625
x_count = 5
y_count = 5
mesh_x_pps = 2
mesh_y_pps = 2
algo = lagrange
tension = 0.2
min_x = 27.0
max_x = 222.0
min_y = 6.0
max_y = 203.0
```

**Status:** ✅ Saved to printer.cfg as "default" profile

**Mesh Analysis:**
- **Range:** -0.565mm (back-left corner) to +0.401mm (back-right corner)
- **Total variance:** ~0.97mm across the bed
- **Pattern:** Bed slopes down from right to left, with back-left being lowest point
- **Center point:** Nearly level (0.0075mm)

**What This Does:**
Maps the bed surface height at 25 points. Klipper uses this map to automatically adjust Z height during printing, compensating for bed unevenness. This ensures consistent first layer across the entire bed.

---

### 4. Z-Offset Calibration

**Method:** Paper test with heated nozzle and bed  
**Duration:** ~10 minutes

**Pre-heating:**
```
M104 S200  # Heat nozzle to 200°C
M140 S60   # Heat bed to 60°C
```

**Command Used:**
```
PROBE_CALIBRATE
```

**Process:**
1. Printer homed all axes
2. Nozzle moved to bed center
3. Manual adjustment using paper feeler gauge
4. Adjusted until paper had slight drag/friction
5. Accepted calibration with `ACCEPT` command

**Result:**
- `z_offset = 1.755`

**Status:** ✅ Saved to printer.cfg

**What This Does:**
Sets the exact distance between the probe trigger point and the nozzle tip when it touches the bed. This is THE most critical calibration for first layer adhesion. Value of 1.755mm means the nozzle is 1.755mm lower than where the probe triggers.

---

## Configuration File Changes

### Auto-Generated Section in printer.cfg

**Location:** `~/printer_data/config/printer.cfg`

**SAVE_CONFIG section appended:**
```ini
#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
#*# [heater_bed]
#*# control = pid
#*# pid_kp = 69.167
#*# pid_ki = 1.210
#*# pid_kd = 988.229
#*#
#*# [extruder]
#*# control = pid
#*# pid_kp = 23.358
#*# pid_ki = 1.455
#*# pid_kd = 93.723
#*#
#*# [bed_mesh default]
#*# version = 1
#*# points =
#*#     0.042500, 0.020469, 0.105781, 0.101719, 0.026094
#*#     -0.054063, -0.022031, 0.101875, 0.135469, 0.086094
#*#     -0.217969, -0.158438, 0.007500, 0.117969, 0.150156
#*#     -0.387344, -0.265000, -0.044531, 0.123281, 0.250312
#*#     -0.564531, -0.345469, -0.070156, 0.164219, 0.400625
#*# x_count = 5
#*# y_count = 5
#*# mesh_x_pps = 2
#*# mesh_y_pps = 2
#*# algo = lagrange
#*# tension = 0.2
#*# min_x = 27.0
#*# max_x = 222.0
#*# min_y = 6.0
#*# max_y = 203.0
#*#
#*# [probe]
#*# z_offset = 1.755
```

**Note:** This section is automatically managed by Klipper's `SAVE_CONFIG` command. Do not manually edit.

---

## Calibration Workflow Summary

### Total Time: ~40 minutes

**Step 1: PID Tune Bed** (~15 min)
```
PID_CALIBRATE HEATER=heater_bed TARGET=60
SAVE_CONFIG
```

**Step 2: PID Tune Hotend** (~5 min)
```
PID_CALIBRATE HEATER=extruder TARGET=200
SAVE_CONFIG
```

**Step 3: Bed Mesh** (~5 min)
```
BED_MESH_CALIBRATE
SAVE_CONFIG
```

**Step 4: Z-Offset** (~10 min)
```
M104 S200
M140 S60
PROBE_CALIBRATE
# Manual adjustment with paper
ACCEPT
SAVE_CONFIG
```

**Step 5: Test Print** (Skipped - moving to OrcaSlicer setup)

---

## Values for Slicer Configuration

When setting up OrcaSlicer (or any slicer), use these calibrated values:

### Temperature Control:
- **Bed PID:** Kp=69.167, Ki=1.210, Kd=988.229
- **Hotend PID:** Kp=23.358, Ki=1.455, Kd=93.723
- **Bed Temp (PLA):** 60°C (baseline, may need adjustment per filament)
- **Nozzle Temp (PLA):** 200°C (baseline, may need adjustment per filament)

### Bed Leveling:
- **Bed Mesh:** Enabled (auto-loaded as "default" profile)
- **Z-Offset:** 1.755mm (already applied, don't add to slicer)
- **Bed Size:** 223mm x 203mm (approximate from mesh bounds)

### Print Settings:
- **First Layer Height:** 0.2mm recommended (20% of 0.4mm nozzle)
- **First Layer Speed:** 20-30 mm/s recommended
- **Bed Adhesion:** Mesh compensation active

---

## Verification Commands

```bash
# View current bed mesh
BED_MESH_OUTPUT

# Check PID values
GET_THERMAL_PARAMETERS HEATER=heater_bed
GET_THERMAL_PARAMETERS HEATER=extruder

# Check Z-offset
GET_POSITION

# Verify saved config
cat ~/printer_data/config/printer.cfg | grep -A50 "SAVE_CONFIG"
```

---

## Notes and Observations

### Bed Condition:
- Bed has noticeable tilt (back-left corner ~0.5mm lower than back-right)
- This is within normal range for budget printers
- Bed mesh compensation will handle this automatically
- Consider manual bed screw adjustment if variance exceeds 1mm

### Temperature Stability:
- PID values should provide stable heating
- May need re-tuning if ambient temperature changes significantly
- Watch for temperature oscillations during first prints

### Z-Offset:
- 1.755mm offset is typical for CR-Touch probes
- May need minor adjustment (±0.05mm) based on first print results
- Use "Babystepping" in Mainsail during first layer if needed
- Fine-tune using `SET_GCODE_OFFSET Z_ADJUST=±0.05 MOVE=1`

---

## Advanced Calibrations (Future)

### Input Shaper (Resonance Compensation)
**Requires:** ADXL345 accelerometer module  
**Purpose:** Eliminate ringing/ghosting artifacts  
**Time:** ~30 minutes  
**Priority:** Medium - improves quality at high speeds

### Pressure Advance
**Purpose:** Eliminate bulging corners and gaps  
**Time:** ~1 hour (includes test prints)  
**Priority:** High - significant quality improvement  
**Range:** Typically 0.02 - 0.10 for direct drive

### Flow Rate Calibration
**Purpose:** Perfect extrusion amount  
**Time:** ~1 hour (includes test prints)  
**Priority:** Medium - fine-tunes dimensional accuracy  
**Method:** Single-wall cube test

### Temperature Towers
**Purpose:** Find optimal temperature per filament  
**Time:** ~30 min per filament type  
**Priority:** Low - only needed for new filament brands

---

## Troubleshooting

### Bed PID Issues
**Symptom:** Temperature oscillates ±5°C or more  
**Solution:** Re-run PID tune with different target (50-70°C range)

### Hotend PID Issues  
**Symptom:** "PID drift" errors or temperature spikes  
**Solution:** Re-run PID tune, check thermistor connection

### Bed Mesh Problems
**Symptom:** First layer inconsistent across bed  
**Solution:** 
- Re-run `BED_MESH_CALIBRATE`
- Clean bed surface thoroughly
- Verify CR-Touch probe is working: `PROBE_ACCURACY`

### Z-Offset Too High/Low
**Symptom:** 
- Too high: Filament not sticking, spaghetti print
- Too low: Nozzle scraping bed, no extrusion

**Solution:**
```
# Adjust on the fly during print
SET_GCODE_OFFSET Z_ADJUST=-0.05 MOVE=1  # Lower nozzle
SET_GCODE_OFFSET Z_ADJUST=+0.05 MOVE=1  # Raise nozzle

# Re-calibrate if more than ±0.2mm adjustment needed
PROBE_CALIBRATE
```

---

## Post-Calibration Checklist

- [x] Bed PID tuned and saved
- [x] Hotend PID tuned and saved
- [x] Bed mesh created and saved
- [x] Z-offset calibrated and saved
- [x] All values verified in printer.cfg
- [ ] Test print completed (skipped)
- [ ] First layer adhesion verified (will verify with OrcaSlicer)
- [ ] Slicer configured with calibrated values (next step)

---

## Next Steps

1. ✅ Essential calibrations complete
2. ⏳ **Install OrcaSlicer on Debian machine**
3. ⏳ Configure printer profile with calibrated values
4. ⏳ Set up Moonraker connection for direct upload
5. ⏳ First real print to verify calibration

**Printer is now ready for slicing and printing!**
