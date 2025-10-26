# Halos Raspberry Pi OS Distribution

Halos is the Hat Labs Operating System, a custom Raspberry Pi OS distribution designed to help end-users run and maintain both Hat Labs and generic Raspberry Pi hardware with ease.

Halos is built on top of Raspberry Pi OS Lite (arm64, Trixie), adding tools for web-based management, hardware monitoring, and containerized services.

**Core Web Management:**
- **Cockpit** (port 9090): System administration, monitoring, terminal access
- **Runtipi** (port 80/443): Docker container management with app store

This repository acts as a workspace manager for all the components that make up Halos.

## Quick Start

```bash
# Clone all component repositories
./run repos:clone

# Update all repositories to latest
./run repos:pull-all-main

# Check status of all repositories
./run repos:status
```

## Repository Layout

- **halos-pi-gen/** - Custom Raspberry Pi image builder based on pi-gen
- **runtipi-marine-app-store/** - Custom marine app store for Runtipi
- **runtipi-docker-service/** - Runtipi Debian package
- **apt.hatlabs.fi/** - Custom APT repository for Halos packages

Each repository is independently managed. The `./run` script provides convenience commands for cloning and updating all repositories at once.
