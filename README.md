# Halos Raspberry Pi OS Distribution

Halos (Hat Labs Operating System) is a custom Raspberry Pi OS distribution designed to help end-users run and maintain both Hat Labs and generic Raspberry Pi hardware with ease. Built on Raspberry Pi OS Lite (arm64, Trixie), Halos adds web-based management tools, hardware monitoring, and containerized services.

> **⚠️ IMPORTANT: Work in Progress**
>
> Halos is under active development and should be considered **beta software**. Major components, architectures, and features are subject to change without notice. Use in production environments at your own risk. Feedback and bug reports are welcome!

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
