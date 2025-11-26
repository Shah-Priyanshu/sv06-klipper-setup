# Sovol SV06 Klipper Configuration

A complete documentation repository for converting a **Sovol SV06** 3D printer to run **Klipper firmware** on a dedicated **Debian 13** laptop server.

## Project Overview

This repository documents the entire journey of "Klipperizing" a Sovol SV06, from initial planning through troubleshooting to a fully operational print server. It serves as both a reference guide and a detailed record of configuration decisions, challenges overcome, and lessons learned.

**Status:** [DONE] **System Operational and Print-Ready** (Updated: 2025-11-26)

## Hardware Configuration

### Printer
- **Model:** Sovol SV06
- **MCU:** STM32F103 @ 72MHz
- **USB Interface:** CH340 serial adapter
- **Serial Device:** `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`
- **USB Port:** Port 1 ([WARN] Port 2 is faulty - see [log 14](logs/14-usb-troubleshooting-index.md))

### Print Server (Laptop)
- **Model:** HP Pavilion 15-cc1xx
- **CPU:** Intel Core i5-8250U (4 cores, 8 threads)
- **RAM:** 8GB
- **Storage:** 232GB HDD (single drive, no dual-boot)
- **OS:** Debian 13 (Trixie) - 64-bit
- **Network:** Static IP `10.0.0.139`
- **SSH:** Key-based authentication (passwordless)
- **Mode:** Clamshell/headless operation

### Peripherals
- **Camera:** XWF-1080P USB camera on USB-C hub
- **Stream:** http://10.0.0.139/webcam/

## Software Stack

| Component | Version | Purpose |
|-----------|---------|---------|
| **Klipper** | Latest (main) | Firmware and motion control |
| **Moonraker** | v0.9.3-128-g960e933 | API server for web interfaces |
| **Mainsail** | Latest | Web-based printer control UI |
| **Crowsnest** | Latest | Camera streaming service |
| **nginx** | Latest | Web server and reverse proxy |
| **OrcaSlicer** | Latest | Slicer software (Windows PC) |

## Quick Start

### Accessing the Printer

**Web Interface:**
```bash
http://10.0.0.139
```

**SSH Access:**
```bash
ssh pri@10.0.0.139
```

**Moonraker API:**
```bash
http://10.0.0.139:7125
```

### Key File Locations

```
~/klipper/                          # Klipper source code
~/klippy-env/                       # Klipper Python environment
~/moonraker/                        # Moonraker source code
~/moonraker-env/                    # Moonraker Python environment
~/mainsail/                         # Mainsail web UI files
~/printer_data/                     # Configuration and runtime data
  ├── config/
  │   ├── printer.cfg               # Main Klipper configuration
  │   ├── moonraker.conf            # Moonraker settings
  │   ├── mainsail.cfg              # Mainsail UI macros
  │   ├── crowsnest.conf            # Camera streaming config
  │   └── cfgs/                     # Modular config files
  ├── logs/                         # Runtime logs
  └── gcodes/                       # Uploaded G-code files
```

## Documentation Structure

This repository uses a **numbered log system** to document the setup process chronologically:

```
logs/
├── 00-github-setup.md                      # Initial repository setup
├── 01-system-info.md                       # Hardware specifications
├── 02-backup-check.md                      # Pre-installation backup verification
├── 03-complete-removal.md                  # Old OS removal (Windows/Ubuntu)
├── 04-fresh-install.md                     # Debian 12 installation attempt
├── 04a-troubleshooting-bus-error.md        # I/O bus errors during install
├── 04b-critical-disk-issues.md             # Drive failure detection and recovery
├── 05-initial-setup.md                     # Hardware decision and rationale
├── 06-debian12-install-plan.md             # Updated install plan
├── 07-post-install-setup.md                # SSH, networking, static IP
├── 08-klipper-installation.md              # Klipper firmware setup
├── 09-firmware-flash-troubleshooting.md    # MCU firmware issues
├── 10-moonraker-mainsail-installation.md   # Web interface setup
├── 11-camera-streaming-setup.md            # Crowsnest camera configuration
├── 12-printer-calibration.md               # PID tuning, bed mesh, Z-offset
├── 13-orcaslicer-setup.md                  # Slicer configuration (Windows)
├── 14-usb-troubleshooting-index.md         # USB communication issues index
│   ├── 14a-initial-failure-autosuspend-fix.md
│   ├── 14b-usb-port-change.md
│   ├── 14c-pyserial-compatibility.md
│   └── 14d-usb-power-delivery-fix.md       # Final resolution (cut +5V wire)
├── 15-gcode-path-fix.md                    # Virtual SD card path correction
├── 16-orcaslicer-bed-size-fix.md           # Bed size configuration (250mm→220mm)
└── 17-first-test-print-analysis.md         # Calibration cube quality analysis
```

**Naming Convention:**
- `##-descriptive-name.md` for sequential logs
- `##x-supplemental-topic.md` for related subtopics (e.g., `04a`, `04b`)

## Image Documentation

All screenshots and diagnostic images are organized in the `images/` directory with comprehensive documentation:

```
images/
├── IMAGE_CATALOG.md                        # Master catalog of all images
├── start_end_gcode.png                     # OrcaSlicer G-code macro configuration
├── slicer_test_print.png                   # OrcaSlicer preview showing bed size issue
├── printer_gcode_viewer_tracking.png       # Mainsail preview showing position discrepancy
├── cube_*.jpg                              # Calibration cube face analysis (6 images)
└── screw_*.jpg                             # Screw thread test analysis (4 images)
```

Each image in the catalog includes:
- Date taken and source application
- Related log file reference
- Problem context and root cause analysis
- Resolution and current status
- Technical details and impact

**See:** [images/IMAGE_CATALOG.md](images/IMAGE_CATALOG.md) for detailed documentation of each image.

## Configuration Files

Klipper configuration files are stored in `configs/` for version control:

```
configs/
└── backup-old-printer.cfg          # Original Sovol firmware backup
```

**Active configurations** are symlinked/managed in `~/printer_data/config/` on the print server.

## System Calibration Status

### Completed Calibrations [DONE]
- **PID Tuning (Bed):** Kp=69.167, Ki=1.210, Kd=988.229
- **PID Tuning (Hotend):** Kp=23.358, Ki=1.455, Kd=93.723
- **Bed Mesh Leveling:** 5x5 grid, range -0.565mm to +0.401mm
- **Z-Offset:** 1.755mm (paper test method)

### Optional Advanced Calibrations
- [PENDING] Pressure Advance tuning (fine-tune extrusion)
- [PENDING] Input Shaper calibration (requires ADXL345 accelerometer)
- [PENDING] Flow rate calibration (material-specific)

## Recent Fixes (2025-11-26)

### G-Code Upload Path Fix [DONE]
**Issue:** OrcaSlicer uploads failed with "Unable to open file" error. Moonraker reported G-code path mismatch.

**Root Cause:** `cfgs/misc-macros.cfg` had incorrect path `/home/pi/printer_data/gcodes` instead of `/home/pri/printer_data/gcodes`.

**Solution:** Updated virtual_sdcard path in misc-macros.cfg and restarted Klipper.

**Documentation:** [logs/15-gcode-path-fix.md](logs/15-gcode-path-fix.md)

### OrcaSlicer Bed Size Fix [DONE]
**Issue:** Objects appeared centered in OrcaSlicer but off-center in Mainsail G-code viewer.

**Root Cause:** OrcaSlicer bed configured as 250mm×250mm instead of actual 220mm×220mm (Sovol SV06 spec).

**Solution:** Corrected OrcaSlicer printer profile bed size to 220mm×220mm.

**Documentation:** [logs/16-orcaslicer-bed-size-fix.md](logs/16-orcaslicer-bed-size-fix.md)

### PRINT_START Macro Configuration [DONE]
**Issue:** Print start failed with Jinja2 template error - macro expecting `BED` and `HOTEND` parameters.

**Root Cause:** OrcaSlicer start G-code used standard commands instead of calling PRINT_START macro.

**Solution:** Changed OrcaSlicer start G-code to: `PRINT_START BED=[bed_temperature_initial_layer_single] HOTEND=[nozzle_temperature_initial_layer]`

**Documentation:** [logs/15-gcode-path-fix.md](logs/15-gcode-path-fix.md)

## Known Issues and Solutions

### [WARN] Critical: USB Power Delivery Issue - RESOLVED
**Issue:** MCU communication failures during combined heating and homing operations.

**Root Cause:** USB 5V power delivery causing ground loops and signal degradation under high electrical load.

**Solution:** Modified USB cable by cutting the +5V wire (red wire), making it data-only. Printer MCU now powered solely by its onboard regulator.

**Documentation:** [logs/14d-usb-power-delivery-fix.md](logs/14d-usb-power-delivery-fix.md)

### USB Autosuspend Disabled
USB power management has been disabled for the CH340 serial adapter to prevent communication interruptions.

**Rule:** `/etc/udev/rules.d/50-usb-autosuspend-ch340.rules`

## Maintenance Commands

### Service Management
```bash
# Check Klipper status
sudo systemctl status klipper

# Check Moonraker status
sudo systemctl status moonraker

# Restart Klipper
sudo systemctl restart klipper

# View Klipper logs
journalctl -u klipper -f
```

### Firmware Updates
```bash
# Update Klipper
cd ~/klipper
git pull
sudo systemctl restart klipper

# Update Moonraker
cd ~/moonraker
git pull
sudo systemctl restart moonraker
```

### Configuration Backup
Configuration files are automatically backed up by Moonraker to:
```
~/printer_data/config/.git/        # Local git repository
```

## Contributing

This is a personal documentation repository, but feel free to use it as a reference for your own Klipper setup. Issues and pull requests are welcome for corrections or improvements to documentation.

## Resources

### Official Documentation
- [Klipper Documentation](https://www.klipper3d.org/)
- [Moonraker Documentation](https://moonraker.readthedocs.io/)
- [Mainsail Documentation](https://docs.mainsail.xyz/)

### Sovol SV06 Specific
- [bassamanator/Sovol-SV06-firmware](https://github.com/bassamanator/Sovol-SV06-firmware) - Configuration used in this build
- [Sovol SV06 Community Resources](https://github.com/Sovol3d/SV06-Fully-Open-Source)

### Helpful Guides
- [Klipper Configuration Reference](https://www.klipper3d.org/Config_Reference.html)
- [Klipper G-Codes](https://www.klipper3d.org/G-Codes.html)
- [Ellis' Print Tuning Guide](https://ellis3dp.com/Print-Tuning-Guide/)

## License

This documentation is provided as-is for educational and reference purposes. Configuration files are based on the bassamanator SV06 firmware (GPL-3.0).

---

**Last Updated:** 2025-11-26  
**Maintained By:** Print server running at `pri@10.0.0.139`
