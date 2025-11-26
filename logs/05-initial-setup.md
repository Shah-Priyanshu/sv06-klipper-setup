# Initial Setup - Hardware Decision

**Date:** 2025-11-16  
**Status:** [DONE] COMPLETED - Decided on fresh Debian 13 installation

---

## Context

After discovering critical disk/filesystem issues in log 04b, a decision was needed on how to proceed with the Klipper printer setup.

## Problem Summary

The existing Debian 11 installation on the HP Pavilion 15-cc1xx laptop had:
- [FAIL] Filesystem corruption (Input/output errors on system binaries)
- [FAIL] Bus errors when running commands
- [FAIL] Unreliable dual-disk configuration (HDD + SSD with EFI boot issues)
- [FAIL] Unknown system state and configuration

## Hardware Review

**HP Pavilion 15-cc1xx Laptop:**
- **CPU:** Intel Core i5-8250U (4 cores, 8 threads, 1.6GHz base)
- **RAM:** 8GB DDR4
- **Storage:**
  - **SSD (sda):** 128GB - Used for EFI boot
  - **HDD (sdb):** 232GB Seagate ST9250411AS - Main OS drive
- **USB Ports:** Multiple USB 3.0 and USB 2.0 ports
- **Network:** Built-in Ethernet and WiFi
- **Age:** Older model with known EFI boot issues in dual-disk configs

## Decision: Fresh Installation

**Rationale:**

1. **Filesystem Corruption:** Attempting to repair was risky and unreliable
2. **Dual-Disk Complexity:** HP Pavilion 15-cc1xx has known EFI boot problems with SSD+HDD setups
3. **Clean Slate:** Fresh install ensures no hidden issues or misconfigurations
4. **Modern OS:** Upgrade to Debian 13 (Trixie) for better hardware support
5. **Simplified Configuration:** Single HDD-only setup eliminates EFI complications

**Alternative Considered:**
- Boot from USB with persistent storage
- Replace failing drive with new SSD
- Use Raspberry Pi 4 instead

**Chosen Path:**
- [DONE] Remove SSD completely
- [DONE] Fresh Debian 13 installation on HDD only
- [DONE] Simple partition layout (EFI + root + swap on single drive)

## Storage Configuration Decision

**Selected Configuration: HDD-Only**

**Why HDD instead of SSD:**
1. **SSD was only 128GB** - Too small for OS + Klipper + future growth
2. **HDD is 232GB** - Plenty of space for Klipper system
3. **Avoids dual-disk EFI issues** - HP Pavilion BIOS struggles with mixed configs
4. **HDD performance sufficient** - Klipper doesn't require SSD speeds
5. **Physical removal of SSD** - Eliminates any boot confusion

**Trade-offs Accepted:**
- Slower boot times (acceptable for dedicated printer system)
- Slightly slower file operations (negligible for Klipper)
- HDD mechanical reliability concerns (mitigated by good backups)

## Planned Partition Layout

**Single HDD (232GB):**
```
sda1: 512MB   FAT32  /boot/efi (EFI System Partition)
sda2: 224GB   ext4   / (root filesystem)
sda3: 8GB     swap
```

**Advantages:**
- Simple, single-disk configuration
- Large root partition for all Klipper data
- Proper swap for 8GB RAM system
- EFI partition on same disk as root (no cross-disk boot issues)

## Installation Method

**Debian 13 (Trixie) - Testing Branch**

**Why Debian 13 instead of Debian 12:**
- Better hardware support for Intel 8th gen CPU
- More recent kernel for USB device compatibility
- Klipper development often uses latest stable packages
- Testing branch is stable enough for dedicated appliance use

**Installation Media:**
- Debian 13 net-install ISO
- USB drive for installation
- Network connection required during install

## Headless Operation Plan

**Clamshell Mode Setup:**
- Laptop will run with lid closed
- No monitor/keyboard/mouse needed after setup
- SSH-only access from Windows PC
- Printer accessible via web interface (Mainsail)

**Network Configuration:**
- Static IP: 10.0.0.139 (planned)
- SSH server enabled
- Key-based authentication (no password)
- Firewall rules for Klipper services

## Next Steps

Proceed to:
- **Log 06:** Debian 13 Installation Plan (detailed partition and package selection)
- **Log 07:** Post-Install Setup (SSH, static IP, dependencies)
- **Log 08:** Klipper Installation

---

## Outcome

This decision proved correct:
- [DONE] Debian 13 installation was smooth and successful
- [DONE] Single-HDD configuration eliminated all EFI boot issues
- [DONE] System has been stable and reliable for Klipper operation
- [DONE] 232GB HDD provides plenty of space with room for growth

**Status:** Decision validated by successful installation and stable operation.
