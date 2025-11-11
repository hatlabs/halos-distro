# Open Questions: Homarr Dashboard & Traefik Integration

**Status**: Awaiting answers
**Date**: 2025-11-11

## Background

Two important components were identified as missing from the current architecture:
1. **Homarr Dashboard** - Landing page for accessing container apps
2. **Traefik Reverse Proxy** - Unified URL scheme instead of random ports

This document captures open questions to finalize the design and integration approach.

---

## Questions about Homarr Dashboard

### 1. Phase Placement
**Question**: When should Homarr integration be implemented?

Options:
- [ ] Phase 1 (launch alongside container store)
- [ ] Phase 1.5 (after basic container apps work, add dashboard)
- [ ] Phase 2 (alongside container config UI)

**Your Answer**:

Phase 3.5 - once we have the WiFi configuration and a good number of apps in the store, so the dashboard has more value right away.

---

### 2. Homarr Adapter Architecture
**Question**: How should the adapter detect and register newly installed apps?

Options:
- [x] systemd service watching `/var/lib/container-apps/` (inotify)
- [ ] Triggered by postinst scripts in each container app package
- [ ] Periodic sync service (check every N seconds for new apps)
- [ ] Other (describe):

**Your Answer**:

The running container apps could provide the required data as labels in their docker-compose.yml files, and the adapter could periodically scan the running containers to update the Homarr configuration. This avoids the need for inotify or postinst scripts, simplifying the architecture.

---

### 3. Homarr Packaging
**Question**: How should Homarr be packaged and distributed?

- Should Homarr itself be packaged as `homarr-container`? YES / NO
- Should the adapter be separate `homarr-adapter`? YES / NO
- Or bundled together? YES / NO

**Your Answer**:

Yes, Homarr should be packaged as `homarr-container` to allow for easy updates and management. However, the adapter could be a separate native Debian package (`homarr-container-adapter`). To minimize the memory footprint, it could be written in Rust.

---

### 4. Dashboard Metadata
**Question**: Does the current `metadata.json` format have enough information?

Current fields available:
- `name`, `description`, `icon`
- `web_ui: {enabled, path, port, protocol}`
- `tags` (debtags like `field::marine`)

Questions:
- Is this sufficient for dashboard presentation? YES / NO
- Do we need additional fields? If yes, what? (e.g., `dashboard_category`, `dashboard_order`, `featured`)

**Your Answer**:

I don't think we need additional fields right now.

---

### 5. User Configuration
**Question**: Should users be able to customize the dashboard?

- Add/remove apps from dashboard? YES / NO
- Reorder apps? YES / NO
- Customize app icons/names? YES / NO
- Purely auto-generated (no user customization)? YES / NO

**Your Answer**:

Yes, the dashboard should be highly customizable. The adapter should probably track the state of apps removed from the dashboard to avoid re-adding them automatically.

---

## Questions about Traefik Reverse Proxy

### 6. Phase Placement
**Question**: When should Traefik integration be implemented?

Options:
- [ ] Phase 1 (launch alongside container store)
- [ ] Phase 1.5 (after basic container apps work)
- [ ] Phase 2 (alongside container config UI)
- [ ] Same phase as Homarr (whatever that is)

**Your Answer**:

After Homarr.

---

### 7. URL Scheme
**Question**: What URL pattern should be used for accessing apps?

Options:
- [ ] Path-based: `halos.local/signalk`, `halos.local/grafana`
- [ ] Subdomain-based: `signalk.halos.local`, `grafana.halos.local`
- [ ] Both with user preference
- [ ] Other (describe):

**Your Answer**:

Remains TBD.

---

### 8. HTTPS/TLS
**Question**: How should HTTPS be handled?

Options:
- [ ] HTTP-only (simplicity for local network)
- [ ] Self-signed certificates (always HTTPS, ignore browser warnings)
- [ ] Let's Encrypt (if domain/external access configured)
- [ ] User choice (HTTP by default, HTTPS optional)

**Your Answer**:

Self-signed or Let's Encrypt depending on whether the user has a domain configured.

---

### 9. Traefik Packaging
**Question**: How should Traefik be packaged?

- Package as `traefik-container`? YES / NO
- Separate systemd service? YES / NO
- Auto-configured or requires user setup?
- Should it be a dependency of `marine-container-store` (or other stores)?

**Your Answer**:

Probably something like `halos-traefik-container` because the configuration will be custom to HaLOS. Preferably auto-configured using labels. I believe that's how Runtipi does it.

Not sure about the dependency question. TBD.

---

### 10. Container App Integration
**Question**: Should Traefik be mandatory or optional?

- ALL container apps MUST go through Traefik? YES / NO
- Optional with direct port access as fallback? YES / NO
- Per-app configuration (some via Traefik, some direct)? YES / NO

**Your Answer**:

Optional with direct port access as fallback.

---

### 11. Port Allocation Strategy
**Question**: How should ports be handled with Traefik?

Options:
- [ ] Apps only accessible via Traefik (no direct port exposure)
- [ ] Both Traefik route AND direct port (for flexibility)
- [ ] User choice per app
- [ ] Other (describe):

**Your Answer**:

User choice per app.

---

### 12. Traefik Configuration Method
**Question**: How should routing be configured?

Options:
- [ ] Traefik labels in `docker-compose.yml` (Runtipi style)
- [ ] Separate Traefik config files per app
- [ ] Generated by adapter service
- [ ] Other (describe):

**Your Answer**:

Traefik labels in `docker-compose.yml` (Runtipi style) seems simplest.

---

## Impact Assessment

### Changes to Architecture

Based on your answers, these components may need updates:

**container-packaging-tools**:
- Generate Traefik labels in docker-compose.yml? YES / NO / MAYBE
- Generate Homarr metadata? YES / NO / MAYBE

Yes to both.

**halos-marine-containers**:
- App definitions need Traefik routing info? YES / NO / MAYBE
- App definitions need dashboard categories? YES / NO / MAYBE

Yes, maybe. Categories are not super-important.

**cockpit-apt**:
- Show dashboard link in UI? YES / NO / MAYBE
- Link to apps via Traefik routes instead of ports? YES / NO / MAYBE

cockpit-apt should do neither. Maybe you confused it with the Config UI?

**metadata.json format**:
- Needs new fields for Traefik? If yes, which?
- Needs new fields for Homarr? If yes, which?

**Your Answers**:

Don't know. TBD.

---

## Additional Considerations

### 13. Runtipi Migration Path
**Question**: How does this relate to migrating away from Runtipi?

- Should we copy Runtipi's Traefik config approach exactly? YES / NO
- Should Homarr replace Runtipi's dashboard? YES / NO
- Any Runtipi features we should preserve?

**Your Answer**:

No migration is required. Runtipi will be removed as soon as we have a working alternative.

---

### 14. Default Landing Page
**Question**: What should users see when they visit `http://halos.local/`?

Options:
- [ ] Homarr dashboard
- [ ] Cockpit login
- [ ] Simple index page with links to Cockpit + Homarr
- [ ] Redirect to Homarr if installed, Cockpit otherwise
- [ ] Other (describe):

**Your Answer**:

Homarr dashboard. Cockpit can be accessed from there.

---

### 15. Dependencies
**Question**: Should Homarr and/or Traefik be:

- Pre-installed in HaLOS images? YES / NO
- Required dependencies of store packages (e.g., `marine-container-store`)? YES / NO
- Optional (user installs if wanted)? YES / NO
- Installed automatically when first container app is installed? YES / NO

**Your Answer**:

Both should be pre-installed.
Neither should be required dependencies. Maybe suggested. They shouldn't be automatically installed. We might provide a meta-package.

---

## Next Steps

Once questions are answered:
1. [x] Create `docs/HOMARR_INTEGRATION_DESIGN.md`
2. [x] Create `docs/TRAEFIK_INTEGRATION_DESIGN.md`
3. [x] Update META-PLANNING.md with new components
4. [x] Update phase definitions
5. [x] Create GitHub issues for tracking
6. [x] Update component DESIGN.md files as needed
7. [x] Adjust `metadata.json` schema if needed
8. [x] Update README.md roadmap

---

## Notes / Additional Thoughts

(Add any additional context, concerns, or ideas here)
