# Agent Instructions

This repository contains documentation and configuration files for Klipperizing a Sovol SV06 3D printer running on a spare laptop with Debian 13.

## Project Type

This is a **documentation and configuration repository**, not a software project. No build, lint, or test commands apply.

## Repository Purpose

- Document the process of installing and configuring Klipper on Debian 13
- Store Klipper configuration files (`.cfg`) for the Sovol SV06
- Keep logs and troubleshooting notes from the setup process
- Track configuration changes and improvements over time

## Session Initialization

**At the start of EVERY session, you MUST:**

1. **Read all log files** in the `logs/` directory in sequential order (00-, 01-, 02-, etc.)
2. **Understand the current state** of the system by reviewing what has been completed
3. **Identify any pending tasks** from the most recent log file
4. **Review configuration files** in `configs/` to understand the current printer setup

This ensures you have full context about:
- What has been done previously
- Current system configuration
- Any issues encountered
- Next steps in the setup process

**Do this automatically without being prompted by the user.**

## File Conventions

- **Configuration Files:** Use `.cfg` extension for Klipper configs (e.g., `printer.cfg`, `macros.cfg`)
- **Documentation:** Use Markdown (`.md`) for logs, guides, and notes
- **Log Files:** Use numbered prefix convention: `00-`, `01-`, `02-`, etc. (e.g., `05-initial-setup.md`, `06-debian12-install-plan.md`)
- **Formatting:** Use 4 spaces for indentation in config files
- **Comments:** In `.cfg` files, use `#` for comments. Be descriptive about why settings were chosen.

## Agent Behavior

- **Be Assertive:** Make decisions proactively without asking for permission on formatting, naming conventions, and standard practices
- **Git Commits:** Commit changes regularly after completing tasks or making significant changes
- **Git Push:** Push commits to remote repository frequently to ensure work is backed up (note: push may fail due to auth, user will handle manually)
- **Follow Conventions:** Always follow the established file naming and formatting conventions in this repository
- **Update Logs:** Update log files after completing significant steps, commit immediately
- **Document Everything:** Keep detailed records of all configuration changes, commands run, and decisions made
- **Run Verification Commands:** When guiding the user through a process, run verification commands via SSH automatically to check results. Do not ask the user to run simple checks - be proactive and verify things yourself. Only ask the user to perform actions that require physical interaction (like inserting SD cards, pressing buttons, etc.) or viewing output directly on their terminal.

## ⚠️ CRITICAL: User-Guided Installation Protocol

**IMPORTANT: When performing installations or system modifications:**

1. **GUIDE, DON'T EXECUTE:** Your role is to GUIDE the user through the installation process, NOT to execute installation commands automatically
2. **WAIT FOR CONFIRMATION:** After providing instructions, WAIT for the user to explicitly ask you to proceed before running ANY installation commands
3. **ASK FIRST:** Before running any `apt install`, `git clone`, or system modification commands, TELL the user what you plan to do and WAIT for their permission
4. **User is in Control:** The user decides when each step happens. Never assume they want you to execute the next step automatically.

**Example of CORRECT behavior:**
```
Agent: "The next step is to install nginx. Here's the command:
        sudo apt install -y nginx
        
        Would you like me to run this command now?"
User: "yes"
Agent: [Runs the command]
```

**Example of INCORRECT behavior:**
```
Agent: [Automatically runs: sudo apt install -y nginx]
```

**Exception:** Read-only verification commands (like `systemctl status`, `ls`, `cat`, etc.) can be run automatically to check system state.

## Key Information

- **Printer Model:** Sovol SV06
- **Host System:** HP Pavilion 15-cc1xx laptop with Debian 13 (Trixie)
- **Hardware:** Intel i5-8250U, 8GB RAM, 232GB HDD (single drive configuration)
- **Network:** IP Address 10.0.0.139
- **SSH Access:** `ssh pri@10.0.0.139` (passwordless with SSH keys)
- **Firmware:** Klipper (installation in progress)

## Current Status

As of last update:
- Debian 13 installed with HDD-only configuration (no EFI boot issues)
- Clamshell mode enabled (headless operation)
- SSH configured with key authentication
- System updated and build dependencies installed
- Klipper Python virtual environment created at `~/klippy-env`
- Klipper repository cloned to `~/klipper`
- Next: Complete Klipper service setup, install Moonraker and web interface
