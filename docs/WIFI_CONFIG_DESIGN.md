# WiFi Configuration Tool - Design Concept

**Status**: Concept (Phase 3)
**Date**: 2025-11-11
**Last Updated**: 2025-11-11

## Overview

This document describes the high-level concept for enhanced WiFi configuration in Cockpit. This is a Phase 3 feature and will be planned in detail only after Phase 1 and Phase 2 are complete.

## Problem Statement

### Current State

**Upstream Cockpit Network Module**:
- Provides basic wired network configuration
- Limited WiFi support (can see networks, basic connect)
- Missing features common in desktop WiFi managers:
  - QR code generation for guest access
  - Saved network management
  - Signal strength indicators
  - Advanced WiFi settings (channel, hidden networks, etc.)
  - Access Point (hotspot) mode configuration

**Raspberry Pi OS Desktop**:
- Has comprehensive WiFi GUI (NetworkManager applet)
- Supports all common WiFi scenarios
- User-friendly for non-technical users

**Gap**: HaLOS needs similar WiFi capabilities but accessible via web interface for headless operation.

## Use Cases

### Marine/Boat Computing Context

1. **Access Point Mode** (Most Important)
   - Boat has no internet, Raspberry Pi creates WiFi network
   - Crew devices connect to boat's WiFi to access HaLOS
   - Need easy setup and QR code for guests

2. **Station Mode** (Secondary)
   - Connect to marina WiFi for internet access
   - Manage multiple saved networks (different marinas)
   - Auto-connect to known networks

3. **Bridge Mode** (Advanced)
   - Connect to marina WiFi (station)
   - Share internet to boat network (access point)
   - Simultaneous AP and station mode

4. **Mobile Hotspot** (Common)
   - Connect to phone hotspot for cellular internet
   - Quick setup for temporary connections

## Core Features (Priority Order)

### 1. Access Point (Hotspot) Setup ⭐⭐⭐

**Priority**: Critical - Primary use case for marine computing

**User Story**: "As a boat owner, I want to create a WiFi network so crew can access navigation systems from their devices."

**Features**:
- One-click AP setup with sensible defaults
- Configure SSID (network name)
- Set password (WPA2/WPA3)
- Choose WiFi channel
- Generate QR code for easy connection
- Show number of connected clients
- Configure IP range and DHCP

**UI Concept**:
```
┌─────────────────────────────────────────────────────┐
│ Access Point (Hotspot) Mode                         │
├─────────────────────────────────────────────────────┤
│ Status: Active ● | 3 devices connected              │
│                                                      │
│ Network Name (SSID):  [Boat WiFi            ]       │
│ Password:             [●●●●●●●●●●●●] [Show]          │
│ Channel:              [Auto ▼]                      │
│ Security:             [WPA2-PSK ▼]                  │
│                                                      │
│ [QR Code for Guest Access]                          │
│ ┌───────────┐                                       │
│ │ █ ▄▀▄ █ █ │  ← Scan to connect                   │
│ │ █ █▄▄ █▄█ │                                       │
│ └───────────┘                                       │
│                                                      │
│ Connected Devices:                                  │
│ • Phone-123 (192.168.4.101)                        │
│ • Tablet-456 (192.168.4.102)                       │
│ • Laptop-789 (192.168.4.103)                       │
│                                                      │
│ [Start AP] [Stop AP] [Advanced Settings...]         │
└─────────────────────────────────────────────────────┘
```

### 2. WiFi Network Management ⭐⭐

**Priority**: High - Needed for marina internet access

**User Story**: "As a boat owner in a marina, I want to connect to marina WiFi for internet access."

**Features**:
- Scan for available networks
- Connect to open/secured networks
- Save network credentials
- Auto-reconnect to known networks
- Forget network
- Network priority (which to prefer)
- Show signal strength (bars/percentage)
- Show connection speed and quality

**UI Concept**:
```
┌─────────────────────────────────────────────────────┐
│ WiFi Networks                                        │
├─────────────────────────────────────────────────────┤
│ [Scan for Networks] [Add Hidden Network...]         │
│                                                      │
│ Connected:                                           │
│ ✓ Marina Guest WiFi  ▂▃▅▇█  [Disconnect]           │
│                                                      │
│ Available Networks:                                  │
│   Boat-Next-Door    ▂▃▅▇▆  🔒 [Connect]            │
│   Harbor Master     ▂▃▅▇▃  🔒 [Connect]            │
│   Marina Public     ▂▃▄▅▃  🔓 [Connect]            │
│                                                      │
│ Saved Networks:                                      │
│   Home WiFi         Not in range                    │
│   Friend's Boat     Not in range                    │
│   [Add Network...]                                   │
└─────────────────────────────────────────────────────┘
```

### 3. Bridge/Router Mode ⭐

**Priority**: Medium - Advanced use case

**User Story**: "As a boat owner, I want to connect to marina WiFi and share it with my boat network."

**Features**:
- Simultaneous AP and station mode
- Internet sharing (NAT)
- Firewall configuration
- Port forwarding (for external access to services)
- Bandwidth monitoring

### 4. Network Diagnostics ⭐

**Priority**: Medium - Useful for troubleshooting

**Features**:
- Connection status and details
- Signal strength over time (graph)
- Speed test (upload/download)
- Ping test to verify connectivity
- DNS resolution test
- Show current IP, gateway, DNS servers

## Architecture Approaches

### Option A: Extend Upstream Cockpit Network Module

**Approach**: Contribute WiFi enhancements to upstream Cockpit project

**Pros**:
- Benefits entire Cockpit community
- Maintains compatibility with upstream
- Easier long-term maintenance

**Cons**:
- Longer development cycle (upstream review)
- Must meet upstream standards and use cases
- May not support HaLOS-specific features

**Feasibility**: High - Cockpit project accepts contributions

### Option B: Fork Cockpit Network Module

**Approach**: Create `cockpit-network-extended` with WiFi features

**Pros**:
- Full control over features and UI
- Faster development (no upstream review)
- Can include HaLOS-specific features

**Cons**:
- Maintenance burden (track upstream changes)
- Risk of divergence from upstream
- Duplicate code

**Feasibility**: Medium - Requires ongoing sync effort

### Option C: Separate WiFi Module

**Approach**: Create `cockpit-wifi` as standalone module

**Pros**:
- Focused scope (WiFi only)
- No conflict with upstream network module
- Can coexist with upstream module

**Cons**:
- Duplicate UI (two network modules)
- Potential confusion for users
- May require coordination between modules

**Feasibility**: High - Clean separation

### Option D: NetworkManager GUI Wrapper

**Approach**: Web-based wrapper around existing NetworkManager tools

**Pros**:
- Reuse proven NetworkManager backend
- Leverage existing WiFi tools (nmcli, nmtui)
- Compatible with Raspberry Pi OS tools

**Cons**:
- Limited by NetworkManager capabilities
- May feel like a thin wrapper
- Less polished UI

**Feasibility**: High - NetworkManager is well-documented

## Technical Considerations

### Backend Technology

**NetworkManager** (Recommended):
- Standard Linux network management
- Comprehensive D-Bus API
- Used by Raspberry Pi OS desktop
- Well-documented and maintained

**wpa_supplicant** (Lower-level):
- Direct WiFi control
- More complex to use
- Better for custom scenarios
- Requires more code

**systemd-networkd** (Alternative):
- Lightweight alternative to NetworkManager
- May lack some WiFi features
- Good for simple scenarios

### Frontend Technology

**Consistency with Cockpit**:
- React + TypeScript
- PatternFly components
- Cockpit APIs for backend communication

**WiFi-Specific UI Elements**:
- Signal strength indicators (PatternFly `Level` or custom)
- QR code generation (qrcode.react library)
- Network scanning animation
- Connection status indicators

### Security Considerations

1. **Password Storage**: Use NetworkManager's secure storage (not plain text)
2. **QR Codes**: Generate client-side, don't log passwords
3. **Access Control**: Require Cockpit authentication for WiFi changes
4. **WPA3 Support**: Use latest security standards
5. **Hidden Network Support**: For security-conscious users

## Integration Points

### With Existing Cockpit

- Cockpit Network module (coordination)
- Cockpit System module (firewall for AP mode)
- Cockpit dashboard (show WiFi status)

### With HaLOS

- Integration with halos-pi-gen for default WiFi setup
- Default AP configuration for first boot
- QR code printed in documentation

### With Container Apps

- WiFi status visible to apps (Signal K, OpenCPN)
- Network change notifications
- Internet connectivity status for apps that need it

## User Workflows

### First-Time Setup: Create Boat WiFi

1. User boots HaLOS for first time (no WiFi configured)
2. System creates default AP "HaLOS-XXXX" with random password
3. User connects laptop to HaLOS AP
4. Opens Cockpit at default IP (192.168.4.1:9090)
5. Goes to WiFi configuration
6. Changes SSID to "Boat Name WiFi"
7. Sets custom password
8. Generates QR code and prints/saves it
9. Crew members scan QR code to connect

### Marina: Connect to Guest WiFi

1. User arrives at marina
2. Opens Cockpit on tablet (already connected to boat WiFi)
3. Goes to WiFi configuration
4. Scans for networks
5. Sees "Marina Guest WiFi"
6. Clicks "Connect"
7. Enters password (if required)
8. Connection established
9. Internet now available to all boat devices

### Advanced: Bridge Marina WiFi to Boat Network

1. User wants internet from marina on all boat devices
2. Opens Cockpit WiFi configuration
3. Switches to "Bridge Mode"
4. Selects marina WiFi as upstream (WAN)
5. Keeps boat AP as downstream (LAN)
6. Enables internet sharing
7. All boat devices now have internet through marina WiFi

## Open Questions (To Resolve in Phase 3 Planning)

1. **Architecture**: Which approach (A/B/C/D) to pursue?
2. **Upstream**: Can we contribute WiFi features to Cockpit upstream?
3. **Feature Scope**: Which features are MVP vs. future?
4. **Multi-Interface**: Support multiple WiFi adapters (USB dongles)?
5. **Cellular**: Should cellular modems be in scope (4G/5G)?
6. **VPN**: Should VPN configuration be included?
7. **Captive Portals**: How to handle marina WiFi login pages?
8. **Mesh Networking**: Support for WiFi mesh (multiple boats)?

## Dependencies

### Requires Phase 1 & 2 Complete

Not strictly dependent, but benefits from:
- User familiarity with Cockpit interface
- Established Cockpit module patterns
- User base for testing and feedback

### System Requirements

- NetworkManager installed and running
- WiFi hardware (onboard or USB adapter)
- Raspberry Pi OS or compatible Linux distribution

## Implementation Timeline

**Phase 3 Planning** (after Phase 2 complete):
1. Research upstream Cockpit network module status
2. Evaluate architecture options (A/B/C/D)
3. Prototype core features (AP setup, network scanning)
4. Get community feedback
5. Create detailed specification

**Phase 3 Implementation**:
1. MVP: Access Point setup
2. MVP: Network scanning and connection
3. Enhanced: Saved networks and auto-connect
4. Enhanced: Bridge mode
5. Polish: Diagnostics and monitoring
6. Documentation and testing

**Estimated Duration**: 6-8 weeks after Phase 2 completion

## Success Criteria

Users can:
- ✓ Set up boat WiFi access point in <5 minutes
- ✓ Generate QR code for crew to connect
- ✓ See who's connected to boat WiFi
- ✓ Connect to marina WiFi when available
- ✓ Manage multiple saved networks
- ✓ Bridge marina WiFi to boat network
- ✓ Diagnose connection issues

## References

### Similar Projects

- [RaspAP](https://raspap.com/) - Raspberry Pi Access Point management
- [OpenWrt LuCI](https://openwrt.org/docs/guide-user/luci/start) - Router web interface
- [DD-WRT](https://dd-wrt.com/) - Router firmware with web UI

### Technical References

- [NetworkManager Documentation](https://networkmanager.dev/)
- [NetworkManager D-Bus API](https://networkmanager.dev/docs/api/latest/)
- [Cockpit Network Module](https://github.com/cockpit-project/cockpit/tree/main/pkg/networkmanager)
- [WiFi QR Code Format](https://github.com/zxing/zxing/wiki/Barcode-Contents#wi-fi-network-config-android-ios-11)
- [WPA Supplicant](https://w1.fi/wpa_supplicant/)

### Internal Documentation

- [META-PLANNING.md](../META-PLANNING.md) - Overall project planning
- [CONTAINER_CONFIG_DESIGN.md](./CONTAINER_CONFIG_DESIGN.md) - Phase 2 design

### Community Resources

- [Raspberry Pi Forums - WiFi](https://forums.raspberrypi.com/viewforum.php?f=36)
- [Cockpit Project - Networking](https://cockpit-project.org/)
- [Marine IoT Connectivity Best Practices](https://community.marinenetworks.ca/)
