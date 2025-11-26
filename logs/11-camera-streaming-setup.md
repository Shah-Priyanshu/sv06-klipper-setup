# Camera Streaming Setup with Crowsnest

**Date:** 2025-11-25  
**Status:** [DONE] COMPLETED  
**Prerequisites:** [DONE] Klipper, Moonraker, and Mainsail installed and functional  
**Installation Method:** KIAUH (Klipper Installation And Update Helper)

---

## Installation Summary

[DONE] **Successfully Installed:**
- **Crowsnest** - Multi-camera streaming daemon for Klipper
- **ustreamer** - MJPEG streaming engine
- Webcam configuration in Mainsail

[DONE] **Camera Detected:**
- **Model:** Sunplus SPCA2281 Web Camera (USB)
- **Device:** `/dev/video0` (XWF-1080P)
- **Resolution:** 640x480 @ 25fps
- **Stream Port:** 8080

[DONE] **Services Status:**
- Crowsnest: Active and streaming
- Klipper: Active and enabled for boot
- Moonraker: Active and connected
- nginx: Active

[DONE] **Web Interface:**
- Camera feed visible in Mainsail
- Stream URL: http://10.0.0.139/webcam/?action=stream
- Snapshot URL: http://10.0.0.139/webcam/?action=snapshot

---

## Hardware Setup

### Connected Cameras

**System detected two cameras:**

1. **XWF-1080P (USB External Camera)** ← Selected for printing
   - Device: `/dev/video0`, `/dev/video1`
   - Manufacturer: Sunplus Innovation Technology Inc.
   - USB ID: `1bcf:2281`
   - Media: `/dev/media0`

2. **HP Wide Vision HD Camera (Built-in Laptop Camera)**
   - Device: `/dev/video2`, `/dev/video3`
   - Manufacturer: Realtek Semiconductor Corp.
   - USB ID: `0bda:58eb`
   - Media: `/dev/media1`

**Selected Camera:** XWF-1080P (external USB camera) on `/dev/video0`

---

## Installation Process

### Step 1: Verify Camera Detection

Checked for USB cameras:
```bash
lsusb | grep -i 'camera\|webcam\|video'
ls -la /dev/video*
```

**Output:**
```
Bus 001 Device 002: ID 0bda:58eb Realtek Semiconductor Corp. HP Wide Vision HD Camera
Bus 001 Device 041: ID 1bcf:2281 Sunplus Innovation Technology Inc. SPCA2281 Web Camera

crw-rw----+ 1 root video 81, 0 Nov 17 18:32 /dev/video0  (USB camera)
crw-rw----+ 1 root video 81, 1 Nov 17 18:32 /dev/video1  (USB camera metadata)
crw-rw----+ 1 root video 81, 2 Nov 25 20:05 /dev/video2  (Built-in camera)
crw-rw----+ 1 root video 81, 3 Nov 25 20:05 /dev/video3  (Built-in camera metadata)
```

### Step 2: Install Crowsnest via KIAUH

**KIAUH Navigation:**
1. Launched KIAUH: `cd ~/kiauh && ./kiauh.sh`
2. Selected: `1) [Install]`
3. Selected: `8) [Crowsnest]`

**Installation Actions:**
- Repository already existed at `~/crowsnest` (skipped clone)
- Ran Crowsnest installer script
- Prompted for sudo password
- Ran `apt-get update`
- Checked for mjpg-streamer (found OK)
- Detected non-Raspberry Pi device (camera-streamer skipped)
- Installed dependencies (already present):
  - git, crudini, bsdutils, findutils, v4l-utils
  - curl, build-essential, libevent-dev, libjpeg-dev
  - libbsd-dev, pkg-config
- Created file structure in `~/printer_data/`
- Linked crowsnest to `/usr/local/bin/`
- Installed systemd service file
- Installed environment file
- Installed logrotate configuration
- Backed up existing config: `crowsnest.conf.2025-11-25-2019`
- Installed new `crowsnest.conf`
- Enabled crowsnest.service
- User `pri` already in `video` group (skipped)
- Cloned ustreamer repository
- Built ustreamer using 4 CPU cores (successful)
- Added update_manager entry to moonraker.conf (already existed, skipped)

**Installation Output:**
```
Installation successful.
Reboot your machine for the changes to take effect!
```

**Note:** Declined reboot, chose to restart services instead.

### Step 3: Verify Crowsnest Service

Checked service status:
```bash
systemctl status crowsnest
```

**Result:** [DONE] Active (running) since 20:16:24, streaming from `/dev/video0`

**Stream Details:**
- Mode: ustreamer
- Port: 8080
- Device: `/dev/video0`
- Resolution: 640x480
- FPS: 15
- Format: MJPEG
- Encoder: HW (hardware)
- Host: 127.0.0.1 (localhost)

### Step 4: Verify Camera Device Mapping

Used `v4l2-ctl` to identify cameras:
```bash
v4l2-ctl --list-devices
```

**Output:**
```
XWF-1080P: XWF-1080P (usb-0000:00:14.0-1):
    /dev/video0  ← USB camera (selected)
    /dev/video1
    /dev/media0

HP Wide Vision HD Camera: HP Wi (usb-0000:00:14.0-3):
    /dev/video2  ← Built-in camera
    /dev/video3
    /dev/media1
```

**Confirmed:** Crowsnest correctly using USB camera on `/dev/video0`

### Step 5: Test Camera Stream

Tested stream endpoint:
```bash
curl -I http://localhost:8080/?action=snapshot
```

**Result:**
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

[DONE] Camera stream accessible and responding

### Step 6: Configure Webcam in Mainsail

**User performed via web interface:**
1. Opened Mainsail: http://10.0.0.139
2. Navigated to Settings (gear icon)
3. Added new webcam with settings:
   - **Name:** Printer Camera
   - **URL Stream:** `/webcam/?action=stream`
   - **URL Snapshot:** `/webcam/?action=snapshot`
   - **Service:** ustreamer
   - **Target FPS:** 15
4. Saved configuration
5. Camera feed appeared in Dashboard

### Step 7: Troubleshoot Klipper Service Issue

**Issue Discovered:** After adding camera, Mainsail showed "DISCONNECTED" error

**Root Cause:** Klipper service had stopped (status: inactive/dead)

**Diagnosis:**
```bash
systemctl is-active klipper  # Output: inactive
systemctl status klipper     # Output: inactive (dead)
```

**Fix Applied:**

1. Started Klipper service:
   ```bash
   sudo systemctl start klipper
   ```

2. Enabled Klipper for auto-start on boot:
   ```bash
   sudo systemctl enable klipper
   ```

3. Verified Moonraker connection:
   ```bash
   curl http://localhost:7125/server/info | grep klippy_state
   # Output: "klippy_state": "ready"
   ```

**Result:** [DONE] All services restored and functional

### Step 8: Final Verification

Verified all services active and enabled:
```bash
systemctl is-active klipper moonraker nginx crowsnest
# All: active

systemctl is-enabled klipper moonraker nginx crowsnest
# All: enabled
```

**Final Status:**
- [DONE] Klipper: Running, enabled, connected to MCU
- [DONE] Moonraker: Running, enabled, connected to Klipper
- [DONE] nginx: Running, enabled, serving Mainsail
- [DONE] Crowsnest: Running, enabled, streaming camera
- [DONE] Mainsail: Connected, camera feed visible
- [DONE] All services will auto-start on boot

---

## Configuration Files

### Crowsnest Configuration

**File:** `~/printer_data/config/crowsnest.conf`

**Key Settings:**
```ini
[crowsnest]
log_path: /home/pri/printer_data/logs/crowsnest.log
log_level: verbose
delete_log: false
no_proxy: false

[cam 1]
mode: ustreamer
port: 8080
device: /dev/video0
resolution: 640x480
max_fps: 25
```

**Backup Created:** `crowsnest.conf.2025-11-25-2019`

**Update (2025-11-26):** Increased `max_fps` from 15 to 25 to improve stream smoothness. Camera supports up to 30 FPS with MJPG format.

### Moonraker Configuration

**File:** `~/printer_data/config/moonraker.conf`

**Update Manager Entry Added by KIAUH:**
```ini
[update_manager crowsnest]
type: git_repo
path: ~/crowsnest
origin: https://github.com/mainsail-crew/crowsnest.git
managed_services: crowsnest
install_script: tools/pkglist.sh
```

### Mainsail Webcam Configuration

**Configured via Mainsail Web Interface**

**Settings:**
- Name: Printer Camera
- Stream URL: `/webcam/?action=stream`
- Snapshot URL: `/webcam/?action=snapshot`
- Service: ustreamer
- Target FPS: 25
- Flip settings: As needed per user preference

**Configuration stored in:** `~/printer_data/config/.mainsail.json` (managed by Mainsail)

---

## Installed Components Details

### Crowsnest
- **Repository:** https://github.com/mainsail-crew/crowsnest.git
- **Location:** `/home/pri/crowsnest`
- **Executable:** `/usr/local/bin/crowsnest`
- **Service:** `/etc/systemd/system/crowsnest.service`
- **Environment:** `/home/pri/printer_data/systemd/crowsnest.env`
- **Logs:** `/home/pri/printer_data/logs/crowsnest.log`
- **Status:** Active, enabled

### ustreamer
- **Repository:** https://github.com/pikvm/ustreamer.git (commit: 2717248)
- **Location:** `/home/pri/crowsnest/bin/ustreamer`
- **Binary:** `/home/pri/crowsnest/bin/ustreamer/src/ustreamer.bin`
- **Build:** Compiled with 4 cores, successful
- **Features:** MJPEG streaming, hardware encoding support

### nginx Proxy Configuration
- **Proxy Path:** `/webcam/` → `http://127.0.0.1:8080/`
- **Configured by:** KIAUH during Mainsail installation
- **Access:** http://10.0.0.139/webcam/?action=stream

---

## Camera Stream URLs

### Internal (on laptop):
- **Stream:** `http://localhost:8080/?action=stream`
- **Snapshot:** `http://localhost:8080/?action=snapshot`

### External (from any device on network):
- **Stream:** `http://10.0.0.139/webcam/?action=stream`
- **Snapshot:** `http://10.0.0.139/webcam/?action=snapshot`

### Direct Port Access (if nginx proxy fails):
- **Stream:** `http://10.0.0.139:8080/?action=stream`
- **Snapshot:** `http://10.0.0.139:8080/?action=snapshot`

---

## Troubleshooting Notes

### Issue 1: Klipper Service Stopped

**Symptom:** Mainsail shows "Moonraker can't connect to Klipper" after camera setup

**Cause:** Klipper service became inactive during camera installation/configuration

**Solution:**
```bash
# Start service
sudo systemctl start klipper

# Enable auto-start
sudo systemctl enable klipper

# Verify
systemctl is-active klipper
curl http://localhost:7125/server/info | grep klippy_state
```

### Issue 2: Wrong Camera Device

**Symptom:** Wrong camera streaming (built-in instead of USB)

**Solution:** Check device mapping with `v4l2-ctl --list-devices` and update `device:` in `crowsnest.conf`

### Issue 3: Camera Not Detected

**Solution:**
```bash
# Check USB devices
lsusb | grep -i camera

# Check video devices
ls -la /dev/video*

# Check device details
v4l2-ctl --list-devices

# Check user permissions
groups | grep video  # User should be in video group
```

### Issue 4: Stream Not Accessible

**Solution:**
```bash
# Check Crowsnest status
systemctl status crowsnest

# Check Crowsnest logs
tail -50 ~/printer_data/logs/crowsnest.log

# Test stream locally
curl -I http://localhost:8080/?action=snapshot

# Check port availability
ss -tlnp | grep 8080
```

---

## Verification Commands

```bash
# Check all services
systemctl status klipper moonraker nginx crowsnest

# Check camera devices
v4l2-ctl --list-devices

# Test camera stream
curl -I http://localhost:8080/?action=snapshot

# View Crowsnest logs
tail -f ~/printer_data/logs/crowsnest.log

# View Klipper logs
tail -f ~/printer_data/logs/klippy.log
```

---

## Post-Installation Checklist

- [x] USB camera detected by system
- [x] Crowsnest installed via KIAUH
- [x] ustreamer built successfully
- [x] Crowsnest service running
- [x] Camera streaming on port 8080
- [x] Webcam configured in Mainsail
- [x] Camera feed visible in Mainsail dashboard
- [x] Klipper service started and enabled
- [x] Moonraker connected to Klipper
- [x] All services enabled for auto-start
- [x] Stream accessible via network

---

## Next Steps

1. [DONE] Camera streaming functional
2. [DONE] All Klipper ecosystem services running
3. [DONE] Web interface fully operational with camera
4. [PENDING] **Ready for printer calibration** (PID tuning, bed mesh, etc.)

**Camera setup complete!**
