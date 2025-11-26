# Moonraker and Mainsail Installation

**Date:** 2025-11-25  
**Status:** Pending  
**Prerequisites:** ✅ Klipper firmware verified and communicating

---

## Current Status

✅ **Completed:**
- Debian 13 system installed and configured
- Static IP configured: 10.0.0.139
- Klipper firmware functional
- MCU communication verified
- Official SV06 configuration downloaded

⏳ **Pending:**
- Moonraker installation (API server)
- Mainsail installation (web interface)
- Initial printer calibration

---

## Installation Plan

### Step 1: Install Moonraker

Moonraker is the API server that provides web access to Klipper.

### Step 2: Install Mainsail

Mainsail is the web-based user interface for controlling the printer.

### Step 3: Configure Services

Ensure both services start automatically on boot.

### Step 4: Test Web Access

Access the printer via web browser at http://10.0.0.139

---

## Next Session Tasks

1. Install Moonraker using official installation script
2. Install Mainsail web interface
3. Configure nginx for web serving
4. Test web interface access
5. Begin printer calibration (following official guide)

