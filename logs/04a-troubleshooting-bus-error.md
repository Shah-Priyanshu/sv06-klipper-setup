# Troubleshooting: Bus Error During KIAUH Clone

**Date:** 2025-11-16
**Issue:** Bus error when cloning KIAUH repository

## Error Encountered

```
user@Klipper:~$ git clone https://github.com/dw-0/kiauh.git
Bus error
```

## Possible Causes

1. **Memory issue** - Corrupted RAM or disk
2. **Disk space full** - No space left on device
3. **Filesystem corruption** - Damaged filesystem
4. **Git cache corruption** - Corrupted git cache

## Troubleshooting Steps

### 1. Check Disk Space
```bash
df -h
```

### 2. Check Memory
```bash
free -h
```

### 3. Check for Filesystem Errors
```bash
dmesg | tail -20
```

### 4. Clear Git Cache and Retry
```bash
rm -rf ~/.gitconfig.lock
git config --global core.compression 0
git clone --depth 1 https://github.com/dw-0/kiauh.git
```

### 5. Alternative: Download as ZIP
```bash
cd ~
wget https://github.com/dw-0/kiauh/archive/refs/heads/master.zip -O kiauh.zip
unzip kiauh.zip
mv kiauh-master kiauh
cd kiauh
./kiauh.sh
```

### 6. Manual Installation (if KIAUH continues to fail)
We can install Klipper manually without KIAUH if needed.

---

## Commands to Run

```bash
# Check system health
df -h
free -h
dmesg | tail -20

# Try shallow clone
git clone --depth 1 https://github.com/dw-0/kiauh.git

# If that fails, try wget method
wget https://github.com/dw-0/kiauh/archive/refs/heads/master.zip -O kiauh.zip
unzip kiauh.zip
mv kiauh-master kiauh
```

---

## Output

(Paste troubleshooting command outputs below)
