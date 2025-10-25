# Halos Raspberry Pi OS Distribution

Halos is the Hat Labs Operating System, a custom Raspberry Pi OS distribution designed to help end-users run and maintain both Hat Labs and generic Raspberry Pi hardware with ease.

Halos is built on top of Raspberry Pi OS Lite (arm64, Trixie), adding tools for web-based management, hardware monitoring, and containerized services.

**Core Web Management:**
- **Cockpit** (port 9090): System administration, monitoring, terminal access
- **Runtipi** (port 80/443): Docker container management with app store

This repository acts as a monorepo for all the components that make up Halos.

## Repository Layout

- **halos-pi-gen/** - Custom Raspberry Pi image builder based on pi-gen (submodule)
- **runtipi-marine-app-store/** - Custom marine app store for Runtipi (submodule)
- **apt.hatlabs.fi/** - Custom APT repository for Halos packages (submodule)
- **casaos-marine-store/** - Legacy CasaOS marine app store (submodule, deprecated)
