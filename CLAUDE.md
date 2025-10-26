# HaLOS Monorepo

Halos (Hat Labs Operating System) - custom Raspberry Pi OS distribution with web management.

The monorepo structure is for convenience only! Each submodule must work independently and can have no directory-level cross-dependencies.

## Git Workflow Policy

**IMPORTANT:** Always ask before committing, pushing, tagging, or running destructive git operations.

## Structure

```
halos-distro/
├── halos-pi-gen/                  # Image builder (submodule)
├── runtipi-marine-app-store/      # Marine app store (submodule)
├── runtipi-docker-service/        # Runtipi Debian package
└── apt.hatlabs.fi/                # Custom APT repo (submodule)
```

**Each submodule has its own CLAUDE.md** - read the appropriate one for detailed context.

## Quick Start

```bash
# Initialize submodules
git submodule update --init --recursive

# Build an image
cd halos-pi-gen
./run docker-build "HaLOS-Marine-HALPI2"
```

## Architecture Layers

1. **Base OS**: Debian-based Raspberry Pi OS (arm64, trixie) built with pi-gen
2. **Web Management** (all): Cockpit (9090) + Runtipi (80/443)
3. **Hardware** (HALPI2 only): HALPI2 drivers, CAN, RS-485, I2C
4. **Marine** (marine variants): Signal K (3000) → InfluxDB (8086) → Grafana (3001)

## Submodule Workflow

```bash
# Update specific submodule
cd halos-pi-gen
git pull origin main
cd ..
git add halos-pi-gen
git commit -m "Update halos-pi-gen"
```
