# Container Configuration Tool - Design Concept

**Status**: Concept (Phase 2)
**Date**: 2025-11-11
**Last Updated**: 2025-11-11

## Overview

This document describes the high-level concept for `cockpit-container-config`, a separate Cockpit module for managing and configuring installed container applications. This is a Phase 2 feature and detailed implementation planning will occur after Phase 1 (Container Store Foundation) is complete.

## Purpose

- Provide web-based configuration UI for installed container apps
- Enable non-technical users to configure apps without editing files
- Integrate service management (start/stop/restart) via systemd
- Display logs and status information
- Link to application web interfaces

## Design Principles

1. **Separate Module**: Independent Cockpit module, not part of cockpit-apt
2. **User-Friendly**: GUI for configuration, no command line required
3. **Safe Defaults**: Preserve working configurations, validate changes
4. **systemd Integration**: Use systemd for all service management
5. **Read-Only Where Possible**: Show information, edit only when necessary

## Module Concept

### Type

Standalone Cockpit module that appears in the left sidebar navigation, similar to:
- Services
- Logs
- Storage
- Terminal

### Name

**Display Name**: "Container Apps"
**Module ID**: `cockpit-container-config`
**Navigation**: Appears between "Software" and "Services" in sidebar

## Key Features

### 1. Application List View

**Purpose**: Overview of all installed container applications

**Display**:
- Card or table layout showing installed container apps
- App icon, name, description
- Current status (running, stopped, failed)
- Quick actions (start/stop/restart)
- Link to web UI (if app provides one)

**Information Shown**:
- Application name and version
- Status indicator (green/red dot)
- Resource usage (CPU, memory) - optional
- Uptime (how long running)
- Port mappings (which ports are exposed)

**Example Layout**:
```
┌────────────────────────────────────────────────────────┐
│ Container Applications                                  │
├────────────────────────────────────────────────────────┤
│ ┌──────────────────────────────────────────────────┐   │
│ │ ⬢ Signal K Server                       ● Running│   │
│ │   Marine data processing and routing              │   │
│ │   Port: 3000 | Uptime: 3 days                    │   │
│ │   [Stop] [Restart] [Configure] [Open Web UI]     │   │
│ └──────────────────────────────────────────────────┘   │
│                                                         │
│ ┌──────────────────────────────────────────────────┐   │
│ │ ⬢ OpenCPN                              ○ Stopped │   │
│ │   Chart plotter and navigation software           │   │
│ │   Port: 8080                                      │   │
│ │   [Start] [Configure]                             │   │
│ └──────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────┘
```

### 2. Configuration Editor

**Purpose**: User-friendly interface for editing app configuration

**Approach Options** (to be decided):

**Option A: Auto-Generated Forms**
- Parse `config.yml` schema
- Generate form fields automatically
- Pros: Consistent, validated, type-safe
- Cons: Complex to implement, limited flexibility

**Option B: Simple Text Editor**
- Edit `config.yml` directly with syntax highlighting
- Show validation errors
- Pros: Simple, flexible, works for all apps
- Cons: Less user-friendly for non-technical users

**Option C: Hybrid**
- Common settings as form fields (ports, paths)
- Advanced settings as YAML editor
- Pros: Balance of usability and flexibility
- Cons: Most complex to implement

**Configuration Workflow**:
1. User clicks "Configure" on app
2. System loads current configuration
3. User edits settings
4. System validates changes
5. On save:
   - Write new configuration to `/etc/container-apps/<app>/.env`
   - Offer to restart service to apply changes
   - Show success/error message

**Safety Features**:
- Backup previous configuration
- Validate before saving
- Option to "Reset to Defaults"
- Confirmation before restarting services

### 3. Service Control

**Purpose**: Start, stop, and restart container applications

**Integration**: Direct systemd control via Cockpit APIs

**Actions**:
- **Start**: `systemctl start <app>-container.service`
- **Stop**: `systemctl stop <app>-container.service`
- **Restart**: `systemctl restart <app>-container.service`
- **Enable**: Auto-start on boot
- **Disable**: Don't auto-start on boot

**UI Patterns**:
- Large buttons for primary actions (PatternFly)
- Confirmation dialogs for destructive actions
- Progress indicators for operations
- Toast notifications for success/failure

**Example**:
```
[Start]  [Stop]  [Restart]
☑ Start automatically on boot
```

### 4. Log Viewer

**Purpose**: Display application logs for troubleshooting

**Integration**: Read from journalctl via Cockpit APIs

**Features**:
- Real-time log streaming (tail -f behavior)
- Filter by log level (error, warning, info, debug)
- Search/filter by text
- Download logs
- Clear display (but keep logs in journal)

**UI Layout**:
```
┌────────────────────────────────────────────────────────┐
│ Signal K Server - Logs                                  │
├────────────────────────────────────────────────────────┤
│ [All Levels ▼] [Search...] [Download] [Clear Display]  │
├────────────────────────────────────────────────────────┤
│ 2025-11-11 14:23:45  INFO   Server started on port 3000│
│ 2025-11-11 14:23:46  INFO   Connected to NMEA source   │
│ 2025-11-11 14:25:12  WARN   GPS fix lost               │
│ 2025-11-11 14:25:15  INFO   GPS fix acquired           │
│ ...                                                     │
└────────────────────────────────────────────────────────┘
```

**Implementation**:
```bash
# Real-time logs for specific service
journalctl -u signalk-server-container.service -f

# Last 100 lines
journalctl -u signalk-server-container.service -n 100

# Errors only
journalctl -u signalk-server-container.service -p err
```

### 5. Application Details

**Purpose**: Show comprehensive information about an app

**Information Displayed**:
- Application metadata (from `metadata.json`)
  - Name, version, description
  - Homepage, license
  - Maintainer
- Installation details
  - Install date
  - Package version
  - Data directory location
  - Configuration file location
- Runtime information
  - Container ID and name
  - Image used
  - Network mode
  - Volume mounts
  - Environment variables
- Web UI link (if available)
  - Open in new tab
  - Show connection info (URL, port)

### 6. Volume Mount Management (Future)

**Note**: This is marked as optional/future enhancement

**Purpose**: Allow users to change volume mount locations

**Use Case**: Moving data to external storage (USB, SD card)

**Complexity**: High - requires:
- Stopping container
- Copying data safely
- Updating compose file
- Restarting container
- Rollback on failure

**Decision**: Defer to Phase 3 or beyond. Power users can edit compose files manually.

## User Workflows

### Install and Configure New App

1. User installs app via cockpit-apt (Phase 1)
2. App appears in Container Apps module
3. User clicks "Configure"
4. Edits settings (e.g., change port from 3000 to 8080)
5. Clicks "Save and Restart"
6. App restarts with new configuration
7. User clicks "Open Web UI" to access app

### Troubleshoot Failing App

1. User notices app status is "Failed"
2. Opens Container Apps module
3. Clicks on failed app
4. Views logs to see error messages
5. Identifies configuration issue
6. Edits configuration
7. Restarts app
8. Verifies app is now running

### Change Port After Installation

1. User has port conflict (app using port 3000, need it for something else)
2. Opens Container Apps module
3. Clicks "Configure" on app
4. Changes HTTP_PORT from 3000 to 8080
5. Saves configuration
6. Restarts app
7. App now accessible on port 8080

## Technical Architecture (High-Level)

### Module Structure

```
cockpit-container-config/
├── src/
│   ├── components/
│   │   ├── AppList.tsx
│   │   ├── AppDetails.tsx
│   │   ├── ConfigEditor.tsx
│   │   ├── ServiceControl.tsx
│   │   └── LogViewer.tsx
│   ├── hooks/
│   │   ├── useApps.ts
│   │   ├── useServiceControl.ts
│   │   └── useLogs.ts
│   ├── services/
│   │   ├── apps.ts         # Load app metadata
│   │   ├── systemd.ts      # systemd integration
│   │   └── config.ts       # Config reading/writing
│   └── pages/
│       ├── AppsPage.tsx
│       └── AppDetailsPage.tsx
├── pkg/
│   └── manifest.json        # Cockpit module manifest
└── README.md
```

### Backend Integration

**Cockpit APIs Used**:
- `cockpit.spawn()` - Execute commands (systemctl, journalctl)
- `cockpit.file()` - Read/write configuration files
- `cockpit.dbus()` - systemd D-Bus interface (alternative to spawn)

**Example API Calls**:
```typescript
// Start service
await cockpit.spawn(['systemctl', 'start', 'signalk-server-container.service']);

// Read logs
const logs = await cockpit.spawn(['journalctl', '-u', service, '-n', '100']);

// Read config
const config = await cockpit.file('/etc/container-apps/signalk-server-container/.env').read();

// Write config
await cockpit.file('/etc/container-apps/signalk-server-container/.env').replace(newConfig);
```

### Data Sources

**Application List**: Scan `/var/lib/container-apps/*/metadata.json`

**Service Status**: Query systemd via D-Bus or `systemctl status`

**Configuration**: Read from `/etc/container-apps/<app>/.env` and `config.yml`

**Logs**: Stream from `journalctl -u <service> -f`

## UI/UX Design

### Technology Stack

- **Framework**: React + TypeScript (matching Cockpit standards)
- **UI Library**: PatternFly 4 (Cockpit's design system)
- **State Management**: React Context + hooks
- **Build System**: Webpack (Cockpit standard)

### Key PatternFly Components

- **AppList**: `Gallery` with `Card` components
- **ServiceControl**: `Button`, `Switch` for enable/disable
- **ConfigEditor**: `Form`, `FormGroup`, `TextInput`, `TextArea`
- **LogViewer**: `TextContent` with monospace font, `Toolbar` for controls
- **Status Indicators**: `Label` with colors (green/red/yellow)

### Responsive Design

- Desktop: Two-column layout (list + details)
- Tablet: Single column, navigate between views
- Mobile: Optimized touch targets, simplified layout

## Security Considerations

1. **File Permissions**: Ensure only root can write config files
2. **Validation**: Validate all user input before writing to disk
3. **Service Control**: Use systemd's built-in permission system
4. **Secrets**: Mask password fields in UI, don't log them
5. **Audit**: Log all configuration changes to system log

## Testing Strategy

### Unit Tests
- Component rendering
- Configuration parsing/serialization
- Validation logic

### Integration Tests
- Service start/stop/restart
- Configuration save and reload
- Log streaming

### E2E Tests
- Complete user workflows (install, configure, restart)
- Error handling (service fails to start, invalid config)

## Open Questions (To Resolve in Phase 2 Planning)

1. **Configuration UI Approach**: Auto-generated forms vs. text editor vs. hybrid?
2. **Volume Management**: Include in Phase 2 or defer to Phase 3?
3. **Multi-Container Apps**: How to handle apps with multiple containers?
4. **Docker vs. Podman**: Support both container runtimes?
5. **Compose File Editing**: Should users be able to edit compose files via UI?
6. **Resource Limits**: Should users be able to set CPU/memory limits?
7. **Backup/Restore**: Built-in backup of app data and configuration?

## Dependencies

### Requires Phase 1 Complete

This module depends on Phase 1 infrastructure:
- Container apps packaged as .deb files
- Apps installed in `/var/lib/container-apps/`
- Configuration in `/etc/container-apps/`
- systemd services managing containers
- metadata.json format standardized

### No Dependency on Stores

This module works independently of the store system. It manages ALL installed container apps, regardless of which store they came from.

## Implementation Timeline

**Phase 2 Planning** (before implementation):
1. Resolve open questions
2. Create detailed SPEC.md (what it does)
3. Create detailed ARCHITECTURE.md (how it works)
4. Create TASKS.md (implementation breakdown)
5. Review with stakeholders

**Phase 2 Implementation** (after Phase 1 complete):
1. Set up module skeleton
2. Implement app list view
3. Implement service control
4. Implement log viewer
5. Implement configuration editor
6. Testing and refinement
7. Documentation

**Estimated Duration**: 4-6 weeks after Phase 1 completion

## Success Criteria

Users can:
- ✓ See all installed container apps in one place
- ✓ Start/stop/restart apps without command line
- ✓ View real-time logs for troubleshooting
- ✓ Configure apps via web interface
- ✓ Access app web UIs with one click
- ✓ Enable/disable auto-start on boot

## References

### Internal Documentation
- [META-PLANNING.md](../META-PLANNING.md) - Overall project planning
- [halos-marine-containers/docs/DESIGN.md](../halos-marine-containers/docs/DESIGN.md) - App definitions
- [cockpit-apt/docs/CONTAINER_STORE_DESIGN.md](../cockpit-apt/docs/CONTAINER_STORE_DESIGN.md) - Store UI

### External References
- [Cockpit Developer Guide](https://cockpit-project.org/guide/latest/) - Module development
- [PatternFly](https://www.patternfly.org/) - UI components
- [systemd](https://www.freedesktop.org/wiki/Software/systemd/) - Service management
- [journalctl](https://www.freedesktop.org/software/systemd/man/journalctl.html) - Log management
- [Docker Compose](https://docs.docker.com/compose/) - Container orchestration
