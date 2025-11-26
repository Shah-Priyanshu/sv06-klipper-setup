# Speed Tuning

**Date:** 2025-11-26  
**Status:** [DONE] COMPLETED

---

## Summary

Updated printer speed settings based on user testing. The original conservative defaults were too slow; new values provide faster prints while maintaining quality.

---

## Changes Made

### 1. printer.cfg - Default Speed Limits

**File:** `~/printer_data/config/printer.cfg`

| Setting | Old Value | New Value |
|---------|-----------|-----------|
| max_velocity | 200 mm/s | 70 mm/s |
| max_accel | 3000 mm/s² | 7000 mm/s² |

**Rationale:** 
- Lower velocity (70 mm/s) with higher acceleration (7000 mm/s²) provides snappier movements
- User tested these settings via Mainsail runtime override and confirmed good print quality
- Values now persist across restarts

### 2. mainsail.cfg - Layer Change Speed Switching

**File:** `~/printer_data/config/mainsail.cfg`

Modified `SET_PRINT_STATS_INFO` macro to automatically switch from first-layer settings to normal settings after layer 1.

**Added code:**
```jinja2
{% if params.CURRENT_LAYER is defined and params.CURRENT_LAYER|int == 2 %}
  _NORMAL_SETTINGS
{% endif %}
```

**Behavior:**
- Layer 1: Uses `_FIRST_LAYER_SETTINGS` (25 mm/s, 1000 mm/s²) - called from `PRINT_START`
- Layer 2+: Uses `_NORMAL_SETTINGS` (70 mm/s, 7000 mm/s²) - triggered automatically on layer change

---

## Speed Profile Summary

| Phase | Velocity | Acceleration | Triggered By |
|-------|----------|--------------|--------------|
| First Layer | 25 mm/s | 1000 mm/s² | `PRINT_START` → `_FIRST_LAYER_SETTINGS` |
| Normal Printing | 70 mm/s | 7000 mm/s² | `SET_PRINT_STATS_INFO` → `_NORMAL_SETTINGS` |
| Print End | 70 mm/s | 7000 mm/s² | `PRINT_END` → `_NORMAL_SETTINGS` |

---

## Verification

```bash
# Check current velocity settings
curl -s http://localhost:7125/printer/objects/query?toolhead | python3 -c "import sys,json; d=json.load(sys.stdin); t=d['result']['status']['toolhead']; print('max_velocity:', t['max_velocity'], 'max_accel:', t['max_accel'])"
```

**Result:** `max_velocity: 70.0 max_accel: 7000.0`

---

## Notes

- These speeds work well without Input Shaper calibration
- If print quality issues (ringing/ghosting) appear at higher speeds, consider:
  - Adding ADXL345 accelerometer for Input Shaper tuning
  - Reducing max_accel to 5000-6000 mm/s²
- OrcaSlicer print speeds are still respected; these are firmware-enforced maximums
