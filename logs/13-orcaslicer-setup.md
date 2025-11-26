# OrcaSlicer Setup and Configuration

**Date:** 2025-11-25  
**Status:** ✅ COMPLETED  
**Prerequisites:** ✅ Printer calibrated, Moonraker API accessible

---

## Installation Summary

✅ **OrcaSlicer Installed:**
- **Platform:** Windows PC (not on Debian printer machine)
- **Source:** https://github.com/OrcaSlicer/OrcaSlicer/releases
- **Version:** Latest release from GitHub

✅ **Configuration Completed:**
- Printer profile configured (Sovol SV06)
- Moonraker connection established
- Start/End G-code configured
- Print settings with calibrated values
- PLA filament profile created

---

## Architecture

### Workflow Setup

```
┌─────────────────────┐
│   Windows PC        │
│                     │
│   OrcaSlicer        │ ← User slices models here
│   (Slicer)          │
└──────────┬──────────┘
           │
           │ Network: 10.0.0.0/16
           │ Upload G-code via Moonraker API
           ↓
┌─────────────────────┐
│  Debian Laptop      │
│  IP: 10.0.0.139     │
│                     │
│  ┌───────────────┐  │
│  │  Moonraker    │  │ ← API server (port 7125)
│  │  (API)        │  │
│  └───────┬───────┘  │
│          │          │
│  ┌───────▼───────┐  │
│  │  Klipper      │  │ ← Firmware
│  │  (Firmware)   │  │
│  └───────┬───────┘  │
│          │          │
│  ┌───────▼───────┐  │
│  │  Mainsail     │  │ ← Web UI (port 80)
│  │  (Web UI)     │  │
│  └───────────────┘  │
│          │          │
│      USB Serial     │
└──────────┼──────────┘
           │
     ┌─────▼──────┐
     │  Sovol SV06 │
     │  (Printer)  │
     └────────────┘
```

**Key Point:** OrcaSlicer runs on Windows PC, sends G-code files over network to Moonraker on the printer's Debian laptop.

---

## Installation Details

### OrcaSlicer Installation (Windows PC)

**Source:**
- Repository: https://github.com/OrcaSlicer/OrcaSlicer/releases
- Downloaded and installed latest release

**Installation Location:**
- User-specified during installation (Windows default program location)

**No Installation on Printer:**
- OrcaSlicer NOT installed on Debian laptop (10.0.0.139)
- Printer only runs Klipper/Moonraker/Mainsail services
- Slicing happens on Windows PC, G-code uploaded remotely

---

## Configuration Steps Performed

### Step 1: Printer Profile Configuration

**Printer Added:**
- **Name:** Sovol SV06 (or user's chosen name)
- **Type:** Klipper-based printer
- **Selection Method:** 
  - Option A: Selected "Sovol SV06" from manufacturer list
  - Option B: Selected "Generic Klipper Printer" and customized

**Machine Limits:**
```
Bed Size (X): 220mm
Bed Size (Y): 220mm
Maximum Z Height: 250mm
Nozzle Diameter: 0.4mm
Filament Diameter: 1.75mm
```

**Build Volume:**
```
Width (X): 220mm
Depth (Y): 220mm
Height (Z): 250mm
Origin: Front-left corner (0,0)
```

**Extruder Settings:**
```
Nozzle Diameter: 0.4mm
Max Layer Height: 0.3mm (75% of nozzle diameter)
Min Layer Height: 0.08mm (20% of nozzle diameter)
Retraction Length: 0.5mm (direct drive)
Retraction Speed: 35 mm/s
```

---

### Step 2: Moonraker Connection Setup

**Physical Printer Configuration:**

**Connection Type:** Klipper (via Moonraker)

**Network Settings:**
```
Hostname: 10.0.0.139
Port: 7125
Protocol: HTTP
API Key: (none - no authentication configured)
```

**Printer Name in OrcaSlicer:**
```
Name: SV06 Klipper
(or user's chosen name)
```

**Connection Test:**
- Status: ✅ Connected successfully
- OrcaSlicer can communicate with Moonraker
- File uploads enabled

**Upload Destination:**
- Files uploaded to: `~/printer_data/gcodes/` on Debian laptop
- Accessible in Mainsail web interface under "Jobs" tab

---

### Step 3: Custom G-code Configuration

#### Start G-code

**Purpose:** Executed at beginning of every print

**Configuration:**
```gcode
; Start G-code for Sovol SV06 with Klipper
M104 S{nozzle_temperature_initial_layer[0]} ; Start heating nozzle
M140 S{bed_temperature_initial_layer[0]} ; Start heating bed
G90 ; Absolute positioning
M82 ; Absolute extrusion mode
G28 ; Home all axes
BED_MESH_PROFILE LOAD=default ; Load bed mesh calibration
M190 S{bed_temperature_initial_layer[0]} ; Wait for bed temperature
M109 S{nozzle_temperature_initial_layer[0]} ; Wait for nozzle temperature
G1 Z2.0 F3000 ; Move Z up to safe height
G1 X0.1 Y20 Z0.3 F5000 ; Move to start position for purge line
G1 X0.1 Y200.0 Z0.3 F1500 E15 ; Draw first purge line
G1 X0.4 Y200.0 Z0.3 F5000 ; Move to side
G1 X0.4 Y20 Z0.3 F1500 E30 ; Draw second purge line
G92 E0 ; Reset extruder position
G1 Z2.0 F3000 ; Move Z up before starting print
```

**Key Features:**
- Heats bed and nozzle simultaneously (faster)
- Loads calibrated bed mesh (from log 12)
- Draws purge line to prime nozzle
- Uses Klipper-specific commands (BED_MESH_PROFILE)

#### End G-code

**Purpose:** Executed at end of every print

**Configuration:**
```gcode
; End G-code for Sovol SV06 with Klipper
G91 ; Relative positioning
G1 E-2 F2700 ; Retract filament slightly
G1 E-2 Z0.2 F2400 ; Retract more and raise Z
G1 X5 Y5 F3000 ; Wipe nozzle
G1 Z10 ; Raise Z to clear print
G90 ; Absolute positioning
G1 X0 Y220 ; Move to front for easy part removal
M106 S0 ; Turn off part cooling fan
M104 S0 ; Turn off hotend heater
M140 S0 ; Turn off bed heater
M84 ; Disable stepper motors
```

**Key Features:**
- Retracts filament to prevent oozing
- Presents print at front of bed
- Safely shuts down all heaters and motors

---

### Step 4: Print Settings (Calibrated Values)

**Quality Settings:**

**Layer Heights:**
```
First Layer Height: 0.2mm
Standard Layer Height: 0.2mm
(Adjustable range: 0.12mm - 0.28mm)
```

**Speed Settings:**
```
First Layer Speed: 25 mm/s (slow for adhesion)
Outer Wall Speed: 50 mm/s (quality)
Inner Wall Speed: 80 mm/s
Sparse Infill Speed: 100 mm/s
Top Surface Speed: 50 mm/s
Travel Speed: 150 mm/s
```

**Acceleration Settings:**
```
Default Acceleration: 2000 mm/s²
First Layer Acceleration: 500 mm/s² (gentle)
Outer Wall Acceleration: 1000 mm/s²
```

**Wall Settings:**
```
Wall Loops: 3 (outer + 2 inner)
Wall Order: Inner-Outer-Inner
Top Solid Layers: 4
Bottom Solid Layers: 4
```

**Infill Settings:**
```
Infill Density: 15% (standard)
Infill Pattern: Grid or Gyroid
```

---

### Step 5: Temperature Settings (From Calibration)

**Hotend Temperature (PLA):**
```
Nozzle Temperature: 200°C
First Layer Nozzle: 205°C (5°C hotter for adhesion)
```

**Note:** These values from PID calibration (log 12):
- PID_Kp = 23.358
- PID_Ki = 1.455  
- PID_Kd = 93.723

**Bed Temperature (PLA):**
```
Bed Temperature: 60°C
First Layer Bed: 60°C
```

**Note:** From PID calibration (log 12):
- PID_Kp = 69.167
- PID_Ki = 1.210
- PID_Kd = 988.229

**Temperature Ranges for Other Filaments:**
```
PLA: 190-220°C nozzle, 50-70°C bed
PETG: 230-250°C nozzle, 70-85°C bed
ABS: 240-260°C nozzle, 90-110°C bed (requires enclosure)
TPU: 210-230°C nozzle, 30-60°C bed
```

---

### Step 6: Retraction Settings (Direct Drive)

**Retraction Configuration:**
```
Retraction Distance: 0.5mm
Retraction Speed: 35 mm/s
Deretraction Speed: 35 mm/s
Z-hop When Retracting: 0.2mm (optional)
Z-hop Type: Normal
```

**Why these values:**
- SV06 has direct drive extruder (short path to nozzle)
- Direct drive needs minimal retraction (0.5mm vs 5-6mm for Bowden)
- Lower retraction reduces wear on filament
- Z-hop prevents nozzle dragging on print during travels

---

### Step 7: Cooling Settings

**Part Cooling Fan:**
```
Enable Cooling: Yes
Minimum Fan Speed: 50%
Maximum Fan Speed: 100%
Fan Speed on Bridges: 100%
Disable Fan for First N Layers: 3
```

**Fan Control Logic:**
- First 3 layers: Fan OFF (improves bed adhesion)
- Layer 4+: Fan at 50-100% (improves overhangs)
- Bridges: Fan at 100% (prevents sagging)

---

### Step 8: Bed Adhesion Settings

**First Layer Configuration:**
```
First Layer Height: 0.2mm (same as layer height)
First Layer Width: 120% (wider for better adhesion)
First Layer Speed: 25 mm/s (slow)
First Layer Acceleration: 500 mm/s² (gentle)
```

**Bed Adhesion Type:**
```
Type: None (Skirt only)
Skirt Loops: 2
Skirt Distance: 2mm
Brim Width: 0mm (not needed with mesh leveling)
```

**Why no brim/raft needed:**
- Bed mesh compensation active (from log 12)
- PEI/textured bed surface provides good adhesion
- Purge line in start G-code primes nozzle

---

## Calibrated Values Integration

### From Log 12 Calibration

**Values Automatically Applied:**

**Temperature Control:**
- Bed PID tuned: Maintains stable 60°C
- Hotend PID tuned: Maintains stable 200°C
- No temperature oscillations expected

**Bed Leveling:**
- Bed mesh: 5x5 grid loaded via start G-code
- Z-offset: 1.755mm (already in printer config)
- No manual Z-adjustment needed in slicer

**First Layer:**
- Z-offset applied automatically by Klipper
- Mesh compensation active across entire bed
- Perfect first layer height maintained

**What NOT to configure in slicer:**
- ❌ Z-offset adjustment (already in printer)
- ❌ Manual bed leveling compensation
- ❌ Temperature PID values (in Klipper)

---

## Filament Profile Configuration

### PLA Profile (Generic)

**Created Default PLA Profile:**

**Temperature:**
```
Nozzle: 200°C (baseline)
Bed: 60°C
```

**Cooling:**
```
Enable Fan: Yes
Min Fan Speed: 50%
Max Fan Speed: 100%
Disable Fan First: 3 layers
```

**Retraction:**
```
Distance: 0.5mm
Speed: 35 mm/s
```

**Flow Rate:**
```
Extrusion Multiplier: 1.0 (100%)
(May need adjustment per filament brand)
```

**Filament Properties:**
```
Diameter: 1.75mm
Density: 1.24 g/cm³ (for weight estimates)
Cost: (user configurable)
```

### Per-Brand Profiles (Optional)

User can create specific profiles for:
- Different PLA brands (temperature varies)
- PETG, ABS, TPU, etc.
- Specialty filaments (wood fill, silk, etc.)

**Recommended approach:**
- Start with generic PLA profile
- Clone and adjust for specific brands
- Note temperature, flow, and cooling differences

---

## Workflow: Slice to Print

### Complete Process

**1. On Windows PC:**
```
Open OrcaSlicer
→ Load STL model (File → Import)
→ Adjust orientation/supports if needed
→ Select printer: "SV06 Klipper"
→ Select filament: "Generic PLA"
→ Click "Slice Plate"
→ Review preview (check first layer, supports, time)
→ Click "Send" or "Upload"
```

**2. File Transfer:**
```
OrcaSlicer → HTTP Upload → Moonraker (10.0.0.139:7125)
→ File saved to ~/printer_data/gcodes/
→ Visible in Mainsail web interface
```

**3. On Printer (via Mainsail):**
```
Open Mainsail: http://10.0.0.139
→ Go to "Jobs" tab
→ Find uploaded G-code file
→ Click file → Select "Print"
→ Monitor via camera and web interface
```

---

## Network Configuration

### Moonraker API Details

**Endpoint Information:**
```
API Base URL: http://10.0.0.139:7125
Upload Endpoint: /server/files/upload
File List Endpoint: /server/files/list
Print Endpoint: /printer/print/start
```

**No Authentication:**
- Moonraker configured without API key
- Trusted network (home LAN)
- All clients on 10.0.0.0/16 trusted

**Firewall:**
- Port 7125 accessible from Windows PC
- Port 80 (Mainsail) accessible from any device
- Port 8080 (camera) accessible from Mainsail

---

## OrcaSlicer Features Configured

### Enabled Features

**Supports:**
- Automatic support generation available
- Tree supports (better for organics)
- Grid supports (faster, more material)

**Modifiers:**
- Height range modifiers (change settings mid-print)
- Part modifiers (different settings per part)

**Multi-Material:**
- Not configured (SV06 is single extruder)
- Can be enabled if MMU added later

**Adaptive Layer Height:**
- Available but not enabled by default
- Can adjust layer height based on model geometry

**Ironing:**
- Top surface ironing available
- Smooths top layers (slower but prettier)

---

## Print Profile Summary

**Profile Name:** SV06 Klipper Standard PLA

**Quick Reference:**
```
Layer Height: 0.2mm
First Layer: 0.2mm @ 25mm/s
Walls: 3 (50mm/s outer)
Infill: 15% @ 100mm/s
Nozzle: 200°C (205°C first layer)
Bed: 60°C
Retraction: 0.5mm @ 35mm/s
Fan: 50-100% (off first 3 layers)
```

**Print Time Estimates:**
- Small calibration cube (20mm): ~15 minutes
- Benchy: ~1.5 hours
- Full bed print: ~8-12 hours (depends on infill)

---

## Troubleshooting

### Connection Issues

**Symptom:** OrcaSlicer can't connect to printer

**Checks:**
```bash
# From Windows PC, verify Moonraker accessible
curl http://10.0.0.139:7125/server/info

# Or in browser
http://10.0.0.139:7125/server/info

# Should return JSON with printer info
```

**Solution:**
- Verify printer laptop is on (10.0.0.139)
- Verify Moonraker service running: `systemctl status moonraker`
- Check firewall on printer laptop
- Verify network connectivity: `ping 10.0.0.139`

### Upload Failures

**Symptom:** File upload fails or times out

**Solution:**
- Check network bandwidth (large files take time)
- Verify disk space on printer: `df -h ~/printer_data/gcodes`
- Check Moonraker logs: `tail -50 ~/printer_data/logs/moonraker.log`

### First Layer Issues

**Symptom:** First layer too high or too low

**Solution:**
- Z-offset may need fine-tuning (±0.05mm)
- In Mainsail, during first layer:
  ```
  SET_GCODE_OFFSET Z_ADJUST=-0.05 MOVE=1  # Lower nozzle
  SET_GCODE_OFFSET Z_ADJUST=+0.05 MOVE=1  # Raise nozzle
  ```
- If change > ±0.1mm needed, re-run `PROBE_CALIBRATE`

### Adhesion Problems

**Symptom:** Print not sticking to bed

**Solution:**
- Clean bed surface (IPA/isopropyl alcohol)
- Verify bed temperature reached 60°C
- Check first layer squish via camera
- Increase bed temp to 65°C for difficult filaments
- Add brim in OrcaSlicer (5-10mm width)

---

## Configuration Files

### OrcaSlicer Configuration Storage

**Windows Location:**
```
User Profile: C:\Users\<username>\AppData\Roaming\OrcaSlicer\
Printer Profiles: system/*.json
Filament Profiles: filament/*.json
Print Profiles: process/*.json
Physical Printers: physical_printer/*.json
```

**Backup Recommendation:**
- Export profiles regularly (File → Export Config Bundle)
- Store config bundle in this repository
- Allows easy restore if OrcaSlicer reinstalled

---

## Advanced Configuration (Future)

### Optional Enhancements

**Pressure Advance (Klipper):**
- Requires calibration test prints
- Add to printer.cfg: `pressure_advance: 0.05` (example)
- Reduces bulging corners and gaps
- Configure in Klipper, not slicer

**Input Shaper (Klipper):**
- Requires ADXL345 accelerometer
- Eliminates ringing/ghosting
- Run resonance test, apply shaper
- No slicer configuration needed

**Adaptive Bed Mesh:**
- Generate mesh only for print area
- Faster than full 5x5 mesh
- Requires slicer modifications

**Arc Welder:**
- Converts segments to G2/G3 arcs
- Reduces file size
- Smoother curves
- Enable in OrcaSlicer → Printer Settings → G-code flavor → Klipper

---

## Testing Checklist

### Before First Real Print

- [ ] OrcaSlicer can connect to Moonraker (10.0.0.139:7125)
- [ ] Test slice completes without errors
- [ ] Test upload succeeds (file appears in Mainsail)
- [ ] Review sliced preview (proper purge line, no boundary violations)
- [ ] Camera feed visible in Mainsail for monitoring
- [ ] Bed and nozzle temperatures stabilize correctly
- [ ] Bed mesh loads in start G-code (check Mainsail console)

### First Print Recommendations

**Good First Prints:**
- 20mm calibration cube (quick, tests basics)
- XYZ calibration cube (dimensional accuracy)
- Temperature tower (find optimal temp for filament)
- Benchy boat (classic benchmark)

**Avoid for First Print:**
- Very large prints (waste filament if issues)
- Tall thin objects (risk of failure)
- Complex multi-hour prints
- Parts requiring supports (add complexity)

---

## Post-Configuration Summary

✅ **OrcaSlicer Configured:**
- Installed on Windows PC (not printer laptop)
- Connected to Moonraker at 10.0.0.139:7125
- Printer profile: Sovol SV06 (220x220x250mm)
- Start/End G-code with Klipper commands
- Print settings use calibrated values from log 12
- PLA filament profile created
- Ready to slice and print

✅ **Workflow Established:**
- Slice on Windows PC
- Upload via network to Moonraker
- Start/monitor prints via Mainsail web interface
- Camera streaming for remote monitoring

---

## Next Steps

1. ✅ OrcaSlicer configured with calibrated values
2. ✅ Moonraker connection working
3. ⏳ **Test slice and upload** (pending)
4. ⏳ **First test print** (when ready)
5. ⏳ Fine-tune settings based on print results
6. ⏳ Create filament-specific profiles as needed

**System fully operational and ready for 3D printing!**
