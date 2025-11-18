# Halos Raspberry Pi OS Distribution

Halos (Hat Labs Operating System) is a custom Raspberry Pi OS distribution designed to help end-users run and maintain both Hat Labs and generic Raspberry Pi hardware with ease. Built on Raspberry Pi OS Lite (arm64, Trixie), Halos adds web-based management tools, hardware monitoring, and containerized services.

> **⚠️ IMPORTANT: Work in Progress**
>
> Halos is under active development and should be considered **beta software**. Major components, architectures, and features are subject to change without notice. Use in production environments at your own risk. Feedback and bug reports are welcome!

## Vision: Browser-Based, Unified Administration

Halos is evolving toward a **unified, browser-based administration experience** through Cockpit. The goal is to consolidate all system management—packages, containers, network configuration—into a single, cohesive web interface using standard Debian tools (APT, systemd, NetworkManager).

### Why Browser-Based?

**No Desktop or Command Line Required**: Manage your entire system from any device with a web browser:
- Configure from your laptop, tablet, or phone
- No need to connect a monitor and keyboard to your Raspberry Pi
- Access your system remotely over your local network
- Works on any operating system (Windows, Mac, Linux, mobile)

**But Still Flexible**: Desktop environment and SSH access remain available for those who prefer them.

### What This Means

**Current State**: Halos uses two separate web interfaces—Cockpit for system administration and Runtipi for container management. This works but requires learning and navigating two different tools.

**Future State**: All management through Cockpit, with specialized modules for different tasks:
- **cockpit-apt**: Full-featured APT package manager (already available)
- **Container Store**: Browse and install containerized apps as native Debian packages
- **Container Config**: Separate Cockpit module to configure installed container apps
- **Enhanced Network**: WiFi configuration and network management improvements

### Key Benefits

- **Browser-Based**: Manage everything from any device on your network—no desktop or terminal needed
- **Single Interface**: One place for all administration tasks
- **Standard Tools**: Built on APT, systemd, and other proven Debian technologies
- **Native Packages**: Container apps as .deb packages, managed like any other software
- **Extensible Stores**: Multiple "stores" (marine apps, development tools, home automation) that filter and present relevant packages
- **Vanilla Compatible**: Works on standard Raspberry Pi OS with just `apt.hatlabs.fi` repo added
- **Upstream Friendly**: cockpit-apt designed to be attractive for inclusion in standard Debian/Ubuntu distributions

### Timeline

This transition is happening in phases:
1. **Phase 1** (In Progress): Container store foundation—tooling, packaging, and cockpit-apt enhancements
2. **Phase 2** (Planned): Container configuration UI for managing installed apps
3. **Phase 3** (Future): Expanded app catalog, WiFi configuration, converters for existing app stores

For detailed planning, see [META-PLANNING.md](META-PLANNING.md).

## What is Halos?

Halos transforms your Raspberry Pi into a managed server platform with:

- **Web-based system administration** via Cockpit (port 9090)
- **Docker container management** via Runtipi (port 80/443) with app store
- **Optional marine software stack** (Signal K, InfluxDB, Grafana) for boats
- **HALPI2 hardware support** for CAN bus, RS-485, I2C interfaces and power management

**Use cases:**
- Marine electronics server for boats (NMEA 2000, instrument displays)
- Home automation and IoT gateway
- Self-hosted web applications and services
- Raspberry Pi development platform with easy container deployment

## Choosing an Image Variant

Halos comes in multiple variants to suit different hardware and use cases:

### Hardware Platform
- **HALPI2 variants**: Include drivers for Hat Labs HALPI2 hardware (CAN, RS-485, I2C)
- **RPI variants**: For generic Raspberry Pi 4/5 (no HALPI2-specific drivers)

### Desktop Environment
- **Headless** (recommended): No desktop GUI, access via web interfaces only
- **Desktop**: Includes XFCE desktop environment

### Software Stack
- **Standard**: Cockpit + Runtipi web management
- **Marine**: Adds a Marine App Store for Runtipi with a number of containerized marine applications

### Available Images

| Image Name | Hardware | Desktop | Marine Stack |
|------------|----------|---------|--------------|
| `Halos-HALPI2` | HALPI2 | No | No |
| `Halos-Desktop-HALPI2` | HALPI2 | Yes | No |
| `Halos-Marine-HALPI2` | HALPI2 | No | Yes |
| `Halos-Desktop-Marine-HALPI2` | HALPI2 | Yes | Yes |
| `Halos-RPI` | Generic RPi | No | No |
| `Halos-Desktop-RPI` | Generic RPi | Yes | No |
| `Halos-Marine-RPI` | Generic RPi | No | Yes |
| `Halos-Desktop-Marine-RPI` | Generic RPi | Yes | Yes |

**Stock Raspberry Pi OS variants** (with HALPI2 drivers but no Halos web stack):
- `Raspios-lite-HALPI2`: Headless Raspberry Pi OS with HALPI2 drivers
- `Raspios-HALPI2`: Desktop Raspberry Pi OS with HALPI2 drivers

## Getting Started

### 1. Download the Image

Download your chosen image from the [Halos releases page](https://github.com/hatlabs/halos-pi-gen/releases).

### 2. Flash to SD Card or SSD

For HALPI2, follow the [Documentation flashing instructions](https://docs.hatlabs.fi/halpi2/user-guide/software.html#flashing-an-operating-system-image-to-ssd) to flash to an SSD.

For generic Raspberry Pi, follow these steps:

1. Download [Raspberry Pi Imager](https://www.raspberrypi.org/software/)
2. Insert your SD card or connect SSD via USB adapter
3. Open Raspberry Pi Imager
4. Select "Use custom" and choose your downloaded Halos image
5. Select your target drive
6. Click "Write" (do not apply OS customization settings)

### 3. First Boot

1. Insert the card/drive into your Raspberry Pi and power on
2. Wait 1-2 minutes for first boot initialization
3. Use halos.local to access the web interface

**Default credentials:**
- Username: `pi`
- Password: `raspberry`

**Important:** Change the default password immediately after first login!

## Using Halos

### Web Management Interfaces

Access these services from any browser on your network:

- **Cockpit** ([`https://halos.local:9090/`](https://halos.local:9090/)): System administration
  - Terminal access
  - System resource monitoring
  - Service management
  - Software updates
  - User management

- **Runtipi** ([`http://halos.local`](http://halos.local)): Container management
  - Browse and install apps from the app store
  - Manage running containers
  - Configure application settings

### Marine Stack

Apps need to be installed first from the Runtipi Marine App Store.

- **Signal K** ([`http://halos.local:3000`](http://halos.local:3000)): Marine data server and API
- **InfluxDB** ([`http://halos.local:8086`](http://halos.local:8086)): Time-series database for marine data
- **Grafana** ([`http://halos.local:3001`](http://halos.local:3001)): Data visualization and dashboards
- **AvNav** ([`http://halos.local:3011`](http://halos.local:3011)): Marine navigation software
- **OpenCPN** ([`http://halos.local:3021`](http://halos.local:3021)): Chartplotter and navigation software

### Terminal Access

- Via Cockpit: Click "Terminal" in the left sidebar
- Via SSH: `ssh pi@halos.local` (if enabled in Cockpit)

## Known Limitations

These features are planned but not yet implemented:

- **Pre-installed Runtipi apps**: Waiting for upstream Runtipi to add support for pre-installing apps in the image. Currently, you must manually install apps after first boot.

- **WiFi configuration in Cockpit**: Cockpit's network module doesn't yet support WiFi setup.
  - **Workaround**: Access the Cockpit terminal and run `sudo nmtui` to configure WiFi using NetworkManager's text interface.

- **Unified dashboard**: A user-friendly landing page showing system status and quick links to installed apps is planned but not yet available. Currently, you need to bookmark or remember the different service ports.

## Development Roadmap

Halos is transitioning to a unified Cockpit-based administration experience. This roadmap outlines the planned phases and current status.

### Phase 1: Container Store Foundation (In Progress)

**Goal**: Establish container app packaging and store infrastructure

**Status**: [In Progress - Track on GitHub](https://github.com/hatlabs/halos-distro/issues/12)

**Components**:
- [container-packaging-tools](https://github.com/hatlabs/container-packaging-tools) - Tool for generating .deb packages from container app definitions ([#15](https://github.com/hatlabs/halos-distro/issues/15))
- [halos-marine-containers](https://github.com/hatlabs/halos-marine-containers) - Marine app definitions and store configuration ([#14](https://github.com/hatlabs/halos-distro/issues/14))
- [cockpit-apt](https://github.com/hatlabs/cockpit-apt) - Store filtering and repository management UI ([#13](https://github.com/hatlabs/halos-distro/issues/13))

**Deliverables**:
- Tool to generate container packages from simple app definitions
- 3-5 working marine container apps as proof of concept
- Store filter working in cockpit-apt UI
- Users can install/remove container apps via cockpit-apt

**Key Features**:
- Container apps packaged as standard Debian .deb files
- systemd manages container lifecycle (no Docker orchestration)
- "Marine Apps" store filter in cockpit-apt
- Repository filtering for vanilla Raspberry Pi OS users too
- Each app has metadata, configuration schema, and systemd service

**Success Criteria**:
- User adds apt.hatlabs.fi repository
- Installs cockpit-apt and marine-container-store packages
- Sees marine apps in filtered store view
- Installs Signal K or OpenCPN successfully
- systemd service starts automatically, container runs

**Documentation**:
- [META-PLANNING.md](META-PLANNING.md) - Overall project planning
- [docs/CATEGORIZATION_STRATEGY.md](docs/CATEGORIZATION_STRATEGY.md) - Tag-based categorization approach
- [halos-marine-containers/docs/DESIGN.md](halos-marine-containers/docs/DESIGN.md) - App and store definition format
- [container-packaging-tools/docs/DESIGN.md](container-packaging-tools/docs/DESIGN.md) - Packaging tool design
- [cockpit-apt/docs/CONTAINER_STORE_DESIGN.md](cockpit-apt/docs/CONTAINER_STORE_DESIGN.md) - UI implementation

### Phase 2: Container Configuration (Planned)

**Goal**: Web UI for configuring installed container apps

**Status**: Planning (starts after Phase 1 complete)

**Components**:
- cockpit-container-config - New Cockpit module for container app management

**Deliverables**:
- List view of installed container apps with status
- Configuration editor (generated from config.yml or simple text editor)
- Service control (start/stop/restart via systemd)
- Real-time log viewer (journalctl integration)
- Quick links to application web UIs

**Key Features**:
- Separate Cockpit module (appears in left sidebar like Services, Logs, Terminal)
- Configure apps via web interface (no file editing required)
- Visual service management with status indicators
- Troubleshooting with integrated log viewer
- QR codes for sharing app URLs

**Documentation**:
- [docs/CONTAINER_CONFIG_DESIGN.md](docs/CONTAINER_CONFIG_DESIGN.md) - High-level design concept

### Phase 3: Expansion & Polish (Future)

**Goal**: More apps, converters, additional stores, and WiFi configuration

**Status**: Concept (after Phase 2)

**Deliverables**:
- 20+ marine container apps available
- Converter tools (import from CasaOS, Runtipi, other app stores)
- Additional store categories (development tools, home automation)
- Enhanced WiFi configuration in Cockpit (access point, saved networks, QR codes)

**Key Features**:
- Comprehensive marine app catalog
- Automated app store imports
- Multi-store support (marine, dev, home automation)
- WiFi access point setup with QR code generation
- Marina WiFi connection management
- Bridge mode (share marina WiFi with boat network)

**Documentation**:
- [docs/WIFI_CONFIG_DESIGN.md](docs/WIFI_CONFIG_DESIGN.md) - WiFi configuration concept

### Phase 3.5: Dashboard Integration (Future)

**Goal**: Unified landing page with Homarr dashboard

**Status**: Design Complete (after Phase 3)

**Deliverables**:
- Homarr packaged as `homarr-container`
- Rust adapter service (`homarr-container-adapter`) for auto-discovery
- Auto-discovery of installed apps via Docker labels
- Pre-installed in HaLOS images

**Key Features**:
- Default landing page at `http://halos.local/`
- Auto-discovery of newly installed container apps
- User-customizable dashboard (add/remove/reorder apps)
- Quick access to Cockpit and all installed apps
- System status widgets (CPU, memory, disk)
- Mobile-friendly responsive design

**Documentation**:
- [docs/HOMARR_INTEGRATION_DESIGN.md](docs/HOMARR_INTEGRATION_DESIGN.md) - Homarr integration design

### Phase 4: Reverse Proxy Integration (Future)

**Goal**: Clean URLs with Traefik reverse proxy

**Status**: Design Complete (after Phase 3.5)

**Deliverables**:
- Traefik packaged as `halos-traefik-container`
- Auto-configuration via Docker labels (Runtipi style)
- HTTPS with self-signed or Let's Encrypt certificates
- Pre-installed in HaLOS images

**Key Features**:
- Clean URLs instead of random ports (e.g., `halos.local/signalk` vs `:3000`)
- Optional integration - apps work with direct ports as fallback
- User choice per app (Traefik route, direct port, or both)
- Automatic HTTPS certificate management
- Integration with Homarr (dashboard shows proxy URLs)
- Traefik labels auto-generated in app packages

**Documentation**:
- [docs/TRAEFIK_INTEGRATION_DESIGN.md](docs/TRAEFIK_INTEGRATION_DESIGN.md) - Traefik integration design

**Open Questions** (TBD during implementation):
- URL scheme: Path-based (`halos.local/app`) vs subdomain-based (`app.halos.local`)?
- Default HTTPS always or HTTP by default?

### Phase 5: Upstream & Community (Long-term)

**Goal**: Upstream contributions and community maintenance

**Status**: Long-term vision

**Deliverables**:
- Submit cockpit-apt to Debian/Ubuntu for inclusion
- Public documentation for creating custom stores
- Contribution guidelines for community apps
- Community moderation workflows

**Key Features**:
- cockpit-apt available in standard Debian/Ubuntu repositories
- Easy for other projects to create their own app stores
- Community-maintained app catalog
- Quality assurance and security review processes

### Timeline Overview

```
2025 Q4: Phase 1 - Container Store Foundation
├─ container-packaging-tools MVP
├─ halos-marine-containers (3-5 apps)
└─ cockpit-apt store features

2026 Q1: Phase 2 - Container Configuration
└─ cockpit-container-config module

2026 Q2: Phase 3 - Expansion & Polish
├─ Expanded app catalog (20+ apps)
├─ Additional stores
└─ WiFi configuration

2026 Q3: Phase 3.5 - Dashboard Integration
├─ Homarr dashboard landing page
├─ Auto-discovery adapter (Rust)
└─ Default page at http://halos.local/

2026 Q4: Phase 4 - Reverse Proxy
├─ Traefik integration
├─ Clean URLs (no ports)
└─ HTTPS with Let's Encrypt

2027+: Phase 5 - Upstream & Community
└─ Debian/Ubuntu submission
```

**Note**: Timelines are estimates and subject to change based on development progress and community feedback.

### Get Involved

- **Follow Progress**: [GitHub Issues](https://github.com/hatlabs/halos-distro/issues) and [Milestones](https://github.com/hatlabs/halos-distro/milestones)
- **Provide Feedback**: Open issues with feature requests or bug reports
- **Contribute**: See individual component repositories for contribution guidelines
- **Test**: Try Phase 1 features as they become available and report issues

### Runtipi Replacement

Runtipi will be replaced (not migrated) once equivalent functionality is available:
- **Phase 1-2**: Container apps via cockpit-apt + configuration UI
- **Phase 3.5**: Homarr replaces Runtipi dashboard
- **Phase 4**: Traefik replaces Runtipi's reverse proxy
- Runtipi removed from HaLOS images after Phase 4 complete

Users installing HaLOS during the transition will have Runtipi available until the new stack is ready.

## Development

This repository acts as a workspace manager for all the components that make up Halos.

### Repository Layout

- **halos-pi-gen/** - Custom Raspberry Pi image builder based on pi-gen
- **runtipi-marine-app-store/** - Custom marine app store for Runtipi
- **runtipi-docker-service/** - Runtipi Debian package
- **apt.hatlabs.fi/** - Custom APT repository for Halos packages
- **avnav-docker/** - AVNav marine navigation software
- **opencpn-docker/** - OpenCPN marine chartplotter

Each repository is independently managed. The `./run` script provides convenience commands for cloning and updating all repositories at once.

### Development Quick Start

```bash
# Clone all component repositories
./run repos:clone

# Update all repositories to latest
./run repos:pull-all-main

# Check status of all repositories
./run repos:status

# Build an image (requires Docker and act)
cd halos-pi-gen
./run docker:build "Halos-Marine-HALPI2"
```

Each repository has its own `CLAUDE.md` with detailed development documentation.

## Contributing

Contributions are welcome! Each component repository accepts pull requests independently. See the individual repository documentation for specific guidelines.

## License

See individual component repositories for license information.
