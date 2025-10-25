# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow Policy

For git workflow policies that apply to all repositories in this monorepo, see the Git Workflow Policy section in @halos-pi-gen/CLAUDE.md

## Documentation Strategy

This is a monorepo with multiple independent components managed as git submodules. Each submodule has its own CLAUDE.md file with detailed, component-specific documentation:

- **[halos-pi-gen/CLAUDE.md](halos-pi-gen/CLAUDE.md)**: Detailed pi-gen build system, stages, image variants, CI/CD workflows
- **[runtipi-marine-app-store/CLAUDE.md](runtipi-marine-app-store/CLAUDE.md)**: Runtipi marine app store structure and curation
- **[casaos-marine-store/CLAUDE.md](casaos-marine-store/CLAUDE.md)**: CasaOS marine app store (legacy, transitioning to Runtipi)

**When to use which documentation:**
- Working in a specific submodule? Read that submodule's CLAUDE.md for detailed context
- Understanding the overall architecture? This file provides the big picture
- Making changes that span multiple components? Start here, then dive into submodule docs

This approach keeps context focused and relevant, improving efficiency and maintainability.

## Project Overview

Halos (Hat Labs Operating System) is a custom Raspberry Pi OS distribution with pre-installed web management tools. The name is a play on GLaDOS and HAL 9000.

**Core Components:**
- **Cockpit** (port 9090): Web-based system administration interface
- **Runtipi** (port 80/443): Container and app management with user-friendly UI

## Image Variants

For detailed information about image variants, naming conventions, and configuration options, see the Image Variants section in @halos-pi-gen/CLAUDE.md

## Monorepo Structure

```
halos-distro/                          # Monorepo aggregator (this repo)
├── CLAUDE.md                          # Architecture overview (you are here)
└── submodules/
    ├── halos-pi-gen/                  # Halos image builder (pi-gen based)
    │   ├── CLAUDE.md                  # Detailed build system documentation
    │   ├── stage-halos-base/          # Cockpit + Runtipi for all variants
    │   ├── stage-halos-marine/        # Marine software stack
    │   ├── stage-halpi2-common/       # HALPI2 hardware support
    │   └── config.*                   # Image variant configurations
    ├── runtipi-marine-app-store/      # Runtipi marine app store
    │   └── CLAUDE.md                  # App store documentation
    ├── casaos-marine-store/           # CasaOS marine apps (legacy)
    │   └── CLAUDE.md                  # App store documentation
    └── apt.hatlabs.fi/                # Custom APT repository
```

**Note:** Legacy OpenPlotter and HALPI (CM4) images are built in a separate repository: `openplotter-halpi` (not part of this monorepo). That repository maintains Bookworm-based images for older Hat Labs hardware.

## High-Level Architecture

### Layer 1: Base Operating System
- Debian-based Raspberry Pi OS (arm64, trixie)
- Built using pi-gen (official Raspberry Pi image builder)
- Hardware support: Generic Raspberry Pi or HALPI2 (CM5-based) compute modules

### Layer 2: Web Management (All Variants)
- **Cockpit**: System monitoring, service management, terminal access, file management
- **Runtipi**: Docker container management, app store interface

### Layer 3: Hardware Support (HALPI2 Variants Only)
- Hat Labs APT repository (apt.hatlabs.fi)
- Hardware daemon (halpid) and firmware
- I2C, CAN bus, RS-485, UART interfaces
- External antenna support
- USB/NVMe boot priority (SD card disabled)

### Layer 4: Marine Services (Marine Variants Only)
```
Marine sensors → Signal K (port 3000) → InfluxDB (port 8086)
                                              ↓
                                          Grafana (port 3001)
```

Services run in Docker containers managed through Runtipi with persistent volumes.

## Common Development Commands

### Working with Submodules

```bash
# Initialize all submodules
git submodule update --init --recursive

# Update all submodules to latest
git submodule update --remote

# Update specific submodule
cd halos-pi-gen
git pull origin main
cd ..
git add halos-pi-gen
git commit -m "Update halos-pi-gen submodule"
```

## Build and Release Pipeline

For detailed information about the build and release pipeline, including Debian package building and CI/CD workflows, see the Build and Release Pipeline section in @halos-pi-gen/CLAUDE.md

## Technology Stack

For comprehensive technology stack information, see the Technology Stack section in @halos-pi-gen/CLAUDE.md

## Development Workflow

For component-specific development workflows and patterns, see the Creating New Image Variants and Common Development Patterns sections in @halos-pi-gen/CLAUDE.md

### Making Changes to a Specific Component

1. Navigate to the appropriate submodule directory
2. Read that submodule's CLAUDE.md for detailed context
3. Make changes following the patterns documented there
4. Test locally if applicable
5. Commit in the submodule, then update the parent repo reference

### Making Cross-Component Changes

1. Review this architecture overview
2. Identify which submodules are affected
3. Read the CLAUDE.md for each affected submodule
4. Make coordinated changes
5. Test the integration
6. Update submodule references in parent repo

## Important Notes

- All images include both Cockpit (port 9090) and Runtipi (port 80/443)
- Marine variants include pre-configured Signal K, InfluxDB, and Grafana
- HALPI2 variants include hardware-specific drivers and configurations
- Default boot (HALPI2) is from USB/NVMe (SD card disabled to prevent wear)
