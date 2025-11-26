# USB Communication Failure - Initial Incident & Autosuspend Fix

**Date:** 2025-11-25 22:09-22:28  
**Status:** ✅ RESOLVED (Temporarily - recurred later)  
**Issue:** MCU communication lost during G28 (home all) command

> **Note:** This is Part 1 of 4. See [14-usb-troubleshooting-index.md](14-usb-troubleshooting-index.md) for the complete troubleshooting journey.

---

## Incident Timeline

**22:09** - User heating bed (60°C) and hotend (200°C)  
**22:09** - Multiple "Extrude below minimum temp" errors (expected, heaters not ready yet)  
**22:11** - User executes G28 (home all)  
**22:11** - **Klipper enters shutdown state: "Lost communication with MCU 'mcu'"**

---

## Error Analysis

### Primary Error from klippy.log

```
Lost communication with MCU 'mcu'
Once the underlying issue is corrected, use the
"FIRMWARE_RESTART" command to reset the firmware, reload the
config, and restart the host software.
Printer is shutdown
```

### Technical Details

**Communication Statistics at Time of Failure:**
```
Stats 3840.4: 
  bytes_write=120764 
  bytes_read=425962 
  bytes_retransmit=1503 (increasing)
  bytes_invalid=8 
  send_seq=9188 
  receive_seq=9181 
  freq=72004548
  
Last successful receive: analog_in_state oid=15 next_clock=3362158464
Then: Lost communication with MCU 'mcu'
```

**Context:**
- Printer was in the middle of X-axis homing sequence
- Bed heating to 60°C (current: 47.7°C)
- Hotend at target 200°C (current: 201.7°C)
- `bytes_retransmit` counter was steadily increasing before disconnect
- Communication timeout occurred during stepper motor movement

### MCU State Before Disconnect

```
Receive: 76 3840.173089: trsync_state oid=8 can_trigger=1 trigger_reason=0
Receive: 77 3840.179077: analog_in_state oid=15
Receive: 78 3840.234437: trsync_state oid=8 can_trigger=0 trigger_reason=1
...
Lost communication with MCU 'mcu'
```

The printer was actively homing (trsync_state messages) when communication failed.

---

## Root Cause Identification

### USB Serial Device: CH340 Chip

**Device Info:**
- Bus 001 Device 003: ID 1a86:7523 QinHeng Electronics CH340 serial converter
- Serial: `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0` → `/dev/ttyUSB0`
- Driver: `ch341` kernel module

### Issue: USB I/O Error After Communication Loss

After the disconnect, attempting to restart Klipper resulted in:

```
mcu 'mcu': Unable to open serial port: [Errno 5] could not open port 
/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0: [Errno 5] 
Input/output error: '/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0'
```

After driver reset (`rmmod ch341 && modprobe ch341`):
```
mcu 'mcu': Unable to open serial port: [Errno 2] could not open port 
/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0: [Errno 2] 
No such file or directory
```

**Observations:**
- USB device detected by `lsusb` (Bus 001 Device 003)
- `ch341` driver loaded but not binding to device
- No `/dev/ttyUSB*` device created
- No `/dev/serial/by-id/` symlinks present
- **Physical USB cable reconnection required**

---

## Probable Causes

### 1. **Electrical Noise from Stepper Motors** (Most Likely)

**Evidence:**
- Failure occurred during active homing (motor movement)
- `bytes_retransmit` counter increasing before disconnect
- CH340 chips are particularly sensitive to electrical interference

**Mechanism:**
- Stepper motor currents generate electromagnetic interference (EMI)
- EMI couples into USB data lines
- CH340 USB-to-serial chip loses sync with host
- Communication fails, USB device enters error state

### 2. **USB Power Issues**

**Potential Contributors:**
- Laptop USB ports may have power-saving features enabled
- Current draw from printer + laptop operations
- USB autosuspend potentially enabled on device

### 3. **CH340 Driver Limitations**

**Known Issues:**
- CH340/CH341 chips have poor noise immunity compared to FTDI chips
- Driver can lose device binding after I/O errors
- Recovery requires physical device reset

### 4. **Cable Quality/Routing**

**Considerations:**
- USB cable may lack proper shielding
- Cable routed near stepper motor wiring
- No ferrite beads for noise suppression

---

## Immediate Recovery Steps

### Attempted Recovery Actions

1. ✅ **FIRMWARE_RESTART button in Mainsail** - No response (Klipper stuck in error state)
2. ✅ **systemctl restart klipper.service** - Failed (USB device I/O error)
3. ✅ **USB driver reset** (`rmmod ch341 && modprobe ch341`) - Device not binding
4. ✅ **udevadm trigger** - No effect
5. ✅ **Physical USB cable reconnection** - SUCCESS

### Required Action

**Physical USB cable disconnection/reconnection:**
1. Unplug USB cable from laptop
2. Wait 5 seconds (allow USB bus to fully reset)
3. Reconnect USB cable
4. Verify device appears: `ls -la /dev/serial/by-id/`
5. Restart Klipper: `sudo systemctl restart klipper.service`

---

## Solution Applied: Disable USB Autosuspend

### Implementation

**Created udev rule:**
```bash
sudo nano /etc/udev/rules.d/50-usb-autosuspend-ch340.rules
```

**Content:**
```bash
# Disable autosuspend for CH340 USB serial adapter (3D printer)
ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="1a86", ATTR{idProduct}=="7523", TEST=="power/autosuspend", ATTR{power/autosuspend}="-1"
```

**Activation:**
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

---

## Testing and Verification

### Status Updates

**2025-11-25 22:15** - Initial incident, Klipper shutdown  
**2025-11-25 22:16** - FIRMWARE_RESTART button unresponsive  
**2025-11-25 22:17** - systemctl restart attempted, USB I/O error discovered  
**2025-11-25 22:18** - Driver reset attempted, device not binding  
**2025-11-25 22:20** - Documentation created, awaiting physical USB reconnection  
**2025-11-25 22:22** - USB cable reconnected, device detected  
**2025-11-25 22:23** - Klipper restarted successfully, MCU communication restored  
**2025-11-25 22:24** - USB autosuspend disabled via udev rule  
**2025-11-25 22:25** - Testing: Single G28 homing - SUCCESSFUL  
**2025-11-25 22:27** - Testing: 3 consecutive G28 cycles - ALL SUCCESSFUL  
**2025-11-25 22:28** - Verification: `bytes_retransmit` counter stable (no new retransmits)

### Test Results

**Test 1: Single G28 Homing**
- ✅ Result: SUCCESS
- No disconnection errors

**Test 2: Stress Test - 3 Consecutive G28 Cycles**
- ✅ Result: ALL SUCCESSFUL
- `send_seq=2453`, `receive_seq=2453` (perfect packet sync)
- Printer state: "ready" (operational)

### Communication Statistics After Fix

```
Stats 4361.6: 
  bytes_write=43668 
  bytes_read=46680 
  bytes_retransmit=9 (unchanged from initial connect)
  bytes_invalid=0 
  send_seq=2453 receive_seq=2453
  freq=72004509
  
USB autosuspend: -1 (disabled)
```

---

## Resolution Summary

### Actions Taken

1. ✅ **Physical USB reconnection** - Restored device binding
2. ✅ **Klipper service restart** - MCU communication restored
3. ✅ **Permanent USB autosuspend disable** - Created udev rule
4. ✅ **Stability testing** - 4 homing cycles completed without errors

### Files Modified

**Created:**
- `/etc/udev/rules.d/50-usb-autosuspend-ch340.rules` - Disables USB autosuspend for CH340 device

### Current Status (After This Fix)

**USB Device:**
- Device: Bus 001 Device 006: ID 1a86:7523 QinHeng Electronics CH340
- Serial: `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0` → `/dev/ttyUSB0`
- Autosuspend: Disabled (`-1`)

**Klipper:**
- Status: Active and running
- MCU: Connected and responsive
- Communication: Stable (no retransmit increase)

---

## Long-Term Solutions (Recommended but Not Yet Implemented)

### Solution 1: Improve USB Cable Quality (HIGH PRIORITY)

**Recommendation:**
- Replace with **high-quality shielded USB cable**
- Look for cables with **ferrite cores/beads** at both ends
- Prefer USB 2.0 certified cables (better noise immunity than cheap USB 3.0 cables)
- Keep cable length reasonable (< 2 meters)

**Cable Routing:**
- Route USB cable **away from stepper motor wires**
- Avoid parallel runs with motor cables
- Cross motor cables at 90-degree angles if necessary
- Use cable management to maintain separation

### Solution 2: Electrical Grounding (MEDIUM PRIORITY)

**Ensure Proper Grounding:**
- Verify printer frame is grounded
- Connect ground wire between printer frame and laptop chassis
- Use 3-prong grounded power outlets for both devices

### Solution 3: Upgrade to FTDI-Based USB Adapter (LONG-TERM)

**Hardware Upgrade:**
- Replace CH340-based USB cable with FTDI FT232RL-based adapter
- FTDI chips have better noise immunity and driver support
- More expensive but more reliable for 3D printing applications

---

## Prevention Checklist

- [x] Disable USB autosuspend via udev rule
- [ ] Use high-quality shielded USB cable with ferrite cores
- [ ] Route USB cable away from stepper motor wires
- [ ] Verify proper grounding of printer and laptop
- [ ] Consider FTDI-based USB adapter if issues persist
- [ ] Monitor `bytes_retransmit` counter in klippy.log during prints
- [ ] Test homing repeatedly to verify stability

---

## Monitoring and Testing Recommendations

### Test Procedure (For Future Verification)

1. **Cold Start Test:**
   ```gcode
   G28          # Home all axes
   G28 X Y Z    # Home individual axes
   G28          # Home again
   ```

2. **Heat + Movement Test:**
   ```gcode
   M140 S60     # Heat bed
   M104 S200    # Heat hotend
   G28          # Home while heating
   ```

3. **Repeated Homing Test:**
   ```gcode
   G28
   G28
   G28
   G28
   G28          # 5x homing in succession
   ```

4. **Monitor klippy.log:**
   ```bash
   tail -f ~/printer_data/logs/klippy.log | grep -i "retransmit\|lost\|disconnect"
   ```

### Success Criteria

- ✅ No "Lost communication" errors during 5 consecutive homing cycles
- ✅ `bytes_retransmit` counter stays at 0 or low values (< 100)
- ✅ No USB I/O errors in system logs
- ✅ Printer can complete full homing sequence while heating

---

## References

### Similar Issues in Community

- Klipper GitHub Issue #1234 (example): CH340 USB disconnect during homing
- Klipper Discourse: USB communication errors with CH340 adapters
- Reddit r/klippers: "Lost communication with MCU" troubleshooting guide

### Technical Documentation

- Klipper Documentation: https://www.klipper3d.org/FAQ.html#klipper-lost-communication-with-mcu
- CH341 Linux Driver: https://github.com/torvalds/linux/blob/master/drivers/usb/serial/ch341.c
- USB Power Management: https://www.kernel.org/doc/html/latest/driver-api/usb/power-management.html

---

**Status:** ✅ RESOLVED (Temporarily)

**Note:** This fix resolved the immediate issue, but problems recurred later. See:
- [14b-usb-port-change.md](14b-usb-port-change.md) - Second failure and USB port change
- [14c-pyserial-compatibility.md](14c-pyserial-compatibility.md) - Python 3.13 compatibility issue
- [14d-usb-power-delivery-fix.md](14d-usb-power-delivery-fix.md) - Final fix (USB cable modification)
