# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow Policy

**IMPORTANT:** Always ask the user before:
- Committing files to git
- Pushing commits to remote repositories
- Creating or modifying git tags
- Running destructive git operations

This applies to all repositories in this monorepo, including submodules.

## Documentation Strategy

This is a monorepo with multiple independent components managed as git submodules. Each submodule has its own CLAUDE.md file with detailed, component-specific documentation:

- **[halos-pi-gen/CLAUDE.md](halos-pi-gen/CLAUDE.md)**: Detailed pi-gen build system, stages, image variants, CI/CD workflows
- **[casaos-docker-service/CLAUDE.md](casaos-docker-service/CLAUDE.md)**: CasaOS deployment and configuration
- **[casaos-marine-store/CLAUDE.md](casaos-marine-store/CLAUDE.md)**: Marine app store structure and curation

**When to use which documentation:**
- Working in a specific submodule? Read that submodule's CLAUDE.md for detailed context
- Understanding the overall architecture? This file provides the big picture
- Making changes that span multiple components? Start here, then dive into submodule docs

This approach keeps context focused and relevant, improving efficiency and maintainability.

## Project Overview

Halos (Hat Labs Operating System) is a custom Raspberry Pi OS distribution with pre-installed web management tools. The name is a play on GLaDOS and HAL 9000.

**Core Components:**
- **Cockpit** (port 9090): Web-based system administration interface
- **CasaOS** (port 80/443): Container and app management with user-friendly UI

**Image Variants:**

Halos uses headless-first naming. Desktop variants include `-Desktop-` in the name.

| Image | Hardware | Desktop? | Marine? |
|-------|----------|----------|---------|
| **Halos-HALPI2** | HALPI2 | No | No |
| **Halos-Desktop-HALPI2** | HALPI2 | Yes | No |
| **Halos-Marine-HALPI2** | HALPI2 | No | Yes |
| **Halos-Desktop-Marine-HALPI2** | HALPI2 | Yes | Yes |
| **Halos-RPI** | Generic Pi | No | No |
| **Halos-Desktop-RPI** | Generic Pi | Yes | No |
| **Halos-Marine-RPI** | Generic Pi | No | Yes |
| **Halos-Desktop-Marine-RPI** | Generic Pi | Yes | Yes |

All images include Cockpit (web system admin) and CasaOS (container management). HALPI2 variants add hardware-specific drivers and the halpid daemon. Marine variants add Signal K, InfluxDB, and Grafana.

## Monorepo Structure

```
halos-distro/                          # Monorepo aggregator (this repo)
├── CLAUDE.md                          # Architecture overview (you are here)
└── submodules/
    ├── halos-pi-gen/                  # Halos image builder (pi-gen based)
    │   ├── CLAUDE.md                  # Detailed build system documentation
    │   ├── stage-halos-base/          # Cockpit + CasaOS for all variants
    │   ├── stage-halos-marine/        # Marine software stack
    │   ├── stage-halpi2-common/       # HALPI2 hardware support
    │   └── config.*                   # Image variant configurations
    ├── casaos-docker-service/         # CasaOS containerized deployment
    │   └── CLAUDE.md                  # CasaOS-specific documentation
    └── casaos-marine-store/           # Curated marine apps
        └── CLAUDE.md                  # App store documentation
```

**Note:** Legacy OpenPlotter and HALPI (CM4) images are built in a separate repository: `openplotter-halpi` (not part of this monorepo). That repository maintains Bookworm-based images for older Hat Labs hardware.

## High-Level Architecture

### Layer 1: Base Operating System
- Debian-based Raspberry Pi OS (arm64, trixie)
- Built using pi-gen (official Raspberry Pi image builder)
- Hardware support: Generic Raspberry Pi or HALPI2 (CM5-based) compute modules

### Layer 2: Web Management (All Variants)
- **Cockpit**: System monitoring, service management, terminal access, file management
- **CasaOS**: Docker container management, app store interface

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

Services run in Docker containers managed through CasaOS with persistent volumes.

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

### Building Images

See [halos-pi-gen/CLAUDE.md](halos-pi-gen/CLAUDE.md) for detailed build instructions. Quick reference:

```bash
cd halos-pi-gen
./run docker-build "Halos-Marine-HALPI2"  # Build specific variant
./run docker-build-all                     # Build all enabled variants
```

## Build and Release Pipeline

### Debian Package Building
Custom packages (like halpid firmware) are built in separate repositories:

1. Package built via GitHub Actions workflow
2. Workflow notifies apt.hatlabs.fi repository
3. apt.hatlabs.fi pulls and publishes updated packages
4. Packages become available during image builds

### Image Building
See [halos-pi-gen/CLAUDE.md](halos-pi-gen/CLAUDE.md) for CI/CD details:

1. **Pull Request Workflow**: Builds images on ARM64 runners, uploads artifacts
2. **Release Workflow**: Creates GitHub releases with built images

## Technology Stack

- **Base OS**: Debian-based Raspberry Pi OS (arm64, trixie)
- **Build System**: pi-gen (official Raspberry Pi image builder)
- **Web Management**: Cockpit (system admin), CasaOS (container/app management)
- **Containers**: Docker + Docker Compose
- **Marine Software**: Signal K, InfluxDB, Grafana
- **Hardware**: HALPI2 (CM5-based compute modules) and generic Raspberry Pi
- **HALPI2 Interfaces**: CAN bus, RS-485, I2C, UART
- **CI/CD**: GitHub Actions on ARM64 runners
- **Package Repository**: apt.hatlabs.fi

## Development Workflow

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

### Adding New Image Variants

See [halos-pi-gen/CLAUDE.md](halos-pi-gen/CLAUDE.md) for the detailed process. The variant system is based on:
- Hardware target (HALPI2 vs Generic Pi)
- Desktop environment (Desktop vs Lite)
- Software profile (Marine vs Non-marine)

## Important Notes

- Image builds require ARM64 architecture (use ARM64 runners or QEMU)
- All images include both Cockpit (port 9090) and CasaOS (port 80/443)
- Marine variants include pre-configured Signal K, InfluxDB, and Grafana
- HALPI2 variants include hardware-specific drivers and configurations
- Custom Debian packages are built separately and published to apt.hatlabs.fi
- Default boot (HALPI2) is from USB/NVMe (SD card disabled to prevent wear)
- Output images are in `.xz` compressed format
