# USB Communication Troubleshooting - Complete Journey

**Date:** 2025-11-25  
**Duration:** 22:09 - 23:42 (1 hour 33 minutes)  
**Final Status:** ✅ FULLY RESOLVED  
**System:** Sovol SV06 on Debian 13 (Trixie) with Klipper

---

## Overview

This document indexes the complete troubleshooting journey for USB communication failures between a Sovol SV06 3D printer (CH340 USB serial) and a laptop running Debian 13. The issue manifested as "Lost communication with MCU" errors during homing operations with heaters active.

**Three separate root causes were discovered and resolved:**
1. USB autosuspend causing device disconnects
2. Python 3.13 / pyserial 3.4 compatibility issue
3. USB power delivery problems under high electrical load **(PRIMARY ISSUE)**

---

## Quick Navigation

### Part 1: Initial Failure & USB Autosuspend Fix
**File:** [14a-initial-failure-autosuspend-fix.md](14a-initial-failure-autosuspend-fix.md)  
**Time:** 22:09 - 22:28  
**Issue:** First MCU communication failure during homing  
**Solution:** Disabled USB autosuspend via udev rule  
**Result:** ✅ Temporarily resolved (recurred later)

### Part 2: USB Port Change
**File:** [14b-usb-port-change.md](14b-usb-port-change.md)  
**Time:** 22:30 - 22:40  
**Issue:** Second failure, suspected faulty USB Port 2  
**Solution:** Moved printer from Port 2 to Port 1  
**Result:** ✅ Temporarily resolved (root cause was different)

### Part 3: Python 3.13 / pyserial Compatibility
**File:** [14c-pyserial-compatibility.md](14c-pyserial-compatibility.md)  
**Time:** 22:57  
**Issue:** AttributeError: 'Serial' object has no attribute 'pipe_abort_read_w'  
**Solution:** Upgraded pyserial from 3.4 to 3.5  
**Result:** ✅ Resolved software compatibility issue  
**Duration:** ~5 minutes

### Part 4: USB Power Delivery Fix (FINAL RESOLUTION)
**File:** [14d-usb-power-delivery-fix.md](14d-usb-power-delivery-fix.md)  
**Time:** 23:30 - 23:42  
**Issue:** MCU disconnects during combined heating + homing (high electrical load)  
**Solution:** Cut +5V wire in USB cable (data-only connection)  
**Result:** ✅ FULLY RESOLVED - Production ready  
**Testing:** 7 comprehensive tests, all passed

---

## Summary of Issues

### Issue 1: USB Autosuspend (Part 1)

**Symptoms:**
- MCU communication lost during homing
- USB device I/O errors
- Required physical cable reconnection

**Root Cause:**
- USB power management putting device to sleep

**Fix:**
```bash
# Created: /etc/udev/rules.d/50-usb-autosuspend-ch340.rules
ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="1a86", ATTR{idProduct}=="7523", TEST=="power/autosuspend", ATTR{power/autosuspend}="-1"
```

**Outcome:** Resolved autosuspend, but deeper issues remained

---

### Issue 2: Faulty USB Port? (Part 2)

**Symptoms:**
- Second failure after autosuspend fix
- Port 2 had history of camera instability

**Suspected Cause:**
- Hardware failure in USB Port 2

**Action Taken:**
- Moved printer from Port 2 to Port 1
- Moved camera to USB hub

**Actual Outcome:**
- Port 2 may not have been faulty
- Real issue was USB power delivery (discovered later)
- Port change temporarily worked due to cable reconnection

---

### Issue 3: Python 3.13 Compatibility (Part 3)

**Symptoms:**
```
AttributeError: 'Serial' object has no attribute 'pipe_abort_read_w'
```

**Root Cause:**
- Debian 13 ships with Python 3.13.5
- pyserial 3.4 incompatible with Python 3.13

**Fix:**
```bash
~/klippy-env/bin/pip install --upgrade pyserial
# Upgraded from 3.4 to 3.5
sudo systemctl restart klipper
```

**Outcome:** Resolved software compatibility, but USB power issue remained

---

### Issue 4: USB Power Delivery (Part 4 - FINAL FIX)

**Symptoms:**
- Failures ONLY when heating bed + hotend + homing simultaneously
- Failed on BOTH Port 1 AND Port 2 (key diagnostic clue)
- `bytes_retransmit` counter climbing (813 → 1925+)

**Root Cause:**
- MCU drawing power from laptop USB 5V line
- High printer PSU loads (heaters + motors) created ground loops
- Voltage sag and ground potential differences degraded USB data signals
- Affected ALL USB ports (shared power distribution)

**Fix:**
- Cut RED (+5V) wire in USB cable
- Created data-only USB connection
- MCU now powered solely by printer's 24V → 5V regulator

**Testing:**
- 7 comprehensive tests conducted
- 13+ minutes continuous operation under full load
- All tests PASSED ✅
- `bytes_retransmit` stable at 9 (no new retransmits)

**Outcome:** FULLY RESOLVED - System production-ready

---

## Timeline Summary

| Time | Event | File |
|------|-------|------|
| 22:09 | Initial MCU disconnect during homing | Part 1 |
| 22:15-22:28 | USB autosuspend fix + testing | Part 1 |
| 22:30 | Second failure on Port 2 | Part 2 |
| 22:40 | Moved to Port 1, seemed stable | Part 2 |
| 22:57 | Python 3.13 / pyserial error discovered | Part 3 |
| 22:57 | Upgraded pyserial to 3.5 | Part 3 |
| 23:30 | Third failure: heating + homing on Port 1 | Part 4 |
| 23:30 | Moved to Port 2: same failure pattern | Part 4 |
| 23:30 | **ROOT CAUSE IDENTIFIED:** USB power delivery | Part 4 |
| 23:30 | Modified USB cable (cut +5V wire) | Part 4 |
| 23:30-23:42 | 7 comprehensive tests, all passed | Part 4 |
| 23:42 | System declared production-ready | Part 4 |

---

## Key Diagnostic Insights

### 1. Multiple Port Failure = Not Port Hardware Issue

If the same problem occurs on DIFFERENT USB ports under the SAME conditions, the issue is likely electrical/power-related, NOT port hardware failure.

**Our Case:**
- Port 2 failed during heating + homing
- Port 1 ALSO failed during heating + homing
- Both ports fine for cold operations
- **Conclusion:** Shared power distribution problem

### 2. Load-Dependent Failures

Failures that ONLY occur under specific electrical loads are diagnostic gold:

**Our Pattern:**
- ✅ Works: Cold homing
- ✅ Works: Bed heating alone
- ✅ Works: Hotend heating alone
- ❌ **FAILS:** Bed + Hotend + Homing (combined load)

This pointed directly to power delivery as the culprit.

### 3. bytes_retransmit Counter

Monitor `bytes_retransmit` in `klippy.log`:
- **Stable counter** (e.g., 9) = healthy communication
- **Climbing counter** (813 → 1925+) = signal integrity degrading

This metric was critical for diagnosing the USB power issue.

---

## Final System Configuration

### Hardware

**Modified USB Cable:**
```
Pin 1: RED (+5V)   → ✂️ CUT - DISCONNECTED
Pin 2: WHITE (D-)  → ✅ CONNECTED
Pin 3: GREEN (D+)  → ✅ CONNECTED
Pin 4: BLACK (GND) → ✅ CONNECTED
```

**Power Flow:**
```
Printer 24V PSU → 5V Regulator → MCU Board
                                   ↓
                              USB Port (data only)
                                   ↓
                              Laptop USB Port 1
```

### Software

```
OS: Debian 13 (Trixie)
Python: 3.13.5
pyserial: 3.5 (upgraded from 3.4)
Klipper: v0.13.0-401-g90b7f823
USB autosuspend: Disabled
```

### Communication

```
Device: CH340 USB serial (1a86:7523)
Port: /dev/ttyUSB0
Baud: 250000
bytes_retransmit: 9 (stable baseline)
Status: Perfect communication under all loads
```

---

## Lessons Learned

### 1. Systematic Troubleshooting

Each "fix" revealed a different layer of issues:
1. Autosuspend → Resolved power management
2. Port change → Temporary relief (wrong hypothesis)
3. pyserial → Resolved software compatibility
4. Cable modification → Resolved true root cause

### 2. Test Under Real Conditions

Don't declare victory until testing under actual operating conditions:
- Cold testing ≠ Hot testing
- Single operations ≠ Combined operations
- Short tests ≠ Sustained operation

### 3. Bleeding Edge OS Risks

Debian 13 (Trixie) + Python 3.13:
- Better hardware support
- BUT: Library compatibility issues
- Solution: Update dependencies immediately after install

### 4. USB Isn't Always USB

USB can serve multiple purposes:
- Power + Data (default)
- Data only (our solution)
- Isolated (with USB isolator)

Separating power from data eliminated electrical interference.

---

## Recommendations for Others

### If You Experience Similar Issues

1. **Check your environment:**
   - Debian 13 / Python 3.13? → Upgrade pyserial to 3.5
   - USB autosuspend enabled? → Disable it

2. **Test systematically:**
   - Cold homing vs. hot homing
   - Individual heaters vs. combined
   - Single operations vs. repeated cycles

3. **Monitor diagnostics:**
   - Watch `bytes_retransmit` in klippy.log
   - Check if failures are load-dependent
   - Test on multiple USB ports

4. **Consider USB cable modification:**
   - If failures occur on multiple ports under high load
   - If `bytes_retransmit` climbs before disconnect
   - If you have a laptop or SBC (limited USB power)

### USB Cable Modification Safety

**Before you cut:**
- ✅ Verify your printer MCU has onboard 5V regulation (most do)
- ✅ Check board documentation
- ✅ Understand you're making a permanent modification

**Alternative solutions:**
- Buy a "USB data-only cable" (commercially available)
- Use a USB isolator
- Use a powered USB hub (may help some scenarios)

---

## Files Modified

### System Configuration

1. `/etc/udev/rules.d/50-usb-autosuspend-ch340.rules`
   - Disables USB autosuspend for CH340 device

2. `~/klippy-env/` (Python virtual environment)
   - pyserial upgraded from 3.4 to 3.5

### Hardware

3. USB Cable
   - RED (+5V) wire physically cut and insulated
   - Data-only configuration

---

## Success Metrics

**Testing Results:**
- ✅ 7 comprehensive tests passed
- ✅ 7 successful homing cycles
- ✅ 13+ minutes continuous operation
- ✅ Zero communication failures
- ✅ Zero new packet retransmissions
- ✅ Perfect stability under full electrical load

**System Status:**
- ✅ Production-ready
- ✅ Extensively tested
- ✅ All issues resolved
- ✅ Stable for printing

**Date Completed:** 2025-11-25 23:42

---

## Reference Information

### Related Documentation

- Klipper FAQ: https://www.klipper3d.org/FAQ.html#klipper-lost-communication-with-mcu
- CH341 Linux Driver: https://github.com/torvalds/linux/blob/master/drivers/usb/serial/ch341.c
- USB Power Management: https://www.kernel.org/doc/html/latest/driver-api/usb/power-management.html

### System Information

**Printer:** Sovol SV06  
**Host:** HP Pavilion 15-cc1xx laptop  
**OS:** Debian 13 (Trixie)  
**Klipper:** v0.13.0-401-g90b7f823  
**Moonraker:** v0.9.3-128-g960e933  
**Interface:** Mainsail @ http://10.0.0.139

---

**Total Troubleshooting Time:** 1 hour 33 minutes  
**Number of Different Root Causes:** 3  
**Final Status:** ✅ FULLY RESOLVED
