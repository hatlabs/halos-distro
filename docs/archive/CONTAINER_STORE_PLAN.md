# Container Store Architecture & Implementation Plan

**Version:** 1.0
**Date:** 2025-11-06
**Status:** Draft - Architecture Proposal

## Executive Summary

### Vision

Replace Runtipi with a native Debian package-based container application delivery system that integrates seamlessly with Cockpit, leveraging standard Linux tools (apt, systemd, journalctl) instead of custom orchestration layers.

### Goals

1. **Simplicity**: Use standard Debian packaging instead of custom container orchestration
2. **Integration**: Leverage Cockpit Package Manager for discovery and installation
3. **Lifecycle Management**: Use systemd for container lifecycle, logs, and monitoring
4. **Extensibility**: Support multiple app sources (CasaOS, Runtipi, custom marine apps)
5. **User Experience**: Provide web-based configuration and management interfaces

### Key Benefits

- **Mature Tooling**: apt, dpkg, systemd are proven, well-understood technologies
- **Unified Management**: All system packages managed through single interface
- **Better Logging**: journalctl provides unified log access
- **Automatic Updates**: Standard apt upgrade process
- **Reduced Complexity**: Eliminate Runtipi's 4-container orchestration stack
- **Standard Practices**: Follows Debian packaging best practices

## Architecture Overview

### Component Layers

```
┌─────────────────────────────────────────────────────────────┐
│                         User Interface                      │
│  ┌──────────────────────┐      ┌──────────────────────────┐ │
│  │ Cockpit Package Mgr  │      │ Cockpit Container Mgr    │ │
│  │ (Install/Remove)     │─────▶│ (Configure/Monitor)      │ │
│  └──────────────────────┘      └──────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                          │                        │
┌─────────────────────────────────────────────────────────────┐
│                      System Layer                           │
│  ┌──────────────┐    ┌──────────────┐    ┌───────────────┐  │
│  │  PackageKit  │    │   systemd    │    │  journalctl   │  │
│  └──────────────┘    └──────────────┘    └───────────────┘  │
└─────────────────────────────────────────────────────────────┘
                          │                        │
┌─────────────────────────────────────────────────────────────┐
│                     Package Layer                           │
│  ┌──────────────┐    ┌──────────────┐    ┌───────────────┐  │
│  │     APT      │    │  .deb files  │    │  Docker/OCI   │  │
│  │   Repository │    │              │    │  containers   │  │
│  └──────────────┘    └──────────────┘    └───────────────┘  │
└─────────────────────────────────────────────────────────────┘
                          │
┌─────────────────────────────────────────────────────────────┐
│                    Build Pipeline                           │
│  ┌────────────────┐  ┌──────────────┐  ┌─────────────────┐  │
│  │ Container Store│  │  Converters  │  │   Package Gen   │  │
│  │   Repository   │─▶│ (CasaOS, etc)│─▶│   & Publish     │  │
│  └────────────────┘  └──────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Information Flow

**Installation Flow:**
1. User browses apps in Cockpit Package Manager
2. CPM queries PackageKit for available packages
3. User selects app → PackageKit installs via apt
4. Package installs container definition, systemd service, metadata
5. Postinst generates default config, starts service
6. CPM shows "Configure" button → opens Container Manager
7. User configures app via web form
8. Container Manager saves config, restarts service

**Update Flow:**
1. apt update pulls new package versions
2. User runs apt upgrade or uses CPM
3. Package upgrade preserves configuration
4. systemd restarts service with new container version

## Component Breakdown

### 1. Container Store Repository

**Purpose:** Central repository of container application definitions

**Structure:**
```
container-store/
├── apps/
│   ├── media/                    # Category-based organization
│   │   ├── jellyfin/
│   │   │   ├── app.yaml          # Application metadata
│   │   │   ├── docker-compose.yml
│   │   │   ├── icon.png
│   │   │   └── screenshots/
│   │   ├── plex/
│   │   └── navidrome/
│   ├── productivity/
│   │   ├── nextcloud/
│   │   └── home-assistant/
│   ├── development/
│   │   ├── gitea/
│   │   └── code-server/
│   ├── marine/                   # Marine-specific apps
│   │   ├── signalk/
│   │   ├── opencpn/
│   │   └── grafana-marine/
│   └── network/
├── converters/                   # Import tools
│   ├── casaos_converter.py
│   ├── runtipi_converter.py
│   ├── common.py                 # Shared utilities
│   └── README.md
├── build/                        # Package generation
│   ├── generate_packages.py
│   ├── debian_template/          # Debian packaging templates
│   │   ├── control.template
│   │   ├── rules.template
│   │   ├── postinst.template
│   │   └── systemd.service.template
│   └── publish.py                # Publish to APT repo
├── schemas/
│   ├── app-schema.json           # JSON schema for app.yaml
│   └── config-schema.json        # JSON schema for configuration
├── docs/
│   ├── CONTRIBUTING.md
│   ├── APP_FORMAT.md
│   └── CONVERTER_GUIDE.md
└── README.md
```

### 2. Application Metadata Format (app.yaml)

**Complete schema for application definition:**

```yaml
#──────────────────────────────────────────────────────────────
# Required Fields
#──────────────────────────────────────────────────────────────

id: signalk                       # Package suffix: container-signalk
name: Signal K Server             # Display name
version: 2.8.0                    # Application version
category: marine/navigation       # Debian section: container/marine

description: |
  Open source data hub for marine systems. Signal K acts as a central
  nervous system for your boat, collecting data from all sensors and
  instruments and making it available to navigation software, monitoring
  displays, and data loggers.

tagline: "Connect all your boat's data"  # Short one-liner

#──────────────────────────────────────────────────────────────
# Container Definition
#──────────────────────────────────────────────────────────────

# Option 1: Reference external compose file
compose_file: docker-compose.yml

# Option 2: Inline compose definition (for simple apps)
# containers:
#   - name: signalk
#     image: signalk/signalk-server:${SIGNALK_VERSION:-latest}
#     ports:
#       - "${SIGNALK_PORT:-3000}:3000"
#     volumes:
#       - "${DATA_DIR:-/var/lib/container-apps/signalk/data}:/data"
#     environment:
#       - PUID=${PUID:-1000}
#       - PGID=${PGID:-1000}

#──────────────────────────────────────────────────────────────
# UI Metadata
#──────────────────────────────────────────────────────────────

icon: icon.png                    # 256x256 PNG (installed to /usr/share/container-apps/)
screenshots:
  - screenshot1.png               # Optional screenshots
  - screenshot2.png

tags:
  - navigation                    # Searchable tags
  - nmea
  - data-hub
  - marine

#──────────────────────────────────────────────────────────────
# Web UI Configuration
#──────────────────────────────────────────────────────────────

web_ui:
  enabled: true                   # Does app have web interface?
  port: 3000                      # Port for web interface
  path: /                         # Path (e.g., /admin, /dashboard)
  protocol: http                  # http or https
  # Used to generate "Open Web UI" button in Container Manager

#──────────────────────────────────────────────────────────────
# System Requirements
#──────────────────────────────────────────────────────────────

requires:
  architecture: [amd64, arm64]    # Supported architectures
  memory_min: 512M                # Minimum RAM
  disk_min: 1G                    # Minimum disk space
  hardware: []                    # e.g., [gps, can, serial] for marine
  kernel_modules: []              # e.g., [can, can-raw] if needed

#──────────────────────────────────────────────────────────────
# Configuration Schema
#──────────────────────────────────────────────────────────────

configuration:
  # Environment variables that can be configured
  env_vars:
    - name: SIGNALK_PORT
      label: "Server Port"
      description: "Port for accessing the web interface"
      type: number
      default: 3000
      required: true
      validation:
        min: 1024
        max: 65535
      hint: "Choose an unused port (1024-65535)"

    - name: PUID
      label: "User ID"
      description: "User ID for file permissions"
      type: number
      default: 1000
      required: true
      hint: "Usually 1000 for first user"

    - name: PGID
      label: "Group ID"
      description: "Group ID for file permissions"
      type: number
      default: 1000
      required: true

    - name: ADMIN_PASSWORD
      label: "Admin Password"
      description: "Password for admin user (leave blank for default)"
      type: password
      required: false
      default: ""

    - name: TZ
      label: "Timezone"
      description: "Timezone for logs and scheduling"
      type: timezone
      default: "UTC"
      required: true

    - name: ENABLE_PLUGINS
      label: "Enable Plugins"
      description: "Allow installation of Signal K plugins"
      type: boolean
      default: true

    - name: LOG_LEVEL
      label: "Log Level"
      description: "Verbosity of application logs"
      type: select
      options:
        - value: debug
          label: Debug (verbose)
        - value: info
          label: Info (recommended)
        - value: warn
          label: Warning
        - value: error
          label: Error (minimal)
      default: info

    - name: FEATURE_FLAGS
      label: "Enabled Features"
      description: "Optional features to enable"
      type: multiselect
      options:
        - value: nmea2000
          label: NMEA 2000 Support
        - value: ais
          label: AIS Integration
        - value: charts
          label: Chart Provider
      default: []

  # Volume mount configurations
  volumes:
    - name: data
      label: "Data Directory"
      description: "Where Signal K stores configuration and logs"
      default: "/var/lib/container-apps/signalk/data"
      required: true
      type: path

  # Port mappings (configurable to avoid conflicts)
  ports:
    - container: 3000
      host: 3000
      label: "Web Interface"
      protocol: tcp
      configurable: true            # User can change host port

  # Configuration templates (quick presets)
  templates:
    - name: development
      label: "Development Setup"
      description: "Debug logging, all features enabled"
      env:
        LOG_LEVEL: debug
        ENABLE_PLUGINS: true
        FEATURE_FLAGS: [nmea2000, ais, charts]

    - name: production
      label: "Production (Recommended)"
      description: "Optimized for stability and performance"
      env:
        LOG_LEVEL: info
        ENABLE_PLUGINS: true
        FEATURE_FLAGS: [nmea2000, ais]

    - name: minimal
      label: "Minimal Setup"
      description: "Basic features only, lowest resource usage"
      env:
        LOG_LEVEL: warn
        ENABLE_PLUGINS: false
        FEATURE_FLAGS: []

#──────────────────────────────────────────────────────────────
# Dependencies
#──────────────────────────────────────────────────────────────

depends:
  packages:                         # APT package dependencies
    - docker.io | docker-ce
  containers:                       # Other container app dependencies
    - container-influxdb            # Optional: for data logging
  suggests:                         # Recommended companion apps
    - container-grafana
    - container-node-red

#──────────────────────────────────────────────────────────────
# Source Tracking (for converters)
#──────────────────────────────────────────────────────────────

source:
  type: runtipi                     # runtipi, casaos, custom, marine
  original_id: signalk
  upstream_url: https://github.com/runtipi/runtipi-appstore/tree/master/apps/signalk
  last_synced: 2025-11-01
  notes: "Converted from Runtipi app store"

#──────────────────────────────────────────────────────────────
# Documentation & Support
#──────────────────────────────────────────────────────────────

homepage: https://signalk.org
documentation: https://signalk.org/docs
support: https://signalk.org/community
source_code: https://github.com/SignalK/signalk-server

license: Apache-2.0

maintainer:
  name: Hat Labs
  email: support@hatlabs.fi
  url: https://hatlabs.fi

#──────────────────────────────────────────────────────────────
# Marine-Specific Metadata (optional, for marine apps)
#──────────────────────────────────────────────────────────────

marine:                             # Only for marine category apps
  features:
    hardware_integration: [nmea0183, nmea2000, i2c]
    data_sources: [gps, depth, wind, temperature]
    protocols: [signalk, nmea]
  integrations:                     # Works with these other apps
    - influxdb                      # For data logging
    - grafana                       # For visualization
    - node-red                      # For automation
  vessel_types: [sailboat, motorboat, yacht]
  use_cases: [navigation, monitoring, logging, autopilot]
```

### 3. Package Naming and Structure

**Naming Convention:**
- Package prefix: `container-`
- Format: `container-<app-id>`
- Examples: `container-signalk`, `container-jellyfin`, `container-nextcloud`

**Debian Section Mapping:**
- Category `media/streaming` → Section `container/media`
- Category `productivity/automation` → Section `container/productivity`
- Category `marine/navigation` → Section `container/marine`
- Category `development/tools` → Section `container/development`

**Installed File Structure:**
```
/opt/container-apps/signalk/
├── docker-compose.yml              # Container definition
└── .env.template                   # Default environment variables

/etc/container-apps/signalk/
├── config.yml                      # User-editable configuration
└── environment                     # Systemd environment file (generated)

/etc/systemd/system/
└── container-signalk.service       # Systemd service unit

/usr/share/container-apps/signalk/
├── metadata.json                   # Rich metadata (from app.yaml)
├── icon.png                        # Application icon
└── screenshots/                    # Screenshots
    ├── screenshot1.png
    └── screenshot2.png

/usr/share/doc/container-signalk/
├── README.md                       # Package documentation
├── changelog.gz                    # Debian changelog
└── copyright                       # License information
```

**Systemd Service Template:**
```ini
[Unit]
Description=%APP_NAME% (Container)
After=docker.service
Requires=docker.service
Documentation=%HOMEPAGE%

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/container-apps/%APP_ID%
EnvironmentFile=/etc/container-apps/%APP_ID%/environment
ExecStartPre=/usr/bin/docker compose pull --quiet
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
ExecReload=/usr/bin/docker compose restart
Restart=on-failure
RestartSec=10s
TimeoutStartSec=300
TimeoutStopSec=120

[Install]
WantedBy=multi-user.target
```

### 4. Debian Package Structure

**Control File Template:**
```
Package: container-signalk
Version: 2.8.0-1
Section: container/marine
Priority: optional
Architecture: all
Depends: docker.io (>= 20.10) | docker-ce (>= 20.10), docker-compose-plugin | docker-compose (>= 2.0)
Recommends: container-influxdb, container-grafana
Suggests: container-node-red
Maintainer: Hat Labs <support@hatlabs.fi>
Homepage: https://signalk.org
Description: Signal K Server - Marine data hub
 Open source data hub for marine systems. Signal K acts as a central
 nervous system for your boat, collecting data from all sensors and
 instruments and making it available to navigation software, monitoring
 displays, and data loggers.
 .
 This package provides a containerized installation of Signal K Server
 with web-based configuration and management through Cockpit.
Source-Type: runtipi
Source-ID: signalk
```

**Postinst Script (simplified):**
```bash
#!/bin/sh
set -e

APP_ID="signalk"
APP_NAME="Signal K Server"
INSTALL_DIR="/opt/container-apps/$APP_ID"
CONFIG_DIR="/etc/container-apps/$APP_ID"
DATA_DIR="/var/lib/container-apps/$APP_ID"

case "$1" in
    configure)
        # Create directories
        mkdir -p "$CONFIG_DIR"
        mkdir -p "$DATA_DIR"

        # Generate default configuration if not exists
        if [ ! -f "$CONFIG_DIR/config.yml" ]; then
            cat > "$CONFIG_DIR/config.yml" <<EOF
# Signal K Server Configuration
# Edit this file to customize your installation

port: 3000
enable_plugins: true
log_level: info
timezone: UTC

# Data directory (container internal path: /data)
data_dir: $DATA_DIR/data
EOF
        fi

        # Generate systemd environment file from config
        "$INSTALL_DIR/generate-env.sh" "$CONFIG_DIR/config.yml" > "$CONFIG_DIR/environment"

        # Set permissions
        chmod 644 "$CONFIG_DIR/config.yml"
        chmod 600 "$CONFIG_DIR/environment"

        # Reload systemd
        systemctl daemon-reload

        # Enable service (but don't start yet - user should configure first)
        systemctl enable container-signalk.service

        # Start service with defaults
        systemctl start container-signalk.service || true

        echo ""
        echo "======================================================================"
        echo "  $APP_NAME has been installed!"
        echo "======================================================================"
        echo ""
        echo "  Configuration: $CONFIG_DIR/config.yml"
        echo "  Data directory: $DATA_DIR"
        echo ""
        echo "  To configure: Open Cockpit → Container Manager → Signal K"
        echo "  Web UI: http://localhost:3000"
        echo ""
        echo "  Service: container-signalk.service"
        echo "  Status: systemctl status container-signalk"
        echo "  Logs: journalctl -u container-signalk -f"
        echo ""
        echo "======================================================================"
        ;;
esac

exit 0
```

## Cockpit Integration

### 1. Package Manager Extension (CPM)

**Design Principle:** CPM remains a pure PackageKit tool. Extensions are configuration-driven, not code changes.

**Configuration File: `/etc/cockpit/packagemanager/sources.d/container-store.json`**

```json
{
  "id": "container-store",
  "name": "Container Applications",
  "version": "1.0",
  "enabled": true,

  "detection": {
    "section_prefix": "container/",
    "metadata_base_path": "/usr/share/container-apps",
    "package_prefix": "container-"
  },

  "categories": {
    "media": {
      "name": "Media & Entertainment",
      "icon": "fa-film",
      "description": "Streaming, music, photos, and media management",
      "color": "#e74c3c"
    },
    "productivity": {
      "name": "Productivity",
      "icon": "fa-briefcase",
      "description": "Document management, calendar, tasks, collaboration",
      "color": "#3498db"
    },
    "marine": {
      "name": "Marine Navigation",
      "icon": "fa-anchor",
      "description": "Navigation, weather routing, and marine data systems",
      "color": "#1abc9c",
      "features": {
        "show_hardware_requirements": true,
        "show_vessel_types": true,
        "highlight_integration": true
      }
    },
    "development": {
      "name": "Development Tools",
      "icon": "fa-code",
      "description": "IDEs, version control, CI/CD, and developer utilities",
      "color": "#9b59b6"
    },
    "home": {
      "name": "Home Automation",
      "icon": "fa-home",
      "description": "Smart home control, automation, and IoT management",
      "color": "#f39c12"
    },
    "network": {
      "name": "Network & Security",
      "icon": "fa-network-wired",
      "description": "VPN, DNS, proxies, and network tools",
      "color": "#34495e"
    }
  },

  "ui": {
    "show_icons": true,
    "show_screenshots": true,
    "show_web_ui_links": true,
    "show_container_status": true,
    "show_source_badge": true,
    "group_by_category": true,
    "category_view_default": true,
    "featured_apps": [
      "container-signalk",
      "container-jellyfin",
      "container-nextcloud",
      "container-home-assistant"
    ]
  },

  "actions": {
    "post_install": {
      "show_configure_button": true,
      "configure_url": "/cockpit/@localhost/containermanager/configure?app={package_name}",
      "show_web_ui_button": true
    }
  }
}
```

**CPM UI Changes:**

The configuration file is read by CPM on startup. When CPM detects packages with `section_prefix: "container/"`:

1. **Category View**: Add "Container Apps" top-level navigation item
2. **Enhanced Listing**: Display icons, screenshots from metadata
3. **Source Badges**: Show origin (CasaOS, Runtipi, Marine)
4. **Post-Install Actions**: Show "Configure" button linking to Container Manager
5. **Rich Details**: Display hardware requirements, integrations for marine apps

**Implementation:** CPM loads `/etc/cockpit/packagemanager/sources.d/*.json` on init, extends UI accordingly.

### 2. Container Manager Module (New)

**Module Name:** `cockpit-containermanager`
**Repository:** New repository in HaLOS organization
**Tech Stack:** React + TypeScript + PatternFly (same as CPM)

**Features:**

**MVP (Phase 1):**
- List all installed `container-*` packages
- Show systemd service status (running, stopped, failed)
- Start/Stop/Restart buttons
- View logs (journalctl integration)
- Text editor for `/etc/container-apps/<app>/config.yml`
- Link to web UI (if app provides one)

**Enhanced (Phase 2):**
- Dynamic configuration forms (generated from metadata.json)
- Form validation (port ranges, paths, types)
- Configuration templates (dev, production, etc.)
- Save → generate environment file → restart service
- Port conflict detection
- Resource usage display (memory, CPU)

**Advanced (Phase 3):**
- Container health monitoring
- Backup/restore configuration
- Bulk operations (start all, stop all)
- Update notifications
- Dependency visualization (which apps depend on what)

**D-Bus Interface:**

Container Manager uses standard Cockpit APIs:
- `systemd1` D-Bus for service management
- `cockpit.js` for shell commands (docker ps, docker logs)
- File operations via Cockpit's file API

**UI Structure:**

```
Container Manager
├── Overview
│   ├── Summary (X apps installed, Y running)
│   ├── Resource usage (total CPU, RAM)
│   └── Recent logs
├── Applications
│   ├── All Applications (list view)
│   │   ├── [Icon] Signal K - Running [Configure] [Open Web UI] [Logs]
│   │   ├── [Icon] Jellyfin - Stopped [Configure] [Start] [Logs]
│   │   └── ...
│   └── By Category (grouped view)
│       ├── Marine (3 apps)
│       ├── Media (5 apps)
│       └── ...
└── Settings
    ├── Default data directory
    ├── Automatic updates
    └── Log retention
```

**Configuration Form Example (Dynamic):**

When user clicks "Configure" on Signal K:
1. Load `/usr/share/container-apps/signalk/metadata.json`
2. Parse `configuration.env_vars` schema
3. Load current values from `/etc/container-apps/signalk/config.yml`
4. Generate PatternFly form with appropriate field types
5. Validate on change (port ranges, required fields, etc.)
6. On save:
   - Validate all fields
   - Write to `config.yml`
   - Run `generate-env.sh` to create systemd environment file
   - Prompt: "Restart service to apply changes?"
   - If yes: `systemctl restart container-signalk`

### 3. Integration Flow

**Installation Flow:**
```
User in CPM → Browse Container Apps → Select Signal K → Install
                                                            ↓
PackageKit installs container-signalk package (via apt)
                                                            ↓
Postinst script runs:
  - Creates directories
  - Generates default config.yml
  - Creates systemd service
  - Starts service with defaults
                                                            ↓
CPM shows success notification with buttons:
  [Configure App] [Open Web UI] [View Logs]
                                                            ↓
User clicks [Configure App] → Opens Container Manager
                                                            ↓
Container Manager shows configuration form
                                                            ↓
User edits settings → Saves → Service restarts
                                                            ↓
User clicks [Open Web UI] → Opens http://localhost:3000
```

**Update Flow:**
```
apt update pulls new versions
                                ↓
CPM shows "Updates Available" with changelog
                                ↓
User clicks "Update" → PackageKit runs apt upgrade
                                ↓
Preinst backs up config.yml
                                ↓
Package files replaced
                                ↓
Postinst detects existing config → preserves it
                                ↓
Systemd service restarted with new container version
                                ↓
User configuration unchanged
```

**Removal Flow:**
```
User in CPM → Select app → Remove
                                ↓
Prerm script:
  - Stops service
  - docker compose down (removes containers)
                                ↓
Package removed
                                ↓
Postrm script:
  - Option to keep data (default) or purge
  - If purge: removes /var/lib/container-apps/<app>
  - Config preserved in /etc/container-apps/<app> (standard Debian)
```

## Converter Strategy

### Purpose

Import existing app definitions from CasaOS and Runtipi app stores to rapidly populate the container store with hundreds of applications.

### CasaOS Converter

**Input:** CasaOS app store (JSON format)
**Source:** https://github.com/IceWhaleTech/CasaOS-AppStore

**CasaOS App Format:**
```json
{
  "name": "Jellyfin",
  "icon": "https://...",
  "tagline": "The Free Software Media System",
  "overview": "Jellyfin is a Free Software Media System...",
  "thumbnail": "https://...",
  "screenshots": ["https://...", "https://..."],
  "category": "Entertainment",
  "developer": {
    "name": "Jellyfin",
    "website": "https://jellyfin.org"
  },
  "port_map": "8096",
  "scheme": "http",
  "index": "/",
  "container": {
    "image": "jellyfin/jellyfin:latest",
    "shell": "bash",
    "privileged": false,
    "network_model": "bridge",
    "web_ui": {
      "http": "8096",
      "path": "/"
    },
    "volumes": [
      {
        "container": "/config",
        "host": "/DATA/AppData/jellyfin/config"
      }
    ],
    "ports": [
      {
        "container": "8096",
        "host": "8096",
        "type": "tcp",
        "allocation": "preferred",
        "configurable": "advanced",
        "description": "WebUI HTTP Port"
      }
    ],
    "envs": [
      {
        "container": "PUID",
        "host": "1000",
        "type": "number",
        "description": "User ID"
      }
    ]
  }
}
```

**Conversion Mapping:**

| CasaOS Field | app.yaml Field | Notes |
|--------------|----------------|-------|
| `name` | `name` | Direct mapping |
| `tagline` | `tagline` | Direct mapping |
| `overview` | `description` | Direct mapping |
| `category` | `category` | Via category mapping table |
| `icon` | `icon` | Download and include |
| `screenshots` | `screenshots` | Download and include |
| `developer.website` | `homepage` | Direct mapping |
| `port_map` | `web_ui.port` | Direct mapping |
| `scheme` | `web_ui.protocol` | Direct mapping |
| `index` | `web_ui.path` | Direct mapping |
| `container` | `compose_file` | Convert to docker-compose.yml |

**Category Mapping:**
```python
CASAOS_CATEGORIES = {
    "Entertainment": "media",
    "Media": "media",
    "Productivity": "productivity",
    "Cloud": "productivity",
    "Developer": "development",
    "Home Automation": "home",
    "Network": "network",
    "Utilities": "utilities",
    "Games": "games"
}
```

**Converter Implementation:**

```python
class CasaOSConverter:
    def convert_app(self, casaos_app: dict) -> AppMetadata:
        """Convert CasaOS app to unified app.yaml format"""

        # Basic mapping
        app = AppMetadata(
            id=self.normalize_id(casaos_app['name']),
            name=casaos_app['name'],
            version=self.extract_version(casaos_app['container']['image']),
            category=self.map_category(casaos_app['category']),
            description=casaos_app['overview'],
            tagline=casaos_app['tagline']
        )

        # Download assets
        app.icon = self.download_asset(casaos_app['icon'])
        app.screenshots = [
            self.download_asset(url)
            for url in casaos_app.get('screenshots', [])
        ]

        # Convert container definition
        app.compose_file = self.generate_compose(casaos_app['container'])

        # Extract configuration
        app.configuration = self.extract_configuration(casaos_app['container'])

        # Add source tracking
        app.source = SourceInfo(
            type='casaos',
            original_id=casaos_app['name'],
            upstream_url=f"https://github.com/IceWhaleTech/CasaOS-AppStore/...",
            last_synced=datetime.now().isoformat()
        )

        return app

    def generate_compose(self, container: dict) -> str:
        """Convert CasaOS container to docker-compose.yml"""
        compose = {
            'version': '3.8',
            'services': {
                container.get('name', 'app'): {
                    'image': container['image'],
                    'container_name': '${CONTAINER_NAME}',
                    'ports': [
                        f"${{{self.env_name(p)}}}:{p['container']}/{p.get('type', 'tcp')}"
                        for p in container.get('ports', [])
                    ],
                    'volumes': [
                        f"${{{self.env_name(v)}}}:{v['container']}"
                        for v in container.get('volumes', [])
                    ],
                    'environment': {
                        env['container']: f"${{{env['container']}}}"
                        for env in container.get('envs', [])
                    },
                    'restart': 'unless-stopped'
                }
            }
        }
        return yaml.dump(compose)
```

### Runtipi Converter

**Input:** Runtipi app store (config.json + docker-compose.json)
**Source:** https://github.com/runtipi/runtipi-appstore

**Runtipi App Format:**

`config.json`:
```json
{
  "id": "signalk",
  "name": "Signal K",
  "description": "Open source data hub for marine systems",
  "version": "2.8.0",
  "categories": ["navigation", "data"],
  "short_desc": "Marine data hub",
  "author": "Signal K Foundation",
  "source": "https://github.com/SignalK/signalk-server",
  "port": 3000,
  "available": true,
  "exposable": true,
  "form_fields": [
    {
      "type": "text",
      "label": "Admin Username",
      "env_variable": "ADMIN_USERNAME",
      "required": true
    }
  ]
}
```

`docker-compose.json`:
```json
{
  "services": {
    "signalk": {
      "image": "signalk/signalk-server:${APP_VERSION}",
      "container_name": "${APP_ID}",
      "ports": ["${APP_PORT}:3000"],
      "volumes": ["${APP_DATA_DIR}/data:/data"],
      "environment": {
        "ADMIN_USERNAME": "${ADMIN_USERNAME}"
      }
    }
  }
}
```

**Conversion similar to CasaOS, with mapping for Runtipi-specific fields.**

### Converter Workflow

```
1. Clone CasaOS/Runtipi repositories
                ↓
2. For each app in source:
   - Run converter
   - Generate app.yaml
   - Download assets (icons, screenshots)
   - Generate docker-compose.yml
                ↓
3. Manual review (optional):
   - Check for issues
   - Add marine-specific metadata
   - Adjust categories
                ↓
4. Run package generator:
   - Generate debian/ directories
   - Build .deb packages
                ↓
5. Publish to apt.hatlabs.fi
                ↓
6. Available in Cockpit Package Manager
```

**Handling Duplicates:**

If same app exists in both CasaOS and Runtipi:
- Prefer one source (configurable priority)
- Store both in `source.alternatives`
- Allow user to switch sources via reconfiguration

## Configuration Management

### Why Not Debconf?

After thorough research, debconf is **not suitable** for container application configuration:

**Limitations:**
1. **Complexity**: Only handles 2-5 simple questions well
2. **No conditional fields**: Cannot show/hide based on other values
3. **Poor validation**: No built-in regex, ranges, or dependency checking
4. **Reconfiguration**: No web-based interface for `dpkg-reconfigure`
5. **User experience**: Linear wizard poor for complex forms

**Debconf is designed for simple install-time questions, not complex application configuration.**

### Recommended Approach

**Use configuration files with web-based management:**

**Installation:**
- Package installs with sensible defaults
- No debconf prompts (or max 1-2 critical questions)
- Service starts immediately with defaults
- User can configure post-install

**Configuration Storage:**
- User-facing: `/etc/container-apps/<app>/config.yml` (YAML, human-readable)
- Systemd-facing: `/etc/container-apps/<app>/environment` (KEY=value, generated)

**Workflow:**
1. User edits `config.yml` (via web UI or manually)
2. Container Manager validates changes
3. Runs helper script: `generate-env.sh config.yml > environment`
4. Systemd reloads: `systemctl restart container-<app>`
5. New settings take effect

**Benefits:**
- Complex forms in web UI
- Conditional fields, validation
- Easy manual editing for advanced users
- Survives package upgrades
- Clear separation of concerns

### Configuration Validation

**Validation happens in Container Manager UI:**
- Port ranges (1024-65535)
- Path validation (exists, writable)
- Type checking (number, boolean, string)
- Custom patterns (URLs, emails, etc.)
- Dependency checks (other services available)

**Validation before apply:**
```javascript
async function validateConfiguration(appId, config) {
  const metadata = await loadMetadata(appId);
  const errors = [];

  for (const field of metadata.configuration.env_vars) {
    const value = config[field.name];

    // Required check
    if (field.required && !value) {
      errors.push(`${field.label} is required`);
    }

    // Type validation
    if (field.type === 'number' && isNaN(value)) {
      errors.push(`${field.label} must be a number`);
    }

    // Range validation
    if (field.validation) {
      if (field.validation.min && value < field.validation.min) {
        errors.push(`${field.label} must be at least ${field.validation.min}`);
      }
      if (field.validation.max && value > field.validation.max) {
        errors.push(`${field.label} must be at most ${field.validation.max}`);
      }
    }

    // Port conflict check
    if (field.name.includes('PORT')) {
      const inUse = await checkPortInUse(value);
      if (inUse) {
        errors.push(`Port ${value} is already in use`);
      }
    }
  }

  return errors;
}
```

## Implementation Phases

### Phase 1: Foundation (Weeks 1-2)

**Deliverables:**
- [ ] Create `container-store` repository
- [ ] Define complete `app.yaml` schema with JSON schema validation
- [ ] Write package generation script (`build/generate_packages.py`)
- [ ] Create Debian package templates
- [ ] Build 3 proof-of-concept packages manually (different categories)
- [ ] Publish to test APT repository
- [ ] Install and test via `apt install`

**Success Criteria:**
- Packages install correctly
- Systemd services start
- Containers run
- Configuration files generated
- Can uninstall cleanly

### Phase 2: CasaOS Converter (Weeks 3-4)

**Deliverables:**
- [ ] Write CasaOS converter (`converters/casaos_converter.py`)
- [ ] Category mapping table
- [ ] Asset downloader (icons, screenshots)
- [ ] Docker compose generator
- [ ] Configuration schema extractor
- [ ] Batch conversion script
- [ ] Convert 50 popular CasaOS apps
- [ ] Manual review and cleanup
- [ ] Build packages and publish

**Success Criteria:**
- 50+ apps converted
- Packages build successfully
- All apps install and start
- Metadata accurate
- Assets (icons, screenshots) present

### Phase 3: CPM Integration (Weeks 5-6)

**Deliverables:**
- [ ] Design CPM configuration schema
- [ ] Implement configuration file loading in CPM
- [ ] Add "Container Apps" category view
- [ ] Display icons and screenshots
- [ ] Show source badges
- [ ] Add "Configure" button post-install
- [ ] Test integration with Phase 2 packages
- [ ] Documentation for configuration format

**Success Criteria:**
- Container apps appear in dedicated section
- Icons and metadata display correctly
- Can install apps via CPM
- "Configure" button appears after install

### Phase 4: Container Manager (MVP) (Weeks 7-9)

**Deliverables:**
- [ ] Create `cockpit-containermanager` repository
- [ ] Project setup (React, TypeScript, esbuild, PatternFly)
- [ ] List installed container apps
- [ ] Show systemd service status
- [ ] Start/Stop/Restart buttons
- [ ] Log viewer (journalctl integration)
- [ ] Text editor for `config.yml`
- [ ] "Open Web UI" button
- [ ] Build Debian package
- [ ] Install on test system

**Success Criteria:**
- Module appears in Cockpit
- Lists all container apps
- Can control services
- Can view logs
- Can edit configuration manually

### Phase 5: Dynamic Configuration Forms (Weeks 10-12)

**Deliverables:**
- [ ] Metadata loader
- [ ] Form generator (React components for each field type)
- [ ] Validation engine
- [ ] Configuration saver (YAML writer)
- [ ] Environment file generator (`generate-env.sh`)
- [ ] Service restart handler
- [ ] Configuration templates
- [ ] Port conflict detection
- [ ] Test with complex apps (many configuration options)

**Success Criteria:**
- Forms generate from metadata
- All field types work (text, number, password, select, boolean, multiselect)
- Validation prevents invalid input
- Save → restart workflow works
- Templates apply correctly

### Phase 6: Runtipi Converter (Weeks 13-14)

**Deliverables:**
- [ ] Write Runtipi converter
- [ ] Handle Runtipi-specific formats
- [ ] Convert 100+ Runtipi apps
- [ ] Merge with CasaOS catalog
- [ ] Handle duplicates (priority rules)
- [ ] Build and publish all packages

**Success Criteria:**
- 150+ total apps available
- No conflicts between sources
- All packages functional

### Phase 7: Polish & Advanced Features (Weeks 15-16)

**Deliverables:**
- [ ] Resource monitoring (CPU, RAM per container)
- [ ] Backup/restore configuration
- [ ] Bulk operations (start all marine apps, etc.)
- [ ] Update notifications
- [ ] Dependency visualization
- [ ] Container health checks
- [ ] Performance optimization
- [ ] UI polish

### Phase 8: Documentation & Release (Weeks 17-18)

**Deliverables:**
- [ ] User documentation (installation, usage)
- [ ] Developer documentation (adding apps, converters)
- [ ] Video tutorials
- [ ] Migration guide (from Runtipi)
- [ ] Release notes
- [ ] Announcement blog post
- [ ] Community feedback integration

## Technical Decisions

### 1. Package Naming: `container-*` vs `app-*` vs `cstore-*`

**Decision:** Use `container-` prefix

**Rationale:**
- Descriptive: Clearly indicates containerized app
- Namespace separation: Avoids conflicts with system packages
- Consistent: All container apps have same prefix
- Future-proof: Can add other app types later (e.g., `flatpak-*`)

### 2. Configuration Format: YAML vs JSON vs TOML

**Decision:** YAML for user-facing, KEY=value for systemd

**Rationale:**
- YAML: Human-readable, supports comments, widely used
- Comments crucial for user understanding
- systemd expects KEY=value environment files
- Generate environment file from YAML (one-way)

### 3. Category Taxonomy

**Decision:** Start with CasaOS categories, extend for marine

**Rationale:**
- CasaOS has well-established taxonomy
- More apps available in CasaOS initially
- Can extend with marine-specific categories
- Runtipi categories map reasonably to CasaOS

**Categories:**
- media (Entertainment, Streaming)
- productivity (Office, Collaboration)
- development (IDEs, Tools)
- marine (Navigation, Weather, Data)
- home (Automation, IoT)
- network (VPN, DNS, Proxy)
- utilities (File Management, Backup)
- games (Gaming Servers)

### 4. Systemd Service Type: `oneshot` vs `forking` vs `simple`

**Decision:** Use `oneshot` with `RemainAfterExit=yes`

**Rationale:**
- `docker compose up -d` returns after containers start
- `oneshot` + `RemainAfterExit` keeps service "active"
- systemd tracks service as running
- `ExecStop` runs `docker compose down` correctly
- Can use `systemctl restart` to reload containers

### 5. Data Location: `/var/lib` vs `/opt` vs `/srv`

**Decision:** Container definitions in `/opt`, data in `/var/lib`

**Rationale:**
- `/opt/container-apps/<app>/`: Static files (compose, scripts)
- `/var/lib/container-apps/<app>/`: Variable data (volumes, databases)
- `/etc/container-apps/<app>/`: Configuration files
- Follows FHS (Filesystem Hierarchy Standard)
- Clear separation of concerns

### 6. Module Architecture: Monolithic vs Separate Modules

**Decision:** Separate modules (CPM + Container Manager)

**Rationale:**
- Clean separation of concerns
- CPM remains PackageKit-only (pure package management)
- Container Manager can be complex without polluting CPM
- Users can choose to install only what they need
- Easier to maintain and test independently

### 7. Configuration Timing: Install-time vs Post-install

**Decision:** Post-install configuration via web UI

**Rationale:**
- Debconf too limited for complex configuration
- Better UX with rich web forms
- Can reconfigure easily
- No blocking during package installation
- Sensible defaults allow immediate operation

### 8. Secrets Handling: Plain files vs systemd credentials vs vault

**Decision:** MVP uses plain environment files, plan for credentials later

**Rationale:**
- Plain files standard for Docker compose (industry practice)
- File permissions (600, root-only) provide basic security
- systemd credentials (LoadCredential) can be added later
- Vault integration overkill for embedded systems
- Document security best practices for users

## Open Questions & Future Considerations

### 1. Multi-Container Applications

**Question:** How to handle apps requiring multiple compose services?

**Options:**
- A) Single package with multi-service compose (simplest)
- B) Multiple packages with dependencies (more granular)
- C) "Meta-packages" that install dependencies (like tasks)

**Recommendation:** Start with A, evaluate B/C if needed

### 2. App Dependencies

**Question:** How to handle app dependencies (e.g., Grafana requires InfluxDB)?

**Current approach:**
- Use Debian `Depends:` for hard requirements
- Use `Recommends:` for optional but suggested
- Document in app metadata

**Future enhancement:**
- Container Manager shows dependency graph
- "Install with dependencies" button
- Warn before removing if other apps depend on it

### 3. Port Conflicts

**Question:** How to handle two apps wanting same port?

**Current approach:**
- Configuration allows changing host port
- Container Manager warns if port in use
- User must manually resolve

**Future enhancement:**
- Auto-assign alternative ports
- Port registry system
- Suggest free ports during configuration

### 4. Updates & Versioning

**Question:** How to handle container image updates vs package updates?

**Approach:**
- Package version tied to container image version
- `docker compose pull` in ExecStartPre gets latest image
- Package update = new compose file + metadata
- User controls when to upgrade package

**Future consideration:**
- Auto-update container images (without package upgrade)
- Rollback mechanism
- Update channels (stable, beta, nightly)

### 5. Backup & Restore

**Question:** How to backup/restore app data and configuration?

**Approach:**
- Configuration in `/etc` backed up by normal system backup
- Data in `/var/lib` backed up separately
- Document backup procedures

**Future enhancement:**
- Container Manager "Backup" button
- Export configuration + data as tarball
- Import to restore on new system
- Scheduled backups via systemd timer

### 6. Resource Limits

**Question:** Should we enforce CPU/memory limits on containers?

**Approach:**
- Not by default (let containers use what they need)
- Advanced users can add limits to compose file
- Document in metadata (memory_min as guidance)

**Future enhancement:**
- Container Manager shows resource usage
- Allow setting limits via UI
- Warn if container exceeds suggested limits

### 7. Multi-Architecture Support

**Question:** How to handle apps that only support amd64 or arm64?

**Approach:**
- Metadata includes `requires.architecture`
- Package generation checks architecture
- Only build packages for supported architectures
- APT filters by architecture automatically

**Implementation:**
- Use `Architecture: amd64` or `Architecture: arm64` in control file
- Or `Architecture: all` if container image supports both

### 8. Migration from Runtipi

**Question:** How to migrate existing Runtipi users to container-store?

**Approach:**
- Write migration tool
- Export Runtipi app configurations
- Map to equivalent container-store packages
- Import configurations
- Document manual steps

**Considerations:**
- Data path changes
- Port changes
- Service name changes

### 9. Community Contributions

**Question:** How to accept community-contributed apps?

**Approach:**
- GitHub repository with PR workflow
- Automated validation (schema checks, linting)
- Manual review by maintainers
- CI builds and tests packages
- Merge → auto-publish to apt repo

**Infrastructure:**
- GitHub Actions for CI/CD
- Automated package building
- Publish to apt.hatlabs.fi via webhook

### 10. Security & Updates

**Question:** How to handle security updates for container images?

**Approach:**
- Monitor upstream container registries
- Update package version when new image released
- Users update via `apt upgrade`
- Consider automated security updates (unattended-upgrades)

**Future:**
- Vulnerability scanning integration
- Security advisories in CPM
- Auto-update critical security patches

## Success Metrics

### Technical Metrics

- **Package count**: 200+ apps available
- **Install success rate**: >95%
- **Service start success**: >98%
- **Build time**: <30s per package
- **Package size**: <100KB (excluding container images)

### User Experience Metrics

- **Time to install app**: <2 minutes
- **Time to configure app**: <5 minutes
- **Documentation completeness**: All apps have README
- **User errors**: <5% misconfiguration rate

### Adoption Metrics

- **HaLOS images**: Pre-installed in marine variants
- **Community engagement**: GitHub stars, issues, PRs
- **User feedback**: Positive sentiment in forums/chat

## Risks & Mitigations

### Risk 1: Package Conflicts

**Risk:** Name conflicts with existing Debian packages

**Mitigation:** Use `container-` prefix, check against Debian package database

### Risk 2: Docker Dependency

**Risk:** All apps require Docker, adds overhead

**Mitigation:** Document system requirements, consider lightweight alternatives (Podman)

### Risk 3: Converter Quality

**Risk:** Automated conversion produces broken packages

**Mitigation:** Manual review, automated testing, gradual rollout

### Risk 4: Configuration Complexity

**Risk:** Dynamic forms difficult to implement correctly

**Mitigation:** Start with simple text editor, iterate to forms, extensive testing

### Risk 5: Maintenance Burden

**Risk:** 200+ packages require ongoing updates

**Mitigation:** Automate update detection, community contributions, upstream monitoring

### Risk 6: User Migration

**Risk:** Runtipi users resist migration

**Mitigation:** Migration tool, clear documentation, side-by-side operation period

## Conclusion

This architecture provides a solid foundation for replacing Runtipi with a Debian-native container application delivery system. By leveraging standard Linux tools and Cockpit's existing infrastructure, we can provide a simpler, more maintainable solution while offering better integration with the underlying system.

**Key Advantages:**
- Standard Debian packaging (no custom orchestration)
- Cockpit integration (familiar UI)
- Systemd lifecycle management (proven, reliable)
- Extensible architecture (easy to add new sources)
- Configuration-driven design (CPM remains pure PackageKit)
- Separation of concerns (Package Manager vs Container Manager)

**Next Steps:**
1. Review and refine this architecture
2. Create proof-of-concept with 3-5 apps
3. Begin Phase 1 implementation
4. Iterate based on user feedback

---

**Document Status:** Draft - Ready for review and discussion

**Last Updated:** 2025-11-06

**Contributors:** Matti Airas, Claude Code

**References:**
- [Debian Policy Manual](https://www.debian.org/doc/debian-policy/)
- [Cockpit Developer Guide](https://cockpit-project.org/guide/latest/)
- [CasaOS App Store](https://github.com/IceWhaleTech/CasaOS-AppStore)
- [Runtipi App Store](https://github.com/runtipi/runtipi-appstore)
- [systemd Service Management](https://www.freedesktop.org/software/systemd/man/systemd.service.html)
