# HaLOS Cockpit Development - Coordination Document

**Status**: Active
**Last Updated**: 2025-11-11

## Purpose

This document coordinates the multi-component HaLOS Cockpit development effort. It provides high-level architecture decisions, phase tracking, and pointers to detailed component designs.

**For implementation details**, see component-specific DESIGN.md files linked below.

## Vision: Browser-Based, Unified Administration

**Goal**: Make Cockpit the single, unified admin interface for HaLOS, eliminating the need for separate tools like Runtipi.

**Philosophy**: All system management—packages, containers, network, configuration—should be accessible through one cohesive browser-based interface using standard Debian tooling (APT, systemd, NetworkManager).

**Key Principles**:
- **No desktop or command line required**: Manage from any device with a web browser
- **Standard Debian tools**: Built on APT, systemd, NetworkManager (not custom solutions)
- **Upstream friendly**: Designed for potential inclusion in Debian/Ubuntu
- **Extensible architecture**: Multiple stores can coexist (marine, dev, home automation)

## Current State (November 2025)

**Complete**:
- ✅ `cockpit-apt` module v0.1.0 - Full APT package manager for Cockpit
- ✅ Design documentation extracted to component repos
- ✅ GitHub project tracking set up ([Project #1](https://github.com/orgs/hatlabs/projects/1))

**In Progress** (Phase 1):
- 🔄 Container packaging tooling
- 🔄 Marine container store
- 🔄 cockpit-apt store filtering UI

**Planned**:
- Phase 2: Container configuration UI
- Phase 3: Expanded app catalog, WiFi configuration
- Phase 4: Upstream contributions

## Architecture Overview

### Core Decisions

**Container Apps as Debian Packages**:
- Each app is a `.deb` package (e.g., `signalk-server-container`)
- systemd manages lifecycle (not Docker orchestration)
- Standard paths: `/var/lib/container-apps/`, `/etc/container-apps/`
- Naming convention: `<upstream-name>-container`

**Store as Extensible Architecture**:
- Store definition packages install configuration to `/etc/container-apps/stores/`
- Debian tags (debtags) for categorization: `role::container-app, field::marine`
- Multiple stores can coexist (marine, dev, home automation)
- Without store packages, cockpit-apt remains standard APT manager

**Package Generation Workflow**:
- App definitions in dedicated repos (e.g., `halos-marine-containers`)
- `container-packaging-tools` generates .deb from definitions
- CI/CD builds and publishes to apt.hatlabs.fi

### Repository Structure

```
halos-distro/                      # Workspace coordinator (this repo)
├── docs/                          # Cross-component design docs
├── halos-marine-containers/       # Marine apps + store definition
│   └── docs/DESIGN.md
├── container-packaging-tools/     # Package generation tooling
│   └── docs/DESIGN.md
└── cockpit-apt/                   # APT manager with store filtering
    └── docs/CONTAINER_STORE_DESIGN.md
```

**Design Principle**: Purely directed graph - sub-repos cannot reference workspace or siblings.

## Component Documentation

### Phase 1 Components

1. **halos-marine-containers** - Marine app definitions and store
   - [DESIGN.md](halos-marine-containers/docs/DESIGN.md) - App/store format, build process
   - [Issue #14](https://github.com/hatlabs/halos-distro/issues/14) - Implementation tracking

2. **container-packaging-tools** - Package generation tooling
   - [DESIGN.md](container-packaging-tools/docs/DESIGN.md) - Tool architecture, templates
   - [Issue #15](https://github.com/hatlabs/halos-distro/issues/15) - Implementation tracking

3. **cockpit-apt** - Store filtering and repository management
   - [CONTAINER_STORE_DESIGN.md](cockpit-apt/docs/CONTAINER_STORE_DESIGN.md) - UI/backend design
   - [ADR-001-container-store.md](cockpit-apt/docs/ADR-001-container-store.md) - Architecture decision
   - [Issue #13](https://github.com/hatlabs/halos-distro/issues/13) - Implementation tracking

### Phase 2+ Components (Concept)

4. **cockpit-container-config** - Container app configuration UI
   - [docs/CONTAINER_CONFIG_DESIGN.md](docs/CONTAINER_CONFIG_DESIGN.md) - High-level concept (Phase 2)
   - Implementation: After Phase 1 complete

5. **WiFi configuration** - Enhanced network management
   - [docs/WIFI_CONFIG_DESIGN.md](docs/WIFI_CONFIG_DESIGN.md) - High-level concept (Phase 3)
   - Implementation: After Phase 2 complete

6. **homarr-container** - Dashboard landing page
   - [docs/HOMARR_INTEGRATION_DESIGN.md](docs/HOMARR_INTEGRATION_DESIGN.md) - Integration design (Phase 3.5)
   - Implementation: After Phase 3 complete

7. **halos-traefik-container** - Reverse proxy for clean URLs
   - [docs/TRAEFIK_INTEGRATION_DESIGN.md](docs/TRAEFIK_INTEGRATION_DESIGN.md) - Integration design (Phase 4)
   - Implementation: After Phase 3.5 complete

## Implementation Phases

### Phase 1: Container Store Foundation ⏳

**Goal**: Establish container app packaging and store infrastructure

**Status**: [In Progress - Track on GitHub](https://github.com/hatlabs/halos-distro/issues/12)

**Deliverables**:
- Tool to generate container packages from app definitions
- 3-5 working marine container apps
- Store filter working in cockpit-apt UI
- Users can install/remove container apps via cockpit-apt

**Success Criteria**:
- User installs `cockpit-apt` and `marine-container-store`
- Sees marine apps in filtered view
- Installs Signal K or OpenCPN successfully
- systemd service starts, container runs

**Timeline**: 2025 Q4

### Phase 2: Container Configuration 📅

**Goal**: Web UI for configuring installed container apps

**Status**: Planning (after Phase 1)

**Key Features**:
- List view of installed apps with status
- Configuration editor (web-based)
- Service control (start/stop/restart)
- Log viewer (journalctl integration)

**Timeline**: 2026 Q1

### Phase 3: Expansion & Polish 🔮

**Goal**: More apps, additional stores, WiFi configuration

**Status**: Concept (after Phase 2)

**Key Features**:
- 20+ marine apps
- Additional store categories (development, home automation)
- WiFi configuration in Cockpit
- Converter tools (import from other app stores)

**Timeline**: 2026 Q2+

### Phase 3.5: Dashboard Integration 🎨

**Goal**: Unified landing page with Homarr dashboard

**Status**: Design Complete (after Phase 3)

**Components**:
- homarr-container - Dashboard application
- homarr-container-adapter - Auto-discovery service (Rust)

**Key Features**:
- Auto-discovery of installed container apps via Docker labels
- Default landing page at `http://halos.local/`
- User-customizable dashboard (add/remove/reorder apps)
- Quick access to Cockpit and all installed apps
- Pre-installed in HaLOS images

**Deliverables**:
- Homarr packaged as container app
- Rust adapter for periodic container scanning
- Docker label generation in container-packaging-tools
- Integration with cockpit-container-config (optional)

**Documentation**:
- [docs/HOMARR_INTEGRATION_DESIGN.md](docs/HOMARR_INTEGRATION_DESIGN.md) - Integration design

**Timeline**: 2026 Q3

### Phase 4: Reverse Proxy Integration 🔀

**Goal**: Clean URLs with Traefik reverse proxy

**Status**: Design Complete (after Phase 3.5)

**Components**:
- halos-traefik-container - Reverse proxy with auto-configuration

**Key Features**:
- Clean URLs instead of random ports (e.g., `halos.local/signalk`)
- Optional integration (direct port access as fallback)
- User choice per app (Traefik, direct port, or both)
- HTTPS with self-signed or Let's Encrypt certificates
- Traefik labels in docker-compose.yml (Runtipi style)
- Pre-installed in HaLOS images

**Deliverables**:
- Traefik packaged as container app
- Traefik label generation in container-packaging-tools
- Auto-detection of domain for Let's Encrypt
- Integration with Homarr (dashboard shows proxy URLs)

**Documentation**:
- [docs/TRAEFIK_INTEGRATION_DESIGN.md](docs/TRAEFIK_INTEGRATION_DESIGN.md) - Integration design

**Timeline**: 2026 Q4

**Open Questions** (TBD during implementation):
- URL scheme: Path-based vs subdomain-based?
- Traefik as required dependency?
- Default HTTPS always or HTTP by default?

### Phase 5: Upstream & Community 🌐

**Goal**: Upstream contributions and community maintenance

**Status**: Long-term vision (after Phase 4)

**Key Features**:
- Submit cockpit-apt to Debian/Ubuntu
- Public documentation for creating custom stores
- Community contribution workflows
- Quality assurance and security review processes

**Timeline**: 2027+

## Progress Tracking

### GitHub Resources

- **Project Board**: [HaLOS Development](https://github.com/orgs/hatlabs/projects/1)
- **Milestones**: [View all milestones](https://github.com/hatlabs/halos-distro/milestones)
- **Issues**: [View open issues](https://github.com/hatlabs/halos-distro/issues)

### Labels

- **Component**: `component:apt`, `component:containers`, `component:tooling`, `component:config`
- **Phase**: `phase:1`, `phase:2`, `phase:3`
- **Priority**: `priority:high`, `priority:low`
- **Type**: `type:design`, `type:implementation`, `type:documentation`

### Current Sprint (Phase 1)

**Active Issues**:
- [#13](https://github.com/hatlabs/halos-distro/issues/13) - cockpit-apt store filtering
- [#14](https://github.com/hatlabs/halos-distro/issues/14) - halos-marine-containers
- [#15](https://github.com/hatlabs/halos-distro/issues/15) - container-packaging-tools

**Recommended Implementation Order**:
1. **container-packaging-tools** - Build the tooling first
2. **halos-marine-containers** - Create store and 3-5 apps
3. **cockpit-apt** - Add UI filtering features

## Next Steps

### Immediate (This Week)

- [x] Extract design docs to component repos
- [x] Set up GitHub project and tracking
- [x] Update README with roadmap

### Phase 1 Kickoff

Per [PROJECT_PLANNING_GUIDE.md](PROJECT_PLANNING_GUIDE.md), each component should:

1. Review extracted DESIGN.md
2. Create SPEC.md (technical specification)
3. Create ARCHITECTURE.md (system architecture)
4. Create TASKS.md (implementation breakdown)
5. Review and refine
6. Create detailed GitHub issues from tasks
7. Begin implementation

**Start with**: container-packaging-tools (enables everything else)

## Reference Documents

### Internal Documentation

- [PROJECT_PLANNING_GUIDE.md](PROJECT_PLANNING_GUIDE.md) - Development workflow for projects
- [README.md](README.md) - User-facing documentation and roadmap

### Component Design Docs

- [halos-marine-containers/docs/DESIGN.md](halos-marine-containers/docs/DESIGN.md)
- [container-packaging-tools/docs/DESIGN.md](container-packaging-tools/docs/DESIGN.md)
- [cockpit-apt/docs/CONTAINER_STORE_DESIGN.md](cockpit-apt/docs/CONTAINER_STORE_DESIGN.md)
- [docs/CONTAINER_CONFIG_DESIGN.md](docs/CONTAINER_CONFIG_DESIGN.md)
- [docs/WIFI_CONFIG_DESIGN.md](docs/WIFI_CONFIG_DESIGN.md)

### External References

- [Debtags Wiki](https://wiki.debian.org/Debtags)
- [Debian Policy Manual](https://www.debian.org/doc/debian-policy/)
- [Cockpit Project](https://cockpit-project.org/)
- [Docker Compose Specification](https://docs.docker.com/compose/compose-file/)

## Change Log

- **2025-11-11**: Simplified to coordination level, extracted details to component designs
- **2025-11-10**: Created initial META-PLANNING.md with full architecture details
- **2025-11-09**: Resolved architecture decisions (paths, tags, packages, workflow)
