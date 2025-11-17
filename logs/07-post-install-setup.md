# Post-Installation Setup - Debian 12

**Date:** 2025-11-17  
**Status:** Completed initial setup

## Installation Summary

- **OS:** Debian 12 (Bookworm)
- **Hardware:** HP Pavilion 15-cc1xx, Intel i5-8250U, 8GB RAM, 232GB HDD
- **User:** pri
- **Installation:** Successful with HDD-only configuration

## Partition Layout (Final)

```
sda1: 512MB   FAT32  /boot/efi (EFI System Partition)
sda2: 224GB   ext4   / (root)
sda3: 8GB     swap
```

## Post-Installation Steps Completed

### 1. Add User to Sudoers

**Issue:** User 'pri' was not in sudoers file

**Solution:**
- Rebooted into recovery mode
- Selected "Advanced options for Debian" → "recovery mode" → "root"
- Remounted filesystem: `mount -o remount,rw /`
- Added user to sudo group: `usermod -aG sudo pri`
- Rebooted

**Verification:**
```bash
groups
# Output should include 'sudo'

sudo whoami
# Should return: root
```

### 2. Enable Clamshell Mode

**Purpose:** Allow laptop to run with lid closed (headless operation)

**Configuration:**
```bash
sudo nano /etc/systemd/logind.conf
```

**Changes made:**
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

**Apply changes:**
```bash
sudo systemctl restart systemd-logind
```

**Verification:**
```bash
grep HandleLid /etc/systemd/logind.conf
```

### 3. Install and Configure SSH

**Installation:**
```bash
sudo apt update
sudo apt install -y openssh-server
```

**Enable and start SSH:**
```bash
sudo systemctl enable ssh
sudo systemctl start ssh
```

**Verification:**
```bash
sudo systemctl status ssh
# Should show: active (running)
```

**Network Configuration:**
- IP Address: `10.0.0.139`
- SSH Access: `ssh pri@10.0.0.139`

### 4. Configure Passwordless Sudo

**Purpose:** Allow sudo commands without password (required for automation)

**Configuration:**
```bash
echo "pri ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/pri
sudo chmod 440 /etc/sudoers.d/pri
```

**Verification:**
```bash
sudo whoami
# Should return: root (without password prompt)
```

### 5. Setup SSH Key Authentication

**Purpose:** Passwordless SSH from main Windows computer

**On main computer (Windows PowerShell):**
```powershell
# Generate SSH key
ssh-keygen -t ed25519 -C "pri@main-computer"

# Copy public key to laptop
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh pri@10.0.0.139 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

**Verification:**
```powershell
ssh pri@10.0.0.139 "echo 'SSH key authentication working!'"
```

### 6. System Updates and Dependencies

**Update system:**
```bash
sudo apt update
sudo apt upgrade -y
```

**Install build tools and Klipper dependencies:**
```bash
sudo apt install -y git python3 python3-pip python3-venv build-essential libncurses-dev libffi-dev libssl-dev
```

**Packages installed:**
- git (version control)
- python3 + python3-dev (Python 3.13.5)
- python3-pip (package manager)
- python3-venv (virtual environments)
- build-essential (gcc, g++, make)
- libncurses-dev, libffi-dev, libssl-dev (development libraries)

## System Status

- [x] Debian 13 (Trixie) installed successfully
- [x] User has sudo access (passwordless)
- [x] Clamshell mode enabled (lid close ignored)
- [x] SSH server running and accessible
- [x] SSH key authentication configured
- [x] Network connectivity confirmed
- [x] System updated and dependencies installed
- [ ] Desktop environment (pending - optional)
- [ ] Klipper installation (in progress)

## Next Steps

1. **Optional:** Install Xfce desktop environment
   ```bash
   sudo apt install xfce4 xfce4-goodies
   ```

2. **Install Klipper dependencies**
   - Git
   - Python 3
   - Build tools
   - Klipper itself

3. **Restore Klipper configuration** from this repository

4. **Connect printer** and test configuration

## Notes

- System is now accessible via SSH for all future configuration
- Can operate headless with lid closed
- HDD-only configuration eliminates HP Pavilion EFI boot issues
- Ready for Klipper installation
