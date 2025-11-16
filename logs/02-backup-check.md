# Step 2: Backup Check

**Date:** 2025-11-16

## Purpose
Check existing Klipper installation for any configs worth backing up before complete removal.

## Commands to Run

```bash
# Check for printer configs
ls -la ~/printer_data/config/ 2>/dev/null || echo "No printer_data directory"

# If it exists, show the printer.cfg
cat ~/printer_data/config/printer.cfg 2>/dev/null || echo "No printer.cfg found"

# Check klipper logs
ls -la ~/klipper_logs/ 2>/dev/null || echo "No klipper_logs"

# Check systemd services
systemctl list-units --type=service | grep -E 'klipper|moonraker'
```

## Output

(Waiting for command outputs...)

---

## Decision

After reviewing, we will proceed with complete removal and fresh installation.
