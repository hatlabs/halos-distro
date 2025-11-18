# Tag-Based Categorization Strategy

**Last Modified:** 2025-11-16

This document explains the tag-based categorization approach used in HaLOS package management for organizing packages in domain-specific stores.

## Overview

HaLOS package management uses a flexible tag-based categorization system that allows domain-specific stores (like "Marine Navigation & Monitoring") to organize packages in meaningful ways beyond traditional Debian sections.

### Why Not Use Debian Sections?

Traditional Debian sections (`web`, `net`, `python`, etc.) are designed for general-purpose Linux distributions and don't capture domain-specific organization needs. For example:

- A marine navigation application might be in section `net` or `misc`
- A marine monitoring dashboard might be in section `web`
- Marine chart plotters could be in `x11` or `graphics`

These section assignments don't help boaters find the software they need. They need categories like "Navigation & Charts", "Communication", and "Data & Monitoring".

### Tag-Based Approach

HaLOS solves this by:

1. **Packages** define semantic tags using faceted classification (`field::marine`, `category::navigation`)
2. **Stores** filter packages by domain tags (`field::marine`) and optionally enhance categories with metadata
3. **System** auto-discovers categories from filtered packages
4. **UI** displays categories for custom stores, traditional sections for "All Packages"

This approach is:
- **Self-documenting** - Tags live in package metadata
- **Flexible** - Packages can belong to multiple categories
- **Automatic** - New categories appear when first package uses them
- **Maintainable** - No configuration redundancy

## Terminology

Understanding the following terms is essential:

### Sections
**Debian sections** are the traditional package organization system defined in [Debian Policy](https://www.debian.org/doc/debian-policy/ch-archive.html#s-subsections). Examples: `web`, `net`, `python`, `devel`.

- Source: Package's `Section` field
- Managed by: Debian package maintainers
- Used in: "All Packages" view
- Immutable: Cannot be customized per store

### Categories
**Categories** are tag-based groups automatically discovered from package tags. Examples: `navigation`, `monitoring`, `communication`.

- Source: Package's `category::*` tags
- Managed by: Package maintainers (via metadata.yaml)
- Used in: Custom store views (when store has category_metadata)
- Flexible: Auto-discovered, multiple per package

### Tags (Debtags)
**Debtags** are faceted classification tags that describe package characteristics. Examples: `field::marine`, `role::container-app`, `category::navigation`.

- Format: `facet::value`
- Source: Package's `Tag` field
- Standard: [Debian Debtags](https://wiki.debian.org/Debtags)
- Extensible: Custom facets allowed

### Facets
**Facets** are classification dimensions in the debtag system. Each facet represents a different aspect of a package.

- Examples: `field::` (domain), `role::` (type), `interface::` (UI)
- Purpose: Organize tags into meaningful dimensions
- Usage: Enable multi-dimensional package classification

## Debtag Taxonomy

HaLOS uses a combination of standard Debian facets and custom facets.

### Standard Debian Facets

From the [Debian Debtags](https://wiki.debian.org/Debtags) taxonomy:

| Facet | Description | Examples |
|-------|-------------|----------|
| `field::` | Application domain or field | `field::marine`, `field::navigation`, `field::geography` |
| `role::` | Software type or role | `role::container-app`, `role::program` |
| `interface::` | User interface type | `interface::web`, `interface::cli` |
| `use::` | Primary use case | `use::routing`, `use::monitoring`, `use::organizing` |
| `works-with::` | Data types or formats handled | `works-with::maps`, `works-with::logfile`, `works-with::network-traffic` |
| `network::` | Network role | `network::server`, `network::client` |
| `scope::` | Application scope | `scope::application`, `scope::utility` |

### Custom HaLOS Facet

HaLOS introduces one custom facet for UI organization:

| Facet | Description | Examples |
|-------|-------------|----------|
| `category::` | User-facing category for store organization | `category::navigation`, `category::monitoring`, `category::communication` |

**Important:** The `category::` facet is specifically for organizing packages within custom stores. It represents how users think about finding applications, not technical characteristics (which are covered by standard facets).

### Facet Usage Guidelines

**For domain identification:**
```yaml
tags:
  - field::marine           # Broad domain
  - field::navigation       # Specific sub-domain
```

**For UI categorization:**
```yaml
tags:
  - category::navigation    # Where users look for this app
  - category::chartplotters # Also appears in specialized category
```

**For technical characteristics:**
```yaml
tags:
  - role::container-app     # It's a containerized application
  - interface::web          # Has a web UI
  - use::routing            # Used for routing/navigation
  - works-with::maps        # Works with map data
```

Packages should use tags from multiple facets to fully describe their characteristics.

## System Architecture

### Complete Flow

```
1. Package Definition (metadata.yaml)
   └─> tags: [field::marine, category::navigation]

2. Store Configuration (marine.yaml)
   └─> filters.include_tags: [field::marine]
   └─> category_metadata: { id: navigation, label: "Navigation & Charts", ... }

3. Backend: Store Filter
   └─> Filters packages matching field::marine tag

4. Backend: Category Discovery
   └─> Extracts category::* tags from filtered packages
   └─> Counts packages per category
   └─> Enhances with category_metadata (if provided)

5. Frontend: Display Logic
   └─> IF store has category_metadata: Show categories
   └─> ELSE: Show traditional sections

6. Frontend: Category Cards
   └─> Displays icon, label, count, description per category
```

### Store Filter Logic

Store filters use **OR logic** across filter types:

```yaml
filters:
  include_origins: ["Hat Labs"]        # OR
  include_sections: ["net", "web"]     # OR
  include_tags: ["field::marine"]      # OR
  include_packages: ["specific-pkg"]   # OR
```

A package matches the store filter if it satisfies **ANY** of these conditions.

Within a filter type, multiple values also use **OR** logic:
- `include_tags: [field::marine, field::aviation]` matches packages with field::marine **OR** field::aviation

### Category Auto-Discovery

The backend automatically discovers categories:

1. Iterate through all packages matching store filter
2. Extract `category::*` tags from each package
3. Count how many packages have each category tag
4. Enhance with metadata from store config (if provided)
5. Return sorted list of categories

**Without metadata:**
```json
{
  "id": "navigation",
  "label": "Navigation",      // Auto-derived from ID
  "icon": null,
  "description": null,
  "count": 2
}
```

**With metadata:**
```json
{
  "id": "navigation",
  "label": "Navigation & Charts",  // From store config
  "icon": "MapIcon",
  "description": "Chart plotters and navigation tools",
  "count": 2
}
```

## Package Maintainer Guide

### How to Tag Your Package

Edit `apps/your-app/metadata.yaml` and add appropriate tags:

```yaml
name: your-app
version: 1.0.0
# ... other fields ...

tags:
  # Domain identification (REQUIRED for store filtering)
  - field::marine

  # User-facing categories (REQUIRED for category organization)
  - category::navigation
  - category::chartplotters

  # Technical characteristics (RECOMMENDED for searchability)
  - role::container-app
  - interface::web
  - use::routing
  - works-with::maps
```

### Tagging Best Practices

**DO:**
- Use `field::` to identify the application domain
- Use `category::` for user-facing organization
- Use multiple `category::` tags if the app fits multiple categories
- Use standard Debian facets for technical characteristics
- Keep tags consistent across similar applications

**DON'T:**
- Don't invent new facets - use standard taxonomy
- Don't over-tag - be selective and meaningful
- Don't use `category::` for technical properties (use `field::`, `use::`, etc.)

### Example: Marine Chart Plotter

```yaml
# apps/opencpn/metadata.yaml
name: opencpn
version: 5.10.2
description: OpenCPN chart plotter and navigation software

tags:
  # Domain (enables filtering into marine store)
  - field::marine
  - field::navigation
  - field::geography

  # Categories (where users find this app)
  - category::navigation
  - category::chartplotters

  # Technical characteristics
  - role::container-app
  - scope::application
  - interface::web
  - use::routing
  - works-with::maps
```

### Example: Marine Monitoring System

```yaml
# apps/signalk-server/metadata.yaml
name: signalk-server
version: 2.8.0
description: Signal K marine data server

tags:
  # Domain
  - field::marine

  # Categories
  - category::communication
  - category::monitoring

  # Technical characteristics
  - role::container-app
  - interface::web
  - use::routing
  - network::server
```

### How New Categories Appear

When you add a new `category::*` tag to any package:

1. The category **automatically appears** in the store's category list
2. **No store configuration update needed**
3. Category gets auto-derived label (e.g., `category::safety` → "Safety")
4. Store maintainer can **optionally** add metadata later for better presentation

This means you can freely create new categories as needed without coordinating with store maintainers.

## Store Maintainer Guide

### Basic Store Configuration

Create a store configuration file in `halos-marine-containers/store/`:

```yaml
# store/marine.yaml
id: marine
name: Marine Navigation & Monitoring
description: |
  Applications for marine navigation, monitoring, and boat systems.

icon: /usr/share/container-stores/marine/icon.svg

filters:
  # Include all packages with marine-related tags
  include_tags:
    - field::marine
```

This minimal configuration:
- Filters packages by `field::marine` tag
- Auto-discovers all `category::*` tags from those packages
- Uses auto-derived labels for categories
- Works without any category_metadata

### Enhanced Store Configuration

Add category metadata to improve presentation:

```yaml
# store/marine.yaml
id: marine
name: Marine Navigation & Monitoring
description: Applications for marine navigation, monitoring, and boat systems.

icon: /usr/share/container-stores/marine/icon.svg

filters:
  include_tags:
    - field::marine

category_metadata:
  - id: navigation
    label: Navigation & Charts
    icon: MapIcon
    description: Chart plotters and navigation tools for marine use

  - id: monitoring
    label: Data & Monitoring
    icon: ChartLineIcon
    description: Data logging, telemetry, and system monitoring

  - id: communication
    label: Communication
    icon: ConnectedIcon
    description: NMEA gateways, AIS receivers, and marine communication tools
```

### Category Metadata Fields

```yaml
category_metadata:
  - id: category-id              # REQUIRED: Must match category:: tag
    label: Display Name          # REQUIRED: User-facing label
    icon: IconName               # OPTIONAL: PatternFly icon name OR file path
    description: Explanation     # OPTIONAL: Help text
```

**Icon Options:**
1. **PatternFly icon name** (e.g., `MapIcon`, `ChartLineIcon`) - most common
2. **File path** (e.g., `/usr/share/icons/navigation.svg`) - for custom icons

### Category Metadata Best Practices

**DO:**
- Provide metadata for common/important categories
- Use clear, descriptive labels
- Choose appropriate PatternFly icons
- Write helpful descriptions
- Keep metadata optional - let auto-discovery handle edge cases

**DON'T:**
- Don't list every possible category in metadata - only enhance the important ones
- Don't duplicate category IDs
- Don't change category IDs lightly - breaks links and bookmarks

### When to Add Category Metadata

Add category metadata when:
- Category is important to your store's user experience
- Auto-derived label isn't clear (e.g., `chartplotters` → "Chartplotters" vs "Chart Plotters")
- You want specific icons for visual navigation
- Category needs explanation (description field)

Skip category metadata when:
- Category is self-explanatory
- Only 1-2 packages in category
- Auto-derived label is good enough

## Benefits & Design Rationale

### Why Tag-Based Categorization?

**1. No Configuration Redundancy**
- Tags live in package metadata
- No separate mapping file to maintain
- Single source of truth

**2. Self-Documenting**
- Package metadata shows all categorizations
- Easy to see which stores include a package
- Git history tracks tag changes

**3. Flexible Organization**
- Packages can belong to multiple categories
- Same package appears in multiple contexts
- Users find apps where they expect them

**4. Automatic Updates**
- New categories emerge when packages use them
- No manual store configuration updates needed
- Reduces maintenance burden

**5. Simple Maintenance**
- Package maintainers control their own categorization
- Store maintainers optionally enhance presentation
- Clear separation of concerns

### Comparison: Traditional vs Tag-Based

| Aspect | Traditional Sections | Tag-Based Categories |
|--------|---------------------|---------------------|
| **Organization** | Debian-defined sections | Domain-specific categories |
| **Flexibility** | One section per package | Multiple categories per package |
| **Discovery** | Manual section assignment | Auto-discovered from tags |
| **Maintenance** | Change package section | Add/remove tags in metadata |
| **Store-specific** | No | Yes - different stores can show different categories |
| **User-friendly** | Technical (net, web, devel) | Domain-specific (navigation, monitoring) |

### Design Decisions

**Why separate `field::` and `category::` facets?**
- `field::` identifies the domain (broad, objective)
- `category::` organizes the UI (specific, user-facing)
- Example: A package might be `field::marine` but appear in both `category::navigation` and `category::safety`

**Why auto-discovery instead of explicit configuration?**
- Reduces coupling between packages and stores
- Allows packages to evolve independently
- Stores automatically reflect new categories
- Less maintenance overhead

**Why optional category metadata?**
- Balances flexibility with presentation quality
- Common categories get polished presentation
- Rare categories still work with auto-derived labels
- Store maintainers control their UX priorities

## References

### Debian Documentation
- [Debian Policy Manual - Archive Organization](https://www.debian.org/doc/debian-policy/ch-archive.html)
- [Debian Debtags](https://wiki.debian.org/Debtags)
- [Debtags Custom Tags](https://wiki.debian.org/Debtags/CustomTags)
- [Faceted Classification](https://wiki.debian.org/Debtags/FacetedClassification)

### HaLOS Documentation
- [Container Store Design](../cockpit-apt/docs/CONTAINER_STORE_DESIGN.md) - Overall store architecture
- [Architecture](../cockpit-apt/docs/ARCHITECTURE.md) - System architecture
- [Specification](../cockpit-apt/docs/SPEC.md) - Requirements and specifications

### Implementation Files
- **Package metadata:** `halos-marine-containers/apps/*/metadata.yaml`
- **Store configs:** `halos-marine-containers/store/*.yaml`
- **Backend code:**
  - `cockpit-apt/backend/cockpit_apt_bridge/commands/list_categories.py`
  - `cockpit-apt/backend/cockpit_apt_bridge/utils/debtag_parser.py`
  - `cockpit-apt/backend/cockpit_apt_bridge/utils/store_filter.py`
- **Frontend code:**
  - `cockpit-apt/frontend/src/views/SectionsView.tsx`
  - `cockpit-apt/frontend/src/components/CategoryCard.tsx`

## Quick Reference

### For Package Maintainers

```yaml
# Minimum required tags for marine store
tags:
  - field::marine          # Gets into marine store
  - category::your-cat     # Appears in "Your Cat" category

# Recommended full tagging
tags:
  - field::marine
  - category::navigation
  - category::safety      # Can be in multiple categories
  - role::container-app
  - interface::web
  - use::routing
```

### For Store Maintainers

```yaml
# Minimum store config (auto-discovery)
id: my-store
name: My Store
filters:
  include_tags: [field::mydomain]

# Enhanced with category metadata
id: my-store
name: My Store
filters:
  include_tags: [field::mydomain]
category_metadata:
  - id: important-category
    label: Important Category
    icon: StarIcon
    description: Our most important category
```

### Command Line Examples

```bash
# List all categories in marine store
cockpit-apt-bridge list-categories --store marine

# List packages in navigation category (marine store)
cockpit-apt-bridge list-packages-by-category navigation --store marine

# List all categories (no store filter)
cockpit-apt-bridge list-categories
```
