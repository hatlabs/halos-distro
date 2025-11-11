# Archived Documentation

This directory contains historical planning documents that have been superseded by newer designs.

## CONTAINER_STORE_PLAN.md

**Date**: November 6, 2025
**Status**: Superseded by META-PLANNING.md and ADR-001

This was an initial comprehensive 18-week implementation plan for the container store architecture. While the document contained many valuable ideas, the architecture has since been refined and documented in:

- [META-PLANNING.md](../../META-PLANNING.md) - Workspace-level coordination and vision
- [cockpit-apt/docs/ADR-001-container-store.md](../../cockpit-apt/docs/ADR-001-container-store.md) - Architecture decision record

**Key differences from current design**:
- Referenced "cockpit-package-manager" (now cockpit-apt)
- Used `/opt/container-apps/` paths (now `/var/lib/container-apps/`)
- Custom metadata format (now using AppStream and debtags)
- 18-week timeline (now using phased, incremental approach)

**Why archived**: The core vision remains the same, but implementation details and architecture have evolved based on further design work and alignment with Debian standards.

**Historical value**: Contains detailed examples of metadata schemas, package structure, and converter designs that may still inform future work.
