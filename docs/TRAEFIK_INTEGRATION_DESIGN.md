# Traefik Reverse Proxy Integration - Design Document

**Status**: Design (Phase 4)
**Date**: 2025-11-11
**Last Updated**: 2025-11-11

## Overview

Traefik provides a reverse proxy for container applications, enabling unified URL schemes instead of accessing apps via random TCP ports. This creates a more cohesive and professional user experience.

## Purpose

- Provide clean URLs for accessing container apps
- Eliminate need to remember port numbers
- Enable HTTPS with automatic certificate management
- Optional integration - apps can still use direct ports
- User choice per application (proxy vs direct access)

## Design Principles

1. **Optional**: Apps work without Traefik (direct port access as fallback)
2. **Auto-Configuration**: Uses Docker labels (Runtipi style) for zero-config routing
3. **Flexible**: User choice per app - Traefik route, direct port, or both
4. **Secure**: Auto-detect self-signed vs Let's Encrypt based on domain configuration
5. **Transparent**: Works seamlessly with Homarr dashboard integration

## Architecture

### Components

```
┌─────────────────────────────────────────────┐
│           User's Browser                     │
│   http://halos.local/signalk                │
│   or http://halos.local:3000 (direct)       │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
     ┌─────────────────────────────┐
     │  halos-traefik-container    │
     │  Reverse Proxy (80/443)     │
     └──────┬──────────────────────┘
            │
            ├─ Routes to signalk:3000
            ├─ Routes to grafana:3001
            └─ Routes to opencpn:8080
                      │
                      ▼
              ┌──────────────────┐
              │  Docker Network  │
              │  Container Apps  │
              └──────────────────┘
```

### Package Structure

**halos-traefik-container**
- Type: Container application package
- Naming: `halos-traefik-container_<version>_<arch>.deb`
- Contains: Docker Compose file, Traefik configuration, systemd service
- Service: `halos-traefik-container.service`
- Ports: 80 (HTTP), 443 (HTTPS)
- Config: `/var/lib/container-apps/halos-traefik-container/traefik.yml`
- Certificates: `/var/lib/container-apps/halos-traefik-container/certs/`

## Routing Configuration

### Docker Label-Based Configuration

Apps define Traefik routing via Docker labels in `docker-compose.yml` (Runtipi style):

```yaml
services:
  signalk:
    image: signalk/signalk-server:latest
    labels:
      # Traefik routing
      - "traefik.enable=true"
      - "traefik.http.routers.signalk.rule=PathPrefix(`/signalk`)"
      - "traefik.http.routers.signalk.entrypoints=web"
      - "traefik.http.services.signalk.loadbalancer.server.port=3000"

      # Optional: Strip path prefix
      - "traefik.http.middlewares.signalk-stripprefix.stripprefix.prefixes=/signalk"
      - "traefik.http.routers.signalk.middlewares=signalk-stripprefix"

      # Optional: HTTPS redirect
      - "traefik.http.routers.signalk-secure.rule=PathPrefix(`/signalk`)"
      - "traefik.http.routers.signalk-secure.entrypoints=websecure"
      - "traefik.http.routers.signalk-secure.tls=true"

    # Still expose port for direct access (optional)
    ports:
      - "3000:3000"

    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

### URL Scheme

**Status**: TBD (To Be Determined)

Options to be decided later:
- **Path-based**: `halos.local/signalk`, `halos.local/grafana`
- **Subdomain-based**: `signalk.halos.local`, `grafana.halos.local`
- **Hybrid**: User preference or app-specific choice

**Current Approach**: Use path-based in examples, final decision during implementation.

## HTTPS/TLS Configuration

### Auto-Detection Strategy

Traefik automatically determines certificate strategy:

1. **Check for domain configuration**: Look for `/etc/traefik/domain.txt`
2. **If domain exists**: Use Let's Encrypt (ACME)
3. **If no domain**: Use self-signed certificate

**Self-Signed Certificate**:
```yaml
# traefik.yml
tls:
  certificates:
    - certFile: /certs/halos.local.crt
      keyFile: /certs/halos.local.key
```

**Let's Encrypt**:
```yaml
# traefik.yml
certificatesResolvers:
  letsencrypt:
    acme:
      email: admin@halos.local
      storage: /certs/acme.json
      httpChallenge:
        entryPoint: web
```

### Certificate Management

**Self-Signed Generation** (on first boot):
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /certs/halos.local.key \
  -out /certs/halos.local.crt \
  -subj "/CN=halos.local"
```

**Let's Encrypt Renewal**: Automatic via Traefik ACME

## Integration Points

### With Homarr Dashboard

Homarr links updated to use Traefik routes:

**Without Traefik** (Phase 3.5):
```yaml
labels:
  - "homarr.url=http://halos.local:3000"
```

**With Traefik** (Phase 4):
```yaml
labels:
  - "homarr.url=http://halos.local/signalk"  # Traefik route
  - "homarr.url.direct=http://halos.local:3000"  # Fallback
```

Homarr dashboard shows both options if available.

### With Container Apps

**User Choice Per App**:

1. **Traefik Only**: Remove port mapping, access via proxy only
   ```yaml
   # No ports section, Traefik labels only
   ```

2. **Direct Port Only**: No Traefik labels
   ```yaml
   ports:
     - "3000:3000"
   # No traefik.enable label
   ```

3. **Both** (Recommended): Port + Traefik route
   ```yaml
   ports:
     - "3000:3000"
   labels:
     - "traefik.enable=true"
     # ... Traefik configuration
   ```

## Traefik Configuration

### Static Configuration

`/var/lib/container-apps/halos-traefik-container/traefik.yml`:

```yaml
# Entry points (ports)
entryPoints:
  web:
    address: ":80"
    # HTTP to HTTPS redirect (if TLS enabled)
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https

  websecure:
    address: ":443"

# Docker provider
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: traefik-net

# API and dashboard
api:
  dashboard: true
  insecure: false  # Dashboard via Traefik route

# Logging
log:
  level: INFO
  filePath: /var/log/traefik/traefik.log

accessLog:
  filePath: /var/log/traefik/access.log

# TLS configuration (dynamic, based on domain detection)
# See "HTTPS/TLS Configuration" section above
```

### Docker Compose

`/var/lib/container-apps/halos-traefik-container/docker-compose.yml`:

```yaml
version: '3.8'

services:
  traefik:
    image: traefik:v3.0
    container_name: halos-traefik
    restart: unless-stopped
    command:
      - "--configFile=/etc/traefik/traefik.yml"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /var/lib/container-apps/halos-traefik-container/traefik.yml:/etc/traefik/traefik.yml:ro
      - /var/lib/container-apps/halos-traefik-container/certs:/certs
      - /var/log/traefik:/var/log/traefik
    networks:
      - traefik-net
    labels:
      # Traefik dashboard
      - "traefik.enable=true"
      - "traefik.http.routers.traefik-dashboard.rule=PathPrefix(`/traefik`)"
      - "traefik.http.routers.traefik-dashboard.service=api@internal"
      - "traefik.http.routers.traefik-dashboard.middlewares=traefik-auth"

      # Basic auth for dashboard (optional)
      # - "traefik.http.middlewares.traefik-auth.basicauth.users=admin:$$apr1$$..."

      # Homarr integration
      - "homarr.enable=true"
      - "homarr.name=Traefik Dashboard"
      - "homarr.url=http://halos.local/traefik"
      - "homarr.category=System"

networks:
  traefik-net:
    driver: bridge
    name: traefik-net
```

## Label Generation

### container-packaging-tools Integration

Generate Traefik labels automatically from `metadata.json`:

```python
def generate_traefik_labels(metadata):
    """Generate Traefik labels from app metadata"""
    package_name = metadata['package_name'].replace('-container', '')
    labels = []

    # Only if web UI enabled
    if not metadata.get('web_ui', {}).get('enabled'):
        return labels

    labels.append("traefik.enable=true")

    # Router rule (path-based for now, TBD: subdomain option)
    labels.append(f"traefik.http.routers.{package_name}.rule=PathPrefix(`/{package_name}`)")
    labels.append(f"traefik.http.routers.{package_name}.entrypoints=web")

    # Service configuration
    port = metadata['web_ui']['port']
    labels.append(f"traefik.http.services.{package_name}.loadbalancer.server.port={port}")

    # Optional: Strip path prefix if app doesn't handle it
    if metadata['web_ui'].get('strip_prefix', True):
        labels.append(f"traefik.http.middlewares.{package_name}-stripprefix.stripprefix.prefixes=/{package_name}")
        labels.append(f"traefik.http.routers.{package_name}.middlewares={package_name}-stripprefix")

    # HTTPS router (if TLS enabled)
    labels.append(f"traefik.http.routers.{package_name}-secure.rule=PathPrefix(`/{package_name}`)")
    labels.append(f"traefik.http.routers.{package_name}-secure.entrypoints=websecure")
    labels.append(f"traefik.http.routers.{package_name}-secure.tls=true")

    return labels
```

### Updated metadata.json Format

Add optional Traefik configuration:

```json
{
  "web_ui": {
    "enabled": true,
    "port": 3000,
    "protocol": "http",
    "path": "/",
    "strip_prefix": true,
    "traefik": {
      "enabled": true,
      "route": "/signalk",  // Optional custom route
      "subdomain": false     // Future: subdomain-based routing
    }
  }
}
```

## Installation and Dependencies

### Pre-Installation in HaLOS Images

`halos-traefik-container` pre-installed in all HaLOS images.

**Rationale**: Provides unified URL experience from first boot.

### Store Package Suggestions

Store packages suggest Traefik:

```
Package: marine-container-store
Suggests: halos-traefik-container
```

**Rationale**: Works without Traefik (direct ports), enhanced with Traefik.

### Dependency Strategy

**TBD**: Should Traefik be a required dependency?

Options:
- Pre-installed (current plan) - always available
- Recommended dependency - installed by default but removable
- Optional - users install manually

**Decision**: Defer until implementation phase.

## Port Allocation Strategy

### Recommended Approach: Both Traefik + Direct Port

Apps expose both routes:
- **Traefik route**: Clean URL (e.g., `halos.local/signalk`)
- **Direct port**: Fallback/power users (e.g., `halos.local:3000`)

**Benefits**:
- Flexibility for different use cases
- Fallback if Traefik fails
- Compatibility with tools expecting specific ports

**Docker Compose Example**:
```yaml
services:
  signalk:
    ports:
      - "3000:3000"  # Direct access
    labels:
      - "traefik.enable=true"  # Also via Traefik
      # ... Traefik configuration
    networks:
      - default
      - traefik-net  # Connect to both networks
```

## User Configuration

### Per-App Traefik Enable/Disable

**Via cockpit-container-config** (Phase 2):
- Toggle "Enable Traefik routing" per app
- Updates `traefik.enable` label and restarts container
- Port access remains available

**Via docker-compose.yml** (Power users):
- Edit compose file directly
- Set `traefik.enable=false` to disable

### Global Traefik Disable

Stop Traefik service:
```bash
sudo systemctl stop halos-traefik-container.service
sudo systemctl disable halos-traefik-container.service
```

All apps remain accessible via direct ports.

## Error Handling

### Traefik Unavailable

- Apps still accessible via direct ports
- Homarr shows direct URLs as fallback
- Error message in Traefik dashboard area

### Routing Conflicts

- Traefik logs conflicts (duplicate routes)
- First-configured app wins
- Admin can resolve via container config UI

### Certificate Errors

**Self-Signed**:
- Browser warnings expected
- Provide documentation on accepting cert

**Let's Encrypt**:
- Retry with exponential backoff
- Fallback to self-signed if repeated failures
- Log errors for troubleshooting

## Testing Strategy

### Unit Tests

- Label generation from metadata
- Traefik config validation
- Certificate detection logic

### Integration Tests

- Full Traefik + container app routing
- HTTPS with self-signed certs
- Dynamic configuration updates
- Fallback to direct ports

### End-to-End Tests

1. Install `halos-traefik-container`
2. Install app with Traefik labels (e.g., `signalk-server-container`)
3. Verify app accessible via Traefik route (`halos.local/signalk`)
4. Verify app accessible via direct port (`halos.local:3000`)
5. Stop Traefik, verify direct port still works
6. Check Homarr shows both URLs

## Migration from Runtipi

Since Runtipi is removed (not migrated), no migration needed.

**Reference**: Review Runtipi's Traefik configuration for inspiration:
- Label patterns
- Middleware configuration
- Certificate management

## Future Enhancements

### Phase 4 (Current)
- Basic Traefik with path-based routing
- Self-signed or Let's Encrypt HTTPS
- Optional integration

### Phase 5+ (Future)
- Subdomain-based routing option
- Advanced middleware (auth, rate limiting)
- Load balancing for multi-instance apps
- Geographic routing (if multiple HaLOS instances)
- Traefik metrics in Homarr dashboard

## Open Questions (TBD)

Decisions deferred to implementation phase:

1. **URL Scheme**: Path-based vs subdomain-based vs hybrid?
2. **Required Dependency**: Should Traefik be required for store packages?
3. **metadata.json Changes**: Exact schema for Traefik configuration?
4. **Default HTTPS**: Always use HTTPS (self-signed) or HTTP by default?

**Note**: These will be resolved during Phase 4 planning, before implementation.

## References

### Internal Documentation
- [META-PLANNING.md](../META-PLANNING.md) - Overall project planning
- [HOMARR_INTEGRATION_DESIGN.md](./HOMARR_INTEGRATION_DESIGN.md) - Dashboard integration
- [container-packaging-tools/docs/DESIGN.md](../container-packaging-tools/docs/DESIGN.md) - Label generation

### External References
- [Traefik Documentation](https://doc.traefik.io/traefik/) - Official docs
- [Traefik Docker Provider](https://doc.traefik.io/traefik/providers/docker/) - Docker label configuration
- [Traefik Let's Encrypt](https://doc.traefik.io/traefik/https/acme/) - Automatic HTTPS
- [Runtipi Traefik Config](https://github.com/runtipi/runtipi) - Reference implementation
