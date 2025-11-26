"# Sovol SV06 Klipper Configuration

A complete documentation repository for converting a **Sovol SV06** 3D printer to run **Klipper firmware** on a dedicated **Debian 13** laptop server.

## Project Overview

This repository documents the entire journey of "Klipperizing" a Sovol SV06, from initial planning through troubleshooting to a fully operational print server. It serves as both a reference guide and a detailed record of configuration decisions, challenges overcome, and lessons learned.

**Status:** ✅ **System Operational and Print-Ready**

## Hardware Configuration

### Printer
- **Model:** Sovol SV06
- **MCU:** STM32F103 @ 72MHz
- **USB Interface:** CH340 serial adapter
- **Serial Device:** `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`
- **USB Port:** Port 1 (⚠️ Port 2 is faulty - see [log 14](logs/14-usb-communication-failure.md))

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
└── 14-usb-communication-failure.md         # USB port troubleshooting
```

**Naming Convention:**
- `##-descriptive-name.md` for sequential logs
- `##x-supplemental-topic.md` for related subtopics (e.g., `04a`, `04b`)

## Configuration Files

Klipper configuration files are stored in `configs/` for version control:

```
configs/
└── backup-old-printer.cfg          # Original Sovol firmware backup
```

**Active configurations** are symlinked/managed in `~/printer_data/config/` on the print server.

## System Calibration Status

### Completed Calibrations ✅
- **PID Tuning (Bed):** Kp=69.167, Ki=1.210, Kd=988.229
- **PID Tuning (Hotend):** Kp=23.358, Ki=1.455, Kd=93.723
- **Bed Mesh Leveling:** 5x5 grid, range -0.565mm to +0.401mm
- **Z-Offset:** 1.755mm (paper test method)

### Optional Advanced Calibrations
- ⏳ Pressure Advance tuning (fine-tune extrusion)
- ⏳ Input Shaper calibration (requires ADXL345 accelerometer)
- ⏳ Flow rate calibration (material-specific)

## Known Issues and Solutions

### ⚠️ Critical: USB Port 2 is Faulty
**Issue:** USB Port 2 on the laptop causes intermittent device disconnections and I/O errors.

**Solution:** Use **USB Port 1** for the printer. Port 2 should be avoided for critical devices.

**Documentation:** See [logs/14-usb-communication-failure.md](logs/14-usb-communication-failure.md) for full troubleshooting details.

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

**Last Updated:** 2025-11-25  
**Maintained By:** Print server running at `pri@10.0.0.139`" 
