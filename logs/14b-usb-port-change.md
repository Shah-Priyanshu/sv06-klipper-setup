# USB Communication Failure - Port 2 Issue & Port Reassignment

**Date:** 2025-11-25 22:30-22:40  
**Status:** ✅ RESOLVED (Temporarily - root cause was different)  
**Issue:** Second communication failure, traced to faulty USB Port 2

> **Note:** This is Part 2 of 4. See [14-usb-troubleshooting-index.md](14-usb-troubleshooting-index.md) for the complete troubleshooting journey.

---

## Problem Discovery

**2025-11-25 22:30** - Second communication failure occurred  
**2025-11-25 22:35** - User observation: Camera previously had device ID instability on same USB port  
**2025-11-25 22:40** - **HYPOTHESIS: Faulty USB Port 2 on laptop**

### Context

After implementing the USB autosuspend fix (see [14a-initial-failure-autosuspend-fix.md](14a-initial-failure-autosuspend-fix.md)), the printer worked briefly but then experienced a second communication failure during normal operation.

---

## Diagnostic Evidence

### USB Topology Before Fix (Port 2 - SUSPECTED FAULTY)

```
Port 002: Dev 006, CH340 Printer ← UNSTABLE PORT
Port 001: Dev 002, XWF-1080P Camera
Port 003: Dev 004, Built-in HP camera
```

### User's Key Observation

> "In my earlier setup I have the camera connected in that port [Port 2] and the Device id kept on changing from video0 to video2 often."

This was the critical clue! Device ID instability indicates:
- Physical port damage or wear
- Bad solder joints on port connector
- Faulty USB controller/hub for that specific port

### Evidence Supporting Port 2 Failure

1. Camera previously experienced device ID instability on Port 2
2. Printer experienced multiple disconnects on Port 2
3. Port 2 has history of unreliability with different devices
4. USB autosuspend was already disabled (not the cause)

---

## Solution Implemented: USB Port Reconfiguration

### New USB Topology

```
Port 001: Dev 051, CH340 Printer ← MOVED HERE (GOOD PORT)
Port 004: Dev 052, USB-C Hub → XWF-1080P Camera ← MOVED TO HUB
Port 003: Dev 004, Built-in HP camera (unchanged)
```

### Actions Taken

1. Unplugged printer from Port 2 (faulty)
2. Moved printer to Port 1 (the port that previously worked fine for camera)
3. Moved camera to USB-C hub (isolates camera from printer's USB bus)

---

## Testing Results After Port Change

### Test 1: Single G28 Homing

- ✅ Result: SUCCESS
- `bytes_retransmit=0` (perfect communication)
- No disconnection errors

### Test 2: Stress Test - 5 Consecutive G28 Cycles

- ✅ Result: ALL SUCCESSFUL
- `bytes_retransmit=0` throughout entire test
- `send_seq=3474`, `receive_seq=3474` (perfect packet sync)
- `print_time` advanced from 90.269 to 110.046 (all homing completed)
- Printer state: "ready" (operational)

---

## Performance Comparison

| Metric | Port 2 (Faulty) | Port 1 (Good) |
|--------|----------------|---------------|
| Disconnects | Multiple | 0 |
| bytes_retransmit | 2345+ | 0 |
| I/O Errors | Yes | No |
| Stability | Failed within minutes | Stable for extended testing |
| Device Binding | Lost after disconnect | Maintained |

---

## Root Cause Analysis (At This Point)

### Hypothesis: Hardware Issue - Faulty USB Port 2

**Evidence Chain:**
1. Camera previously experienced device ID instability on Port 2
2. Printer experienced multiple disconnects on Port 2
3. After moving printer to Port 1: ZERO issues (initially)
4. USB autosuspend was already disabled (not the cause)
5. CH340 driver working correctly (proved by Port 1 stability)

### Initial Conclusion (Later Proven Wrong)

- ❌ **NOT** a CH340 chip problem
- ❌ **NOT** an electrical noise problem
- ❌ **NOT** a USB power management problem
- ✅ **HYPOTHESIS** - Hardware failure in laptop's USB Port 2

**Probable Cause (Suspected):**
- Worn/damaged USB port connector
- Bad solder joint on motherboard
- Intermittent connection due to physical defect
- Common issue in aging laptops (HP Pavilion 15-cc1xx is older model)

---

## Hardware Configuration After Port Change

**USB Device:**
- Port: USB Port 1 (laptop left side)
- Device: Bus 001 Device 051: ID 1a86:7523 QinHeng Electronics CH340
- Serial: `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0` → `/dev/ttyUSB0`
- Autosuspend: Disabled (`-1`)

**Other Devices:**
- Camera: Moved to USB-C hub (Port 4)
- Built-in camera: Port 3 (unchanged)

---

## Status After Port Change

**Hardware Configuration:**
- ✅ Printer on Port 1 (stable, verified)
- ✅ Camera on USB-C Hub (isolated, stable)
- ✅ USB autosuspend disabled (preventive measure)
- ✅ Tested with 5+ homing cycles (no failures)

**Recommendations:**
- ⚠️ **Avoid using USB Port 2** for any critical devices
- ✅ Continue using Port 1 for printer (proven stable)
- ✅ Consider marking Port 2 with tape/label ("DO NOT USE - FAULTY")
- 📝 If Port 1 eventually fails, consider external USB hub or new host machine

---

## Lessons Learned (From This Stage)

1. **Historical Device Behavior Matters:**
   - Previous instability with other devices on same port is a strong indicator
   - User observations about device ID changes were the key clue

2. **Test Different Ports:**
   - Don't assume all USB ports have equal quality
   - Older laptops can have port-specific hardware failures

3. **Isolate Devices:**
   - Moving camera to hub isolated potential interference
   - Reduces load on laptop's USB controller

---

## Why This Fix Was Only Temporary

**IMPORTANT:** This fix appeared to work initially, but problems recurred later. The TRUE root cause was not a faulty Port 2, but rather:

1. **Python 3.13 / pyserial compatibility issue** (see [14c-pyserial-compatibility.md](14c-pyserial-compatibility.md))
2. **USB power delivery problems** affecting BOTH ports (see [14d-usb-power-delivery-fix.md](14d-usb-power-delivery-fix.md))

The port change coincidentally worked temporarily because:
- It involved physically reconnecting the USB cable (reset)
- Testing was done with cold printer (no heaters active)
- Full electrical load testing wasn't performed at this stage

---

**Status:** ✅ RESOLVED (Temporarily)

**Actual Outcome:** Port 2 may not have been faulty at all. The real issues were:
1. Software compatibility (pyserial)
2. USB power delivery problems affecting all ports under high electrical load

**Next Steps:** See remaining logs for the complete troubleshooting journey:
- [14c-pyserial-compatibility.md](14c-pyserial-compatibility.md) - Python 3.13 compatibility issue
- [14d-usb-power-delivery-fix.md](14d-usb-power-delivery-fix.md) - Final fix (USB cable modification)
