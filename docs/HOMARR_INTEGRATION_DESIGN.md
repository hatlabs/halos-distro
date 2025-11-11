# Homarr Dashboard Integration - Design Document

**Status**: Design (Phase 3.5)
**Date**: 2025-11-11
**Last Updated**: 2025-11-11

## Overview

Homarr provides a unified dashboard landing page for accessing all installed container applications. It serves as the primary entry point for users accessing their HaLOS system, eliminating the need to remember individual application ports.

## Purpose

- Provide a visual dashboard showing all installed container apps
- Single landing page at `http://halos.local/` for system access
- Quick access to Cockpit, container apps, and system information
- User-customizable layout and organization
- Auto-discovery of newly installed container apps

## Design Principles

1. **Auto-Discovery**: Automatically detect and add newly installed container apps
2. **User Control**: Highly customizable - users can add/remove/reorder apps
3. **Simplicity**: Minimal configuration required, works out of the box
4. **Performance**: Lightweight native adapter (Rust) with minimal memory footprint
5. **Separation**: Dashboard (Homarr) separate from adapter service

## Architecture

### Components

```
┌─────────────────────────────────────────────────────────┐
│                    User's Browser                        │
│                  http://halos.local/                     │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
        ┌──────────────────────────────┐
        │   homarr-container (port 80)  │
        │   Dashboard Web UI            │
        └──────────┬───────────────────┘
                   │ Homarr API
                   ▼
      ┌────────────────────────────────────┐
      │  homarr-container-adapter (native) │
      │  Rust service, systemd timer       │
      └─────┬──────────────────────────────┘
            │
            ├─ Scans Docker containers ────►┐
            ├─ Reads container labels  ◄────┤
            └─ Updates Homarr config ────────┘
                         │
                         ▼
                ┌────────────────┐
                │  Docker Engine  │
                │  Running Apps   │
                └────────────────┘
```

### Package Structure

**1. homarr-container**
- Type: Container application package
- Naming: `homarr-container_<version>_<arch>.deb`
- Contains: Docker Compose file, systemd service, Homarr configuration
- Service: `homarr-container.service`
- Port: 80 (HTTP), 443 (HTTPS if Traefik enabled)
- Volume: `/var/lib/container-apps/homarr-container/data` for persistent config

**2. homarr-container-adapter**
- Type: Native Debian package (Rust binary)
- Naming: `homarr-container-adapter_<version>_<arch>.deb`
- Binary: `/usr/bin/homarr-container-adapter`
- Service: `homarr-container-adapter.service` (oneshot) + `homarr-container-adapter.timer`
- Config: `/etc/homarr-container-adapter/config.toml`
- State: `/var/lib/homarr-container-adapter/state.json` (tracks removed apps)

## Auto-Discovery Mechanism

### Docker Label-Based Discovery

Container apps expose metadata via Docker labels in their `docker-compose.yml`:

```yaml
services:
  signalk:
    image: signalk/signalk-server:latest
    labels:
      # Homarr integration
      - "homarr.enable=true"
      - "homarr.name=Signal K Server"
      - "homarr.description=Marine data processing and routing"
      - "homarr.icon=/icons/signalk.png"
      - "homarr.url=http://halos.local:3000"
      - "homarr.category=Marine"

      # Optional: Traefik labels (Phase 4)
      - "traefik.enable=true"
      - "traefik.http.routers.signalk.rule=PathPrefix(`/signalk`)"
      # ... more Traefik config
```

### Adapter Workflow

**Periodic Scan** (systemd timer, every 60 seconds):

1. **Query Docker API**: List all running containers
2. **Extract Labels**: Read `homarr.*` labels from each container
3. **Load State**: Read `/var/lib/homarr-container-adapter/state.json`
4. **Filter Apps**:
   - Include: Apps with `homarr.enable=true`
   - Exclude: Apps in `removed_apps` list (user removed from dashboard)
5. **Update Homarr**: Call Homarr API to add/update apps
6. **Save State**: Write updated state to disk

**State File Format** (`/var/lib/homarr-container-adapter/state.json`):
```json
{
  "version": "1.0",
  "removed_apps": [
    "signalk-server-container",
    "another-app-container"
  ],
  "last_sync": "2025-11-11T12:00:00Z",
  "discovered_apps": {
    "grafana-container": {
      "name": "Grafana",
      "url": "http://halos.local:3001",
      "added_at": "2025-11-10T15:30:00Z"
    }
  }
}
```

### Rust Adapter Implementation

**Key Crates**:
- `bollard` - Docker API client
- `reqwest` - HTTP client for Homarr API
- `serde` / `serde_json` - Serialization
- `tokio` - Async runtime
- `systemd` - systemd integration

**Pseudo-code**:
```rust
async fn sync_apps() -> Result<()> {
    // 1. Connect to Docker
    let docker = Docker::connect_with_local_defaults()?;

    // 2. List running containers
    let containers = docker.list_containers(None).await?;

    // 3. Extract Homarr labels
    let mut discovered_apps = Vec::new();
    for container in containers {
        if let Some(labels) = container.labels {
            if labels.get("homarr.enable") == Some(&"true".to_string()) {
                discovered_apps.push(parse_homarr_labels(&labels)?);
            }
        }
    }

    // 4. Load state (removed apps)
    let state = State::load()?;

    // 5. Filter out removed apps
    let apps_to_add: Vec<_> = discovered_apps
        .into_iter()
        .filter(|app| !state.removed_apps.contains(&app.id))
        .collect();

    // 6. Call Homarr API to update
    let homarr_client = HomarrClient::new("http://localhost/api")?;
    for app in apps_to_add {
        homarr_client.add_or_update_app(app).await?;
    }

    // 7. Save state
    state.last_sync = Utc::now();
    state.save()?;

    Ok(())
}
```

## Homarr API Integration

The adapter communicates with Homarr via its REST API:

**Add/Update App**:
```http
POST /api/modules/ping
Content-Type: application/json

{
  "name": "Signal K Server",
  "url": "http://halos.local:3000",
  "icon": "/icons/signalk.png",
  "category": "Marine",
  "description": "Marine data processing and routing"
}
```

**List Apps** (for state reconciliation):
```http
GET /api/modules/ping
```

**Remove App** (manual user action tracked):
```http
DELETE /api/modules/ping/{id}
```

## User Customization

### Capabilities

Users can customize via Homarr's built-in UI:
- ✅ Add/remove apps from dashboard
- ✅ Reorder apps (drag and drop)
- ✅ Customize app icons and names
- ✅ Group apps into categories
- ✅ Add widgets (weather, system stats, etc.)
- ✅ Choose themes and layouts

### Tracking Removed Apps

When a user removes an app from Homarr dashboard:

1. **Homarr Event**: User clicks "Remove" on an app
2. **Adapter Detection**: Next sync detects missing app (compare Docker vs Homarr)
3. **State Update**: Add app ID to `removed_apps` list
4. **Persistence**: App stays removed even after adapter restarts

**Recovery**: User can manually re-add via Homarr UI, adapter won't remove it again.

## Default Landing Page

### Configuration

Homarr runs on port 80, making it the default landing page:
- `http://halos.local/` → Homarr dashboard
- `https://halos.local:9090/` → Cockpit (unchanged)
- Individual apps: Via Homarr links or direct ports

### Homarr Dashboard Content

**Default Widgets/Apps**:
- Cockpit (always shown)
- Installed container apps (auto-discovered)
- System stats widget (CPU, memory, disk)
- Quick links (documentation, support)

**Example Layout**:
```
┌───────────────────────────────────────────────────────┐
│ HaLOS Dashboard                          [Settings ⚙] │
├───────────────────────────────────────────────────────┤
│                                                        │
│  System Management                                    │
│  ┌─────────┐                                          │
│  │ Cockpit │  Manage system, users, services         │
│  └─────────┘                                          │
│                                                        │
│  Marine Applications                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │Signal K  │  │ OpenCPN  │  │ Grafana  │           │
│  └──────────┘  └──────────┘  └──────────┘           │
│                                                        │
│  System Status                                        │
│  CPU: 23%  Memory: 45%  Disk: 67%                    │
└───────────────────────────────────────────────────────┘
```

## Installation and Dependencies

### Pre-Installation in HaLOS Images

Both packages pre-installed in HaLOS image builds:
- `homarr-container` - Dashboard application
- `homarr-container-adapter` - Auto-discovery service

**Rationale**: Provides immediate, cohesive user experience on first boot.

### Store Package Suggestions

Store packages (e.g., `marine-container-store`) suggest but don't require:

```
Package: marine-container-store
Suggests: homarr-container, homarr-container-adapter
```

**Rationale**: Works without Homarr (users access apps via ports), but enhanced experience with Homarr.

### Meta-Package Option

Optional meta-package for manual installations:

```
Package: halos-dashboard
Depends: homarr-container, homarr-container-adapter
Description: HaLOS unified dashboard
 Installs Homarr dashboard and auto-discovery adapter
```

## Container App Label Requirements

### Mandatory Labels

For an app to appear in Homarr:
```yaml
labels:
  - "homarr.enable=true"
  - "homarr.name=App Name"
  - "homarr.url=http://halos.local:PORT"
```

### Optional Labels

```yaml
labels:
  - "homarr.description=Short description"
  - "homarr.icon=/path/to/icon.png"  # Or URL
  - "homarr.category=Marine"  # For grouping
```

### Label Generation

**container-packaging-tools** generates these labels automatically from `metadata.json`:

```python
# In template rendering
def generate_homarr_labels(metadata):
    labels = [
        "homarr.enable=true",
        f"homarr.name={metadata['name']}",
    ]

    if metadata.get('web_ui', {}).get('enabled'):
        port = metadata['web_ui']['port']
        protocol = metadata['web_ui'].get('protocol', 'http')
        labels.append(f"homarr.url={protocol}://halos.local:{port}")

    if metadata.get('description'):
        labels.append(f"homarr.description={metadata['description']}")

    # Icon URL (served from package or external)
    if metadata.get('icon'):
        labels.append(f"homarr.icon=/icons/{metadata['package_name']}.png")

    # Category from tags
    for tag in metadata.get('tags', []):
        if tag.startswith('field::'):
            category = tag.split('::')[1].capitalize()
            labels.append(f"homarr.category={category}")
            break

    return labels
```

## Configuration

### Adapter Configuration

`/etc/homarr-container-adapter/config.toml`:

```toml
[docker]
# Docker socket path
socket = "/var/run/docker.sock"

[homarr]
# Homarr API endpoint
api_url = "http://localhost/api"
# API key (if authentication enabled)
api_key = ""

[sync]
# Sync interval (seconds)
interval = 60
# Enable debug logging
debug = false

[state]
# State file location
state_file = "/var/lib/homarr-container-adapter/state.json"
```

### systemd Units

**Service** (`homarr-container-adapter.service`):
```ini
[Unit]
Description=Homarr Container Adapter
After=docker.service homarr-container.service
Requires=docker.service

[Service]
Type=oneshot
ExecStart=/usr/bin/homarr-container-adapter sync
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**Timer** (`homarr-container-adapter.timer`):
```ini
[Unit]
Description=Homarr Container Adapter Timer
Requires=homarr-container-adapter.service

[Timer]
OnBootSec=2min
OnUnitActiveSec=60s
AccuracySec=5s

[Install]
WantedBy=timers.target
```

## Error Handling

### Adapter Resilience

**Docker Connection Failure**:
- Retry with exponential backoff
- Log error, continue next sync cycle
- Don't crash adapter

**Homarr API Unavailable**:
- Skip update, try next cycle
- Cache discovered apps for retry
- Log warning

**Invalid Labels**:
- Skip malformed labels
- Log validation errors
- Continue processing other containers

**State File Corruption**:
- Rebuild state from current Docker state
- Backup corrupted file for debugging
- Start fresh with empty `removed_apps`

## Testing Strategy

### Unit Tests

- Label parsing logic
- State file serialization/deserialization
- Filter logic (removed apps)
- Homarr API client

### Integration Tests

- Docker API integration
- Full sync workflow
- State persistence across restarts
- Error recovery scenarios

### End-to-End Tests

1. Install `homarr-container` + `homarr-container-adapter`
2. Install container app (e.g., `signalk-server-container`)
3. Verify app appears in Homarr dashboard (within 60s)
4. Remove app from dashboard via Homarr UI
5. Restart adapter
6. Verify app doesn't re-appear (removed_apps tracking)

## Future Enhancements

### Phase 3.5 (Current)
- Basic Homarr + adapter implementation
- Auto-discovery via Docker labels
- User customization support

### Phase 4 (Traefik Integration)
- Homarr links use Traefik routes instead of ports
- Labels include both Homarr and Traefik config
- Unified URL scheme

### Phase 5+ (Future)
- App health monitoring in dashboard
- One-click app restart from Homarr
- Container resource usage widgets
- Notification system (app failures, updates available)
- Mobile app for dashboard access

## References

### Internal Documentation
- [META-PLANNING.md](../META-PLANNING.md) - Overall project planning
- [TRAEFIK_INTEGRATION_DESIGN.md](./TRAEFIK_INTEGRATION_DESIGN.md) - Reverse proxy integration
- [container-packaging-tools/docs/DESIGN.md](../container-packaging-tools/docs/DESIGN.md) - Label generation

### External References
- [Homarr Documentation](https://homarr.dev/) - Dashboard features and API
- [Homarr GitHub](https://github.com/ajnart/homarr) - Source code
- [Bollard Crate](https://docs.rs/bollard/) - Docker API for Rust
- [Docker Labels](https://docs.docker.com/config/labels-custom-metadata/) - Label specification
