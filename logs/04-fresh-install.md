# Step 4: Fresh Klipper Installation using KIAUH

**Date:** 2025-11-16

## What is KIAUH?

**KIAUH** (Klipper Installation And Update Helper) is the recommended tool for installing and managing Klipper. It automates:
- Installing Klipper, Moonraker, and web interfaces
- Dependency management
- Updates and backups
- Multiple instance management

## Prerequisites Check

[DONE] Python 3.10.12 installed
[DONE] Git 2.34.1 installed
[DONE] Old installation completely removed
[DONE] System clean and ready

---

## Installation Steps

### 1. Download and Run KIAUH

```bash
cd ~
git clone https://github.com/dw-0/kiauh.git
cd kiauh
./kiauh.sh
```

This will open the KIAUH menu interface.

---

### 2. Install Klipper

In KIAUH menu:
1. Press `1` - [Install]
2. Press `1` - [Klipper]
3. Answer `1` - Python 3.x (recommended)
4. Answer `1` - Number of Klipper instances (just 1 for single printer)
5. Wait for installation to complete

**Expected output:** Klipper installed successfully

---

### 3. Install Moonraker

In KIAUH menu:
1. Press `1` - [Install]
2. Press `2` - [Moonraker]
3. Answer `1` - Number of Moonraker instances (1)
4. Wait for installation to complete

**Expected output:** Moonraker installed successfully

---

### 4. Install Mainsail (Web Interface)

In KIAUH menu:
1. Press `1` - [Install]
2. Press `3` - [Mainsail]
3. Answer `1` - Install for instance 1
4. Wait for installation to complete

**Expected output:** Mainsail installed successfully

---

### 5. Verify Installation

Exit KIAUH and run:

```bash
# Check services are running
systemctl status klipper
systemctl status moonraker

# Check directories created
ls -la ~/klipper
ls -la ~/moonraker
ls -la ~/printer_data/config/

# Check web interface
# Open browser to: http://10.0.0.139
```

---

## Output

(Paste outputs from each installation step below)

### KIAUH Download:

### Klipper Installation:

### Moonraker Installation:

### Mainsail Installation:

### Service Status:

---

## Next Steps

After successful installation:
1. [DONE] Verify services are running
2. [PENDING] Build Klipper firmware for SV06
3. [PENDING] Flash firmware to printer
4. [PENDING] Configure printer.cfg
