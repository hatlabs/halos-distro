# HaLOS Workspace

**Document Purpose**: Instructions for AI assistants working with the HaLOS workspace. Defines repository structure, architecture layers, and development workflows.

Halos (Hat Labs Operating System) - custom Raspberry Pi OS distribution with web management.

This workspace manages multiple independent repositories for convenience. Each repository must work independently and can have no directory-level cross-dependencies.

## Git Workflow Policy

**MANDATORY**: PRs must ALWAYS have all checks passing before merging. No matter what.

## Structure

```
halos-distro/
├── halos-pi-gen/                  # Image builder (submodule)
├── runtipi-marine-app-store/      # Marine app store (submodule)
├── runtipi-docker-service/        # Runtipi Debian package
├── apt.hatlabs.fi/                # Custom APT repo (submodule)
├── cockpit-apt/                   # Cockpit APT package manager (Phase 1)
├── container-packaging-tools/     # Container package generator (Phase 1)
└── halos-marine-containers/       # Marine app definitions + store (Phase 1)
```

**Each repository has its own AGENTS.md** - read the appropriate one for detailed context.

### Phase 1 Components (Cockpit-based Management)

These three repositories implement the new Cockpit-based container management system:

- **cockpit-apt**: APT package manager with container store filtering
- **container-packaging-tools**: Generates .deb packages from container definitions
- **halos-marine-containers**: Marine app definitions + store package

## Quick Start

```bash
# Clone all component repositories
./run repos:clone

# Update all repositories
./run repos:pull-all-main

# Build an image
cd halos-pi-gen
./run docker-build "Halos-Marine-HALPI2"
```

## Architecture Layers

1. **Base OS**: Debian-based Raspberry Pi OS (arm64, trixie) built with pi-gen
2. **Web Management** (all): Cockpit (9090) + Runtipi (80/443)
3. **Hardware** (HALPI2 only): HALPI2 drivers, CAN, RS-485, I2C
4. **Marine** (marine variants): Signal K (3000) → InfluxDB (8086) → Grafana (3001)

## Repository Management

```bash
# Update all repositories to latest main/halos branches
./run repos:pull-all-main

# Check status of all repositories
./run repos:status

# Work in a specific repository
cd halos-pi-gen
git pull origin halos
# make changes, commit, push
cd ..
```

Each repository is managed independently. The halos-distro workspace tracks only shared documentation and convenience scripts.
