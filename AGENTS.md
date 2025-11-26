# Agent Instructions

This repository contains documentation and configuration files for Klipperizing a Sovol SV06 3D printer running on a spare laptop with Debian 13.

## Project Type

This is a **documentation and configuration repository**, not a software project. No build, lint, or test commands apply.

## Repository Purpose

- Document the process of installing and configuring Klipper on Debian 13
- Store Klipper configuration files (`.cfg`) for the Sovol SV06
- Keep logs and troubleshooting notes from the setup process
- Track configuration changes and improvements over time

## Session Initialization

**At the start of EVERY session, you MUST:**

1. **Read all log files** in the `logs/` directory in sequential order (00-, 01-, 02-, etc.)
2. **Understand the current state** of the system by reviewing what has been completed
3. **Identify any pending tasks** from the most recent log file
4. **Review configuration files** in `configs/` to understand the current printer setup

This ensures you have full context about:
- What has been done previously
- Current system configuration
- Any issues encountered
- Next steps in the setup process

**Do this automatically without being prompted by the user.**

## File Conventions

- **Configuration Files:** Use `.cfg` extension for Klipper configs (e.g., `printer.cfg`, `macros.cfg`)
- **Documentation:** Use Markdown (`.md`) for logs, guides, and notes
- **Log Files:** Use numbered prefix convention: `00-`, `01-`, `02-`, etc. (e.g., `05-initial-setup.md`, `06-debian12-install-plan.md`)
- **Images/Screenshots:** Store in `images/` directory with descriptive filenames
- **Formatting:** Use 4 spaces for indentation in config files
- **Comments:** In `.cfg` files, use `#` for comments. Be descriptive about why settings were chosen.

## Image Management

### Image Storage and Organization

All screenshots and images must be stored in the `images/` directory and cataloged in `images/IMAGE_CATALOG.md`.

### Adding New Images

When adding a new screenshot or image:

1. **Use descriptive filenames:**
   - Pattern: `source_description_context.png`
   - Good: `mainsail_bed_mesh_visualization.png`
   - Bad: `screenshot1.png`, `IMG_20250126.png`

2. **Save to images directory:**
   ```bash
   # Move image to correct location
   mv screenshot.png images/descriptive_name.png
   ```

3. **Update IMAGE_CATALOG.md:**
   - Add entry to the Image Index table
   - Create detailed section with:
     - Date taken
     - Source (application/interface)
     - Related log file(s)
     - Description of what image shows
     - Problem context (if applicable)
     - Root cause (if applicable)
     - Resolution/fix applied (if applicable)
     - Current status ([DONE] RESOLVED, [PENDING], [REFERENCE])

4. **Reference in documentation:**
   ```markdown
   ![Description](../images/filename.png)
   ```

5. **Commit with descriptive message:**
   ```bash
   git add images/new_image.png images/IMAGE_CATALOG.md
   git commit -m "Add screenshot showing [specific issue/feature]"
   ```

### Image Naming Conventions

**Format:** `source_description_context.png`

**Examples:**
- `orcaslicer_start_gcode_configuration.png`
- `mainsail_gcode_preview_centering_issue.png`
- `klipper_console_mcu_connection_error.png`
- `moonraker_api_warning_path_mismatch.png`

### Image Catalog Template

When adding to `images/IMAGE_CATALOG.md`, use this template:

```markdown
### N. filename.png

**Date Taken:** YYYY-MM-DD  
**Source:** Application/Interface name  
**Related Log:** [log-file.md](../logs/log-file.md)

**Description:**
Brief description of what the image shows.

**What It Shows:**
- Bullet points of specific elements visible
- Key information displayed
- Notable features or settings

**Problem Context:** (if applicable)
Description of the problem this image helped diagnose or document.

**Root Cause:** (if applicable)
Technical explanation of what caused the issue.

**Resolution:** (if applicable)
How the problem was fixed.

**Status:** [DONE] RESOLVED / [PENDING] / [REFERENCE]

**Technical Details:**
- Issue Type: Configuration/Hardware/Software
- Fix Type: Description of fix
- Files Modified: List of changed files
```

### Image Quality Guidelines

- **Format:** PNG for screenshots (lossless)
- **Resolution:** Keep readable text (minimum 1280x720)
- **File Size:** Keep under 5MB per image
- **Content:** Crop to relevant area, remove sensitive information

## Agent Behavior

- **Be Assertive:** Make decisions proactively without asking for permission on formatting, naming conventions, and standard practices
- **Git Commits:** Commit changes regularly after completing tasks or making significant changes
- **Git Push:** Push commits to remote repository frequently to ensure work is backed up (note: push may fail due to auth, user will handle manually)
- **Follow Conventions:** Always follow the established file naming and formatting conventions in this repository
- **Update Logs:** Update log files after completing significant steps, commit immediately
- **Document Everything:** Keep detailed records of all configuration changes, commands run, and decisions made
- **Run Verification Commands:** When guiding the user through a process, run verification commands via SSH automatically to check results. Do not ask the user to run simple checks - be proactive and verify things yourself. Only ask the user to perform actions that require physical interaction (like inserting SD cards, pressing buttons, etc.) or viewing output directly on their terminal.

## [WARN] CRITICAL: User-Guided Installation Protocol

**IMPORTANT: When performing Klipper ecosystem installations or modifications:**

### What is "Klipper Installation"?

**"Klipper installation" means the ENTIRE Klipper ecosystem**, including:
- Klipper firmware and service
- Moonraker (API server)
- Mainsail/Fluidd (web interfaces)
- nginx (web server for Klipper UIs)
- Printer configuration files
- Firmware flashing and updates
- Printer calibration procedures
- Any Klipper-related plugins or extensions

### Installation Protocol

1. **GUIDE, DON'T EXECUTE:** Your role is to GUIDE the user through the installation process, NOT to execute installation commands automatically
2. **WAIT FOR CONFIRMATION:** After providing instructions, WAIT for the user to explicitly ask you to proceed before running ANY installation commands
3. **ASK FIRST:** Before running any Klipper-related commands (`apt install`, `git clone`, configuration changes, systemctl operations), TELL the user what you plan to do and WAIT for their permission
4. **User is in Control:** The user decides when each step happens. Never assume they want you to execute the next step automatically.

**Example of CORRECT behavior:**
```
Agent: "The next step is to install nginx for the Mainsail web interface. Here's the command:
        sudo apt install -y nginx
        
        Would you like me to run this command now?"
User: "yes"
Agent: [Runs the command]
```

**Example of INCORRECT behavior:**
```
Agent: [Automatically runs: sudo apt install -y nginx]
```

**Exception:** Read-only verification commands (like `systemctl status`, `ls`, `cat`, checking logs, etc.) can be run automatically to check system state.

## Key Information

- **Printer Model:** Sovol SV06
- **Host System:** HP Pavilion 15-cc1xx laptop with Debian 13 (Trixie)
- **Hardware:** Intel i5-8250U, 8GB RAM, 232GB HDD (single drive configuration)
- **Network:** IP Address 10.0.0.139
- **SSH Access:** `ssh pri@10.0.0.139` (passwordless with SSH keys)
- **Firmware:** Klipper (installation in progress)

## Current Status

**Last Updated:** 2025-11-25

### Completed:
- [DONE] Debian 13 installed with HDD-only configuration (no EFI boot issues)
- [DONE] Clamshell mode enabled (headless operation)
- [DONE] SSH configured with key authentication (passwordless)
- [DONE] System updated and build dependencies installed
- [DONE] Static IP configured: 10.0.0.139
- [DONE] Klipper installed and running
  - Python environment: `~/klippy-env`
  - Repository: `~/klipper`
  - Service: Active, connected to MCU
  - MCU: STM32F103 @ 72MHz via CH340 USB
  - Serial: `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`
- [DONE] Klipper firmware flashed (or working with existing firmware)
- [DONE] Moonraker v0.9.3-128-g960e933 installed via KIAUH
  - Repository: `~/moonraker`
  - Python environment: `~/moonraker-env`
  - Service: Active, connected to Klipper
  - Port: 7125
- [DONE] Mainsail web interface installed via KIAUH
  - Location: `~/mainsail`
  - Access: http://10.0.0.139
  - Status: Fully functional
- [DONE] Mainsail-Config installed
  - Location: `~/mainsail-config`
  - Config: `~/printer_data/config/mainsail.cfg` (symlink)
- [DONE] nginx web server installed and configured
  - Serving Mainsail on port 80
  - Proxying Moonraker API
- [DONE] Configuration issues fixed
  - Fixed `/home/pi/` to `/home/pri/` paths in configs
  - Moonraker socket path corrected
  - Virtual SD card path corrected
- [DONE] Printer controls verified (homing successful)
- [DONE] Camera streaming configured with Crowsnest
  - XWF-1080P USB camera on `/dev/video0`
  - Accessible at http://10.0.0.139/webcam/
  - Visible in Mainsail interface
- [DONE] Printer calibration completed
  - PID tuning: Bed (Kp=69.167, Ki=1.210, Kd=988.229)
  - PID tuning: Hotend (Kp=23.358, Ki=1.455, Kd=93.723)
  - Bed mesh: 5x5 grid, range -0.565mm to +0.401mm
  - Z-offset: 1.755mm (paper test method)
  - All values saved to printer.cfg
- [DONE] OrcaSlicer configured (Windows PC)
  - Connected to Moonraker at 10.0.0.139:7125
  - Sovol SV06 profile with calibrated values
  - Start/End G-code with Klipper commands
  - PLA filament profile (200°C/60°C)
  - Ready to slice and upload prints

### Configuration Files:
- **Main Config:** `~/printer_data/config/printer.cfg` (bassamanator SV06 config)
- **Moonraker:** `~/printer_data/config/moonraker.conf`
- **Mainsail:** `~/printer_data/config/mainsail.cfg` (symlink)
- **Crowsnest:** `~/printer_data/config/crowsnest.conf`
- **Additional:** `~/printer_data/config/cfgs/` (modular configs)

### System Ready for Printing:
- [DONE] All services running and enabled for auto-start
- [DONE] Web interface accessible with camera feed
- [DONE] Printer fully calibrated
- [DONE] Slicer configured and connected

### Next Steps (Optional Advanced Calibration):
- [PENDING] **First test print** to verify all systems
- [PENDING] **Pressure Advance tuning** (fine-tune extrusion)
- [PENDING] **Input Shaper calibration** (requires ADXL345 accelerometer)
- [PENDING] **Flow rate calibration** (material-specific)
