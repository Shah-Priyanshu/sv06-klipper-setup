# Debian 12 Installation Plan for HP Pavilion 15-cc1xx

**Date:** 2025-11-17  
**Purpose:** Fresh Debian 12 install on HDD only (SSD removed)

## Hardware Configuration

- **Model:** HP Pavilion 15-cc1xx
- **CPU:** Intel Core i5-8250U (4 cores, 8 threads)
- **RAM:** 8GB
- **Storage:**
  - **HDD (sda):** 232.9GB Seagate ST9250411AS

## Configuration Notes

- SSD removed to simplify installation
- Single HDD configuration eliminates HP Pavilion EFI boot issues
- 232GB is sufficient for Debian 12 + Klipper + future updates

## Partition Plan

### HDD (sda) - All partitions
- **sda1:** 512MB, FAT32, EFI System Partition (ESP)
  - Mount point: `/boot/efi`
  - Bootable flag: YES
  - This is where GRUB will be installed

- **sda2:** 224GB, ext4, Root filesystem
  - Mount point: `/`
  - This is where the entire OS will live

- **sda3:** 8GB, swap
  - Swap space (equal to RAM)

## Installation Steps

1. **Download Debian 12**
   - URL: https://www.debian.org/download
   - Version: Debian 12 (Bookworm) netinstall or full ISO
   - Architecture: amd64

2. **Create Bootable USB**
   - Use Rufus (Windows), Etcher (cross-platform), or dd (Linux)
   - Partition scheme: GPT
   - Target system: UEFI

3. **Backup Important Data**
   - Klipper configs (already backed up in this repo)
   - Any other important files

4. **Boot and Install**
   - Boot from USB
   - Select "Graphical Install" or "Install"
   - Follow prompts until partitioning step

5. **Manual Partitioning (CRITICAL STEP)**
   
   **For HDD (sda):**
   - Delete all existing partitions
   
   - Create partition 1 (EFI):
     - Size: 512MB
     - Type: EFI System Partition
     - Format: FAT32
     - Mount: `/boot/efi`
     - Bootable: YES
   
   - Create partition 2 (Root):
     - Size: 224GB (use remaining space minus 8GB)
     - Type: ext4
     - Mount: `/` (root)
   
   - Create partition 3 (Swap):
     - Size: 8GB
     - Type: Linux swap

6. **GRUB Installation**
   - When asked "Install GRUB bootloader", select YES
   - Choose `/dev/sda` (the HDD) as the target device

7. **Post-Install Verification**
   ```bash
   # Check EFI boot entries
   sudo efibootmgr -v
   
   # Verify mount points
   lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINT
   
   # Check GRUB location
   sudo fdisk -l /dev/sda
   ```

## Notes

- Simple single-drive configuration
- HDD is sufficient for Klipper hosting (not IO-intensive)
- 232GB provides plenty of space for OS, logs, and future expansion
- Desktop environment can be added later if needed (Xfce recommended)

## Next Steps After Installation

1. Update system: `sudo apt update && sudo apt upgrade`
2. Install SSH: `sudo apt install openssh-server`
3. Install desktop (optional): `sudo apt install xfce4 xfce4-goodies`
4. Install Klipper dependencies
5. Restore Klipper configuration from this repo
