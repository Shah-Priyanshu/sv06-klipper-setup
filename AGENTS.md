# Agent Instructions

This repository contains documentation and configuration files for Klipperizing a Sovol SV06 3D printer running on a spare laptop with Linux Mint.

## Project Type

This is a **documentation and configuration repository**, not a software project. No build, lint, or test commands apply.

## Repository Purpose

- Document the process of installing and configuring Klipper on Linux Mint
- Store Klipper configuration files (`.cfg`) for the Sovol SV06
- Keep logs and troubleshooting notes from the setup process
- Track configuration changes and improvements over time

## File Conventions

- **Configuration Files:** Use `.cfg` extension for Klipper configs (e.g., `printer.cfg`, `macros.cfg`)
- **Documentation:** Use Markdown (`.md`) for logs, guides, and notes
- **Log Files:** Use numbered prefix convention: `00-`, `01-`, `02-`, etc. (e.g., `05-initial-setup.md`, `06-debian12-install-plan.md`)
- **Formatting:** Use 4 spaces for indentation in config files
- **Comments:** In `.cfg` files, use `#` for comments. Be descriptive about why settings were chosen.

## Agent Behavior

- **Be Assertive:** Make decisions proactively without asking for permission on formatting, naming conventions, and standard practices
- **Git Commits:** Commit changes regularly after completing tasks or making significant changes
- **Git Push:** Push commits to remote repository frequently to ensure work is backed up
- **Follow Conventions:** Always follow the established file naming and formatting conventions in this repository

## Key Information

- **Printer Model:** Sovol SV06
- **Host System:** HP Pavilion 15-cc1xx laptop with Debian 12
- **Hardware:** Intel i5-8250U, 8GB RAM, HDD + SSD configuration
- **Firmware:** Klipper
