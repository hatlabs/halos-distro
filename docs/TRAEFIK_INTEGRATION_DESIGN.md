# Traefik Reverse Proxy Integration - Design Document

**Status**: Partially Implemented
**Date**: 2025-11-11
**Last Updated**: 2025-12-29

## Overview

Traefik provides a reverse proxy for container applications, enabling unified URL schemes instead of accessing apps via random TCP ports. This creates a more cohesive and professional user experience.

## Purpose

- Provide clean subdomain-based URLs for accessing container apps
- Eliminate need to remember port numbers
- Enable HTTPS with self-signed certificates
- Centralize authentication via Authelia integration
- Homarr dashboard links always point to subdomain URLs

## Design Principles

1. **Subdomain-Only**: All HTTP apps are accessed exclusively via subdomain URLs (e.g., `grafana.halos.local`)
2. **No Direct Port Fallback**: Apps should NOT expose ports unless absolutely necessary
3. **Auto-Configuration**: Uses Docker labels for zero-config routing
4. **Secure**: Self-signed certificates for HTTPS (Let's Encrypt deferred to future work)
5. **Transparent**: Works seamlessly with Homarr dashboard and SSO integration

### When Direct Port Exposure Is Allowed

Apps may expose ports only when required:
- **Non-HTTP protocols**: NMEA streams, CAN bus, WebSocket-only services
- **Host networking requirement**: Apps that must use `network_mode: host`
- **External tool compatibility**: When third-party tools require specific ports

## Architecture

### Components

```
┌─────────────────────────────────────────────────┐
│              User's Browser                      │
│   https://grafana.halos.local                   │
│   https://signalk.halos.local                   │
│   https://halos.local (Homarr dashboard)        │
└──────────────────┬──────────────────────────────┘
                   │
                   ▼
     ┌─────────────────────────────────┐
     │  traefik-container              │
     │  Reverse Proxy (80/443)         │
     │  + ForwardAuth → Authelia       │
     └──────┬──────────────────────────┘
            │
            ├─ grafana.halos.local → grafana:3001
            ├─ signalk.halos.local → host:3000 (host networking)
            ├─ influxdb.halos.local → influxdb:8086
            ├─ auth.halos.local → authelia:9091
            └─ halos.local → homarr:7575
                      │
                      ▼
              ┌──────────────────┐
              │  Docker Network  │
              │halos-proxy-network│
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

Apps define Traefik routing via Docker labels in `docker-compose.yml`:

```yaml
services:
  grafana:
    image: grafana/grafana:latest
    labels:
      # Traefik routing - subdomain based
      - "traefik.enable=true"
      - "traefik.http.routers.grafana.rule=Host(`grafana.${HALOS_DOMAIN}`)"
      - "traefik.http.routers.grafana.entrypoints=web"
      - "traefik.http.routers.grafana.middlewares=redirect-to-https@file"
      - "traefik.http.routers.grafana-secure.rule=Host(`grafana.${HALOS_DOMAIN}`)"
      - "traefik.http.routers.grafana-secure.entrypoints=websecure"
      - "traefik.http.routers.grafana-secure.tls=true"
      - "traefik.http.routers.grafana-secure.middlewares=authelia@file"
      - "traefik.http.services.grafana.loadbalancer.server.port=3000"

      # mDNS subdomain advertisement
      - "halos.subdomain=grafana"

    # NO port exposure - access via Traefik only
    # ports:
    #   - "3001:3000"  # REMOVED

    networks:
      - halos-proxy-network

networks:
  halos-proxy-network:
    external: true
```

### URL Scheme

**Decision**: Subdomain-based routing only.

All apps are accessed via subdomains:
- `halos.local` - Homarr dashboard (root domain)
- `auth.halos.local` - Authelia login portal
- `grafana.halos.local` - Grafana
- `signalk.halos.local` - Signal K Server
- `influxdb.halos.local` - InfluxDB
- `cockpit.halos.local` - Cockpit web console

Path-based routing (e.g., `halos.local/grafana`) is NOT supported.

## HTTPS/TLS Configuration

### Current Implementation

Self-signed certificates are used for all HTTPS traffic. Let's Encrypt integration is deferred to future work.

**Self-Signed Certificate**:
```yaml
# traefik.yml
tls:
  certificates:
    - certFile: /certs/halos.local.crt
      keyFile: /certs/halos.local.key
```

### Certificate Management

**Self-Signed Generation** (on first boot):
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /certs/halos.local.key \
  -out /certs/halos.local.crt \
  -subj "/CN=halos.local" \
  -addext "subjectAltName=DNS:halos.local,DNS:*.halos.local"
```

The wildcard SAN (`*.halos.local`) enables the certificate to cover all subdomain URLs.

### Future: Let's Encrypt Support

Let's Encrypt integration remains a valid future enhancement for users with public domain names:
- Requires public DNS and port 80/443 accessible from internet
- Not applicable for `.local` mDNS domains
- Implementation deferred until there's user demand

## Integration Points

### With Homarr Dashboard

Homarr links always use subdomain URLs via the `homarr-container-adapter`:

```yaml
labels:
  - "homarr.enable=true"
  - "homarr.name=Grafana"
  - "homarr.url=https://grafana.${HALOS_DOMAIN}"
  - "homarr.category=Monitoring"
```

Direct port URLs are NOT shown in Homarr. Users access all apps through their subdomain URLs.

### With Container Apps

**Standard Apps** (Traefik-only):
```yaml
# NO ports section - access via Traefik subdomain only
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myapp.rule=Host(`myapp.${HALOS_DOMAIN}`)"
  # ... Traefik configuration
networks:
  - halos-proxy-network
```

**Host Networking Apps** (e.g., Signal K):
```yaml
network_mode: host
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.signalk.rule=Host(`signalk.${HALOS_DOMAIN}`)"
  - "traefik.http.services.signalk.loadbalancer.server.port=3000"
  # Direct port access remains available as side effect of host networking
```

**Apps Requiring Direct Ports** (non-HTTP protocols):
```yaml
ports:
  - "34567:34567"  # NMEA TCP stream - required for external tools
labels:
  - "traefik.enable=true"
  # HTTP UI still goes through Traefik
networks:
  - halos-proxy-network
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

### Default: Traefik-Only (No Direct Ports)

Apps are accessed exclusively via Traefik subdomain URLs. Direct port exposure is removed.

**Benefits**:
- Consistent user experience (all apps via subdomains)
- Centralized authentication via Authelia
- No port conflicts between apps
- Cleaner security model (only ports 80/443 exposed)

**Docker Compose Example**:
```yaml
services:
  grafana:
    # NO ports section
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.grafana.rule=Host(`grafana.${HALOS_DOMAIN}`)"
      # ... Traefik configuration
    networks:
      - halos-proxy-network
```

### Exception: Apps Requiring Direct Ports

Some apps legitimately need direct port access:

| App | Port | Reason |
|-----|------|--------|
| Signal K | 3000 | Host networking for hardware access |
| AvNav | 34567 | NMEA TCP stream for external tools |
| AvNav | 8083 | WebSocket for chart plotters |

These apps still get Traefik routing for their HTTP UI, but also expose necessary ports.

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

## Resolved Design Decisions

Previously open questions that have been resolved:

1. **URL Scheme**: Subdomain-based only (e.g., `grafana.halos.local`)
2. **Direct Port Fallback**: No - apps should not expose ports unless required
3. **metadata.yaml Schema**: Uses `routing:` key (see SSO_SPEC.md)
4. **Default HTTPS**: Yes, always HTTPS with self-signed certificates
5. **Let's Encrypt**: Deferred to future work

## Implementation Status

### Completed
- Traefik container with subdomain routing
- Authelia SSO integration
- mDNS subdomain advertisement
- Homarr behind Traefik (root domain)
- Authelia behind Traefik (auth subdomain)

### In Progress
- Marine container apps (Grafana, InfluxDB, Signal K, AvNav, OpenCPN)
- Cockpit web console

### Not Started
- Let's Encrypt support (future)

## References

### Internal Documentation
- [SSO_SPEC.md](../halos-core-containers/docs/SSO_SPEC.md) - SSO technical specification
- [SSO_ARCHITECTURE.md](../halos-core-containers/docs/SSO_ARCHITECTURE.md) - SSO architecture details
- [HOMARR_INTEGRATION_DESIGN.md](./HOMARR_INTEGRATION_DESIGN.md) - Dashboard integration
- [HOSTNAME_POLICY.md](./HOSTNAME_POLICY.md) - Policy on hostname references

### External References
- [Traefik Documentation](https://doc.traefik.io/traefik/) - Official docs
- [Traefik Docker Provider](https://doc.traefik.io/traefik/providers/docker/) - Docker label configuration
- [Authelia Documentation](https://www.authelia.com/configuration/) - SSO provider
