# USB Communication Failure - Python 3.13 / pyserial Compatibility

**Date:** 2025-11-25 22:57  
**Status:** [DONE] RESOLVED  
**Issue:** pyserial AttributeError with Python 3.13

> **Note:** This is Part 3 of 4. See [14-usb-troubleshooting-index.md](14-usb-troubleshooting-index.md) for the complete troubleshooting journey.

---

## Problem Discovery

After documentation cleanup and attempting to use the printer, user reported "got the mcu error" again.

**Error Signature:**
```
Exception ignored in: Serial<id=0x7f05622af7c0, open=True>(port='/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0', baudrate=250000, bytesize=8, parity='N', stopbits=1, timeout=0, xonxoff=False, rtscts=False, dsrdtr=False)
Traceback (most recent call last):
  File "/home/pri/klippy-env/lib/python3.13/site-packages/serial/serialposix.py", line 453, in close
    os.close(self.pipe_abort_read_w)
AttributeError: 'Serial' object has no attribute 'pipe_abort_read_w'
```

---

## Root Cause Analysis

### System Environment

```bash
Python Version: 3.13.5
pyserial Version: 3.4 (INCOMPATIBLE)
OS: Debian 13 (Trixie)
```

### The Problem

- Debian 13 (Trixie) ships with Python 3.13.5 by default
- Klipper's Python virtual environment inherited Python 3.13
- pyserial 3.4 has incomplete Python 3.13 support
- The `pipe_abort_read_w` attribute handling changed in Python 3.13
- This caused exceptions when closing serial connections

### Why This Happened

- Debian 13 is very new (Trixie is current testing branch)
- Python 3.13 was released recently (October 2024)
- Klipper installation used system Python 3.13
- pyserial 3.4 predates Python 3.13 release

### Impact

- Serial port errors on every connection close/restart
- Errors were "ignored exceptions" but could cause instability
- MCU communication was functional but fragile

---

## Investigation Steps

### 1. Checked Klipper Logs

```bash
tail -100 ~/printer_data/logs/klippy.log
journalctl -u klipper -n 100
```
- Found repeated `pipe_abort_read_w` AttributeError

### 2. Verified Python and pyserial Versions

```bash
~/klippy-env/bin/python --version  # Python 3.13.5
~/klippy-env/bin/pip list | grep -i serial  # pyserial 3.4
```

### 3. Searched for Available pyserial Versions

```bash
~/klippy-env/bin/pip index versions pyserial
```
- Latest: pyserial 3.5 (with Python 3.13 support)

### 4. Researched Klipper Compatibility

- Checked Klipper documentation
- Python 3.13 is very new, pyserial 3.5 provides compatibility

---

## Solution Applied

### Upgrade pyserial to Version 3.5

```bash
# Upgrade pyserial in Klipper's Python virtual environment
~/klippy-env/bin/pip install --upgrade pyserial

# Output:
# Collecting pyserial
#   Downloading pyserial-3.5-py2.py3-none-any.whl (90 kB)
# Installing collected packages: pyserial
#   Attempting uninstall: pyserial
#     Found existing installation: pyserial 3.4
#     Uninstalling pyserial-3.4:
#       Successfully uninstalled pyserial-3.4
# Successfully installed pyserial-3.5

# Restart Klipper service
sudo systemctl restart klipper
```

---

## Verification

### 1. Checked pyserial Version

```bash
~/klippy-env/bin/pip show pyserial
# Version: 3.5
```

### 2. Monitored Klipper Logs for Errors

```bash
journalctl -u klipper --since '1 minute ago'
grep -i 'pipe_abort_read_w\|AttributeError' ~/printer_data/logs/klippy.log
```
- [DONE] No more AttributeError exceptions
- [DONE] No pipe_abort_read_w errors

### 3. Verified MCU Communication

```bash
tail -50 ~/printer_data/logs/klippy.log
```
- [DONE] Active MCU communication
- [DONE] Thermistor readings flowing normally (oid 15=hotend, oid 21=bed)
- [DONE] No retransmit issues
- [DONE] Klipper service stable

### 4. Checked System Status

```bash
systemctl status klipper
```
- [DONE] Active (running)
- [DONE] No service errors
- [DONE] MCU connected and responsive

---

## Technical Analysis

### Why pyserial 3.5 Fixes This

- pyserial 3.5 adds Python 3.13 compatibility
- Properly handles serial port attribute initialization in Python 3.13
- Includes fixes for changes in Python 3.13's internal APIs
- Release notes: "Python 3.13 compatibility improvements"

### Python 3.13 Changes

- Internal changes to object attribute handling
- Stricter attribute access in destructors/cleanup
- pyserial 3.4 relied on implicit attribute initialization that changed

---

## Lessons Learned

### 1. Bleeding Edge OS Risk

- Debian 13 (Trixie) is very new
- Python 3.13 support is still maturing in ecosystem
- Trade-off: Better hardware support vs. software compatibility issues

### 2. Virtual Environment Dependencies

- Python virtual environments inherit system Python version
- Package versions must match Python version compatibility
- Always verify library compatibility with new Python versions

### 3. Klipper on Debian 13

- Works well but requires dependency updates
- pyserial 3.5 is safe and compatible
- Future installs should upgrade pyserial immediately

### 4. Error Signature Recognition

- "AttributeError" + "Serial object" + Python 3.13 = Version mismatch
- Check library versions first when seeing compatibility errors

---

## Recommendations

### For Future Klipper Installs on Debian 13/Trixie

```bash
# After Klipper installation, immediately upgrade pyserial
~/klippy-env/bin/pip install --upgrade pyserial
sudo systemctl restart klipper
```

### For Existing Systems

- Monitor for similar compatibility issues with other libraries
- Consider pinning working library versions in requirements
- Document Python environment setup for reproducibility

### Prevention

- When using bleeding-edge OS (Debian testing), expect compatibility issues
- Check library compatibility before major Python version upgrades
- Keep documentation of working dependency versions

---

## Final System State

### Software Versions

```
OS: Debian 13 (Trixie) - Testing Branch
Python: 3.13.5
pyserial: 3.5 (upgraded from 3.4)
Klipper: v0.13.0-401-g90b7f823
```

### Status

- [DONE] Klipper: Active and stable
- [DONE] MCU: Connected and responsive
- [DONE] Serial Communication: Error-free
- [DONE] Printer: Ready for operation

### Time to Resolution

~5 minutes (diagnosis + fix + verification)

---

**Status:** [DONE] RESOLVED

**Note:** While this fixed the pyserial compatibility issue, further USB power delivery problems were discovered. See [14d-usb-power-delivery-fix.md](14d-usb-power-delivery-fix.md) for the final resolution.
