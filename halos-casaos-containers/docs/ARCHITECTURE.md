# HaLOS Imported Containers - Architecture

**Version**: 1.0
**Status**: Draft
**Last Updated**: 2025-11-28

## Overview

The HaLOS Imported Containers repository implements a multi-source architecture for automatically converting and packaging container applications from various upstream app stores. This document describes the system architecture, component relationships, technology stack, and deployment strategy.

## System Architecture

### High-Level Architecture

The system consists of four major layers:

1. **Source Management Layer**: Handles upstream repository monitoring, sync configuration, and change detection
2. **Conversion Layer**: Transforms upstream app definitions to HaLOS package format using container-packaging-tools
3. **Build Layer**: Generates Debian packages for apps and store definitions
4. **Distribution Layer**: Publishes packages to APT repository and manages releases

Each layer operates independently, allowing for parallel processing of multiple sources and incremental updates.

### Multi-Source Design

The architecture supports unlimited upstream sources through a source-grouped directory structure. Each source is self-contained with its own:

- **Apps directory**: Converted application definitions
- **Store package**: Debian package defining the Cockpit UI store
- **Upstream configuration**: Sync settings and source metadata
- **Independent versioning**: Store packages evolve separately from app packages

This design enables:
- Adding new sources without modifying existing ones
- Per-source build and deployment
- Independent sync schedules
- Source-specific quality standards
- Clear attribution in package names and UI

### Component Relationships

**Upstream Sources** provide app definitions in their native formats (CasaOS JSON, Runtipi compose, etc.). The **Sync System** monitors these sources for changes and creates pull requests when updates are detected.

**Converters** (part of container-packaging-tools) transform upstream formats to the HaLOS standard format (metadata.yaml, config.yml, docker-compose.yml). Each source type has a dedicated converter that understands its specific format.

**Build System** processes converted apps and store definitions to create Debian packages. The build process is source-aware, applying correct naming prefixes and dependencies.

**APT Repository** (apt.hatlabs.fi) receives packages through the CI/CD pipeline, making them available for installation on HaLOS systems via standard APT mechanisms.

**Cockpit UI** discovers stores through installed store packages and filters apps based on package name patterns defined in each store's configuration.

## Directory Structure

### Root Level Organization

The repository uses a source-grouped structure at the root level:

**sources/** - Contains all upstream source integrations, each in its own subdirectory
**tools/** - Build automation scripts and validation utilities
**docs/** - Specifications, architecture, and guides
**.github/** - CI/CD workflows and GitHub configuration

### Source Directory Structure

Each source directory follows a standard template:

**apps/** - Contains converted application definitions, one subdirectory per app
**store/** - Store definition YAML and Debian packaging files for the store package
**upstream/** - Sync configuration and metadata about the upstream source
**README.md** - Source-specific documentation

### App Directory Structure

Each app within a source's apps/ directory contains:

**metadata.yaml** - Package metadata (name, description, version, license, maintainer)
**config.yml** - User-configurable parameters schema (environment variables, volumes, ports)
**docker-compose.yml** - Container service definition
**debian/** - Debian packaging files (control, rules, install, changelog)

### Template Directory

The **sources/_template/** directory provides a skeleton for adding new sources:

**apps/.gitkeep** - Placeholder for apps directory
**store/template.yaml** - Example store configuration
**upstream/source.yaml.example** - Template sync configuration
**README.md.template** - Documentation template with placeholders

## Data Models and Schemas

### Package Metadata Schema

Package metadata follows the HaLOS container package standard defined in container-packaging-tools. Key elements:

- **Package name**: Source-prefixed, lowercase (e.g., casaos-uptimekuma-container)
- **Version**: Tracks upstream app version
- **Description**: Synopsis (80 char limit) and long description
- **Origin**: Always "Hat Labs" for imported packages
- **Source attribution**: Metadata indicates which upstream source the app came from
- **Dependencies**: Declares docker.io and any other required packages
- **Architecture**: Typically "all" (container images handle architecture)

### Configuration Schema

Configuration schemas define user-customizable parameters:

- **Environment variables**: Key-value pairs with types, defaults, and descriptions
- **Volumes**: Mount points with host paths and container paths
- **Ports**: Published ports with host/container port mapping
- **Groups**: Logical grouping of related configuration options
- **Validation**: Type checking, required fields, allowed values

### Store Definition Schema

Store packages define how apps appear in Cockpit UI:

- **Store ID**: Unique identifier for the store
- **Display name**: User-visible store name
- **Description**: Markdown-formatted store description
- **Icon and banner**: Visual assets for store branding
- **Package filters**: Patterns to include packages in this store (e.g., casaos-*-container)
- **Origin filters**: Filter by package origin (Hat Labs)
- **Display options**: Sorting, screenshot display, categories

### Upstream Source Schema

Upstream source configuration defines sync behavior:

- **Source type**: GitHub, GitLab, HTTP, or other
- **Repository URL**: Location of upstream app store
- **Path within repo**: Directory containing app definitions
- **Branch**: Which branch to track
- **Converter type**: Which converter to use (casaos, runtipi, custom)
- **Sync schedule**: How often to check for updates
- **Conversion options**: Converter-specific settings

## Technology Stack

### Core Technologies

**Git and GitHub**: Version control and collaboration platform. GitHub Actions provides CI/CD execution.

**Debian Packaging**: Standard .deb package format for distribution. Uses dpkg-buildpackage, debhelper, and dh-python.

**Python 3.11+**: Runtime for container-packaging-tools and build scripts. Uses uv for dependency management.

**Docker**: Required at runtime for container execution. Not used in build process.

**APT Repository**: Custom repository at apt.hatlabs.fi for package distribution.

### Build Tools

**container-packaging-tools**: Python package providing converters for different upstream formats. Installed via uv from GitHub repository.

**dpkg-buildpackage**: Standard Debian package builder. Generates .deb, .buildinfo, and .changes files.

**bash**: Shell scripts for build automation and validation.

### CI/CD Infrastructure

**GitHub Actions**: Runs workflows for PR validation, main branch builds, and releases.

**Hat Labs Shared Workflows**: Reusable workflow definitions from hatlabs/shared-workflows repository. Provides standardized PR checks, build processes, and release publishing.

**APT Repository Integration**: Automated package upload using APT_REPO_PAT secret for authentication.

### Technology Decisions and Rationale

**Why source-grouped structure**: Enables independent evolution of sources without conflicts. New sources can be added without touching existing code.

**Why Debian packages**: Standard distribution format for Debian-based systems. Integrates with APT dependency resolution and version management.

**Why per-source stores**: Provides clear attribution in UI. Users can choose which sources to install and trust.

**Why container-packaging-tools**: Proven converter with 100% success rate for CasaOS. Extensible to new formats through plugin system.

**Why GitHub Actions**: Native integration with GitHub repositories. Shared workflows reduce duplication across HaLOS projects.

## Integration Points

### Upstream Integration

The system integrates with upstream app stores through:

**Git repository monitoring**: Daily checks for changes using GitHub Actions scheduled workflows
**Format-specific converters**: Transform upstream app definitions to HaLOS format
**Change detection**: Compare current apps with upstream to identify new/updated/removed apps
**Pull request creation**: Automated PRs when upstream changes are detected

### Container Packaging Tools Integration

Integration with container-packaging-tools provides:

**Batch conversion**: Convert all apps from a source in a single operation
**Validation**: Schema validation during conversion ensures quality
**Metadata generation**: Automatic generation of Debian control files
**Fallback handling**: Intelligent defaults for missing or invalid upstream data

### APT Repository Integration

Packages are published through:

**Unstable channel**: Automatic on merge to main branch
**Stable channel**: Manual via GitHub release creation
**Repository structure**: Organized by distribution (trixie) and component (main)
**Authentication**: GitHub PAT with write access to apt.hatlabs.fi repository

### Cockpit UI Integration

Store packages integrate with Cockpit through:

**Package installation**: Installing a store package makes it visible in Cockpit
**Store configuration files**: Installed to /usr/share/container-stores/
**Package filtering**: Cockpit filters packages based on store configuration
**Multi-store support**: Multiple stores coexist and appear separately in UI

## Build Process

### Build Workflow

The build process operates in these phases:

**Phase 1 - Validation**: Validate directory structure and all app definitions against schemas

**Phase 2 - Per-Source Building**: For each source in sources/:
- Build store package using Debian packaging in store/debian/
- For each app in apps/, generate Debian package with container-packaging-tools
- Apply source-specific naming prefix
- Collect all .deb, .buildinfo, and .changes files

**Phase 3 - Aggregation**: Collect all packages from all sources into build/ directory

**Phase 4 - Verification**: Ensure all expected packages were created successfully

**Phase 5 - Publishing**: Upload packages to APT repository using shared workflows

### Package Naming Convention

All packages follow strict naming conventions:

**App packages**: {source}-{appname}-container (e.g., casaos-uptimekuma-container)
**Store packages**: {source}-container-store (e.g., casaos-official-container-store)
**All lowercase**: Debian package names must be lowercase
**Hyphens only**: No underscores or other special characters

### Build Modes

The build system supports multiple modes:

**Full build**: Build all sources and all apps (default for main branch)
**Source build**: Build only a specific source
**Incremental build**: Build only changed apps (planned for future optimization)

### Build Scripts

**tools/build-all.sh**: Iterates through all non-template sources and builds each one. Creates build/ directory and collects all packages.

**tools/build-source.sh**: Builds a single source. Takes source name as argument. Builds store package first, then all app packages.

**tools/validate-structure.sh**: Validates repository structure before building. Ensures all required directories exist and contain expected files.

## Version Management

### Repository Version

The VERSION file at repository root tracks infrastructure version:

- Incremented when directory structure changes
- Incremented when build process changes
- Incremented when adding new sources
- NOT incremented for app updates or store package updates

Used for git tags: v{version}+{N}_pre (unstable) and v{version}+{N} (stable)

### Store Package Versions

Each store package has independent versioning in its debian/changelog:

- Incremented when store configuration changes
- Incremented when store metadata (icon, banner, description) changes
- Follows Debian versioning: {upstream}+{revision} or {upstream}-{debian}

### App Package Versions

App packages track upstream versions:

- Version comes from upstream app definition
- Debian revision may be added for packaging fixes
- Different sources may have different versions of the same app

### Release Management

**Unstable releases**: Automatic on every merge to main. Packages uploaded to unstable component.

**Stable releases**: Manual via GitHub release creation. Packages promoted from unstable or rebuilt for stable component.

**Tag format**: Git tags use unified versioning format with +N revision suffix.

## CI/CD Architecture

### Workflow Organization

**PR Workflow (.github/workflows/pr.yml)**: Runs on pull requests
- Validates directory structure
- Validates all app definitions
- Builds all packages (but doesn't publish)
- Reports status to PR

**Main Workflow (.github/workflows/main.yml)**: Runs on merge to main
- Builds all packages
- Publishes to unstable channel
- Creates git tags
- Generates changelog

**Release Workflow (.github/workflows/release.yml)**: Runs on release publication
- Builds packages for stable channel
- Publishes to stable component
- Creates release artifacts

**Sync Workflow (.github/workflows/sync-upstream.yml)**: Scheduled daily (planned)
- Checks each source for upstream changes
- Creates PR if changes detected
- Runs converter on modified apps

### Shared Workflows

The repository uses Hat Labs shared workflows:

**pr-checks.yml**: Standard PR validation pattern
**build-release.yml**: Build and publish to unstable
**publish-stable.yml**: Promote packages to stable

Benefits:
- Consistent CI/CD across all HaLOS repositories
- Centralized workflow maintenance
- Reduced duplication
- Standard release process

### GitHub Actions Structure

**Actions (.github/actions/)**: Reusable action definitions
- build-deb/: Build Debian packages action
- run-tests/: Run validation tests action

**Scripts (.github/scripts/)**: Helper scripts for workflows
- rename-packages.sh: Add distribution/component suffix to packages
- generate-changelog.sh: Generate changelog from git history
- generate-release-notes.sh: Create release notes from changes

### Secrets and Configuration

**APT_REPO_PAT**: GitHub personal access token with write access to apt.hatlabs.fi repository

**Workflow Configuration**:
- apt-distro: trixie (Debian 13)
- apt-component: main
- package-name: imported-containers-meta (metapackage for releases)

## Deployment Architecture

### Development Environment

Developers work in this workflow:

**Local repository**: Clone halos-imported-containers
**Container-packaging-tools**: Install via uv for local conversion testing
**Build locally**: Run tools/build-source.sh to test builds
**Create PR**: Push branch and create PR for review
**CI validates**: GitHub Actions runs full validation
**Merge**: After PR approval, merge to main triggers unstable release

### Unstable Channel

The unstable channel provides continuous deployment:

**Trigger**: Every merge to main branch
**Build**: All packages rebuilt
**Publishing**: Uploaded to trixie/main component with _pre suffix
**Availability**: Immediately available for testing on HaLOS systems
**Purpose**: Rapid iteration and testing

### Stable Channel

The stable channel provides vetted releases:

**Trigger**: Manual GitHub release creation
**Build**: Packages built for stable (or promoted from unstable)
**Publishing**: Uploaded to trixie/main component without _pre suffix
**Availability**: Available for production HaLOS systems
**Purpose**: Production-ready, tested releases

### Multi-Stage Deployment

The deployment follows this progression:

**Stage 1 - Development**: Local builds and testing
**Stage 2 - PR**: Validation in CI, review by maintainers
**Stage 3 - Unstable**: Deployed to unstable channel for testing
**Stage 4 - Validation**: Testing on HaLOS systems
**Stage 5 - Stable**: Release creation publishes to stable channel

## Security Considerations

### Package Verification

All packages include:

**Source attribution**: Metadata clearly indicates upstream source
**Upstream URL**: Link back to original upstream definition
**License information**: Recorded from upstream metadata
**Maintainer**: Hat Labs as package maintainer
**Signature**: APT repository signing (handled by apt.hatlabs.fi)

### Upstream Trust

The system does not:

- Validate upstream container images (trust upstream source)
- Scan for vulnerabilities (rely on upstream scanning)
- Modify container images (only package metadata)

Users should:

- Understand which sources they trust
- Install only store packages from trusted sources
- Review app descriptions and permissions before installing

### Build Security

Build process runs in:

**Isolated GitHub Actions runners**: Fresh environment for each build
**No network access during build**: Builds use only repository contents
**Secrets properly scoped**: APT_REPO_PAT has minimal required permissions
**Reproducible builds**: Same input produces same output

### Repository Access Control

**Main branch protection**: Requires pull request reviews
**Branch protection rules**: Enforce checks passing before merge
**Write access**: Limited to maintainers
**PAT rotation**: APT_REPO_PAT should be rotated periodically

## Scalability Considerations

### Handling Growth

The architecture scales to support:

**Many sources**: Source-grouped structure isolates sources
**Many apps per source**: Parallel build of apps within a source
**Frequent updates**: Incremental conversion of changed apps only
**Large packages**: Build artifacts collected efficiently

### Performance Optimizations

Current optimizations:

**Source independence**: Sources can be built in parallel (future)
**Shared workflows**: Reduce CI/CD execution time through caching
**Validation early**: Catch errors before expensive builds
**Incremental sync**: Only process changed apps (planned)

Future optimizations:

**Parallel source builds**: Build multiple sources simultaneously
**Cached conversions**: Skip conversion if upstream unchanged
**Incremental package builds**: Build only changed apps
**Distributed builds**: Use matrix strategy for parallelism

### Resource Management

Build resource usage:

**GitHub Actions minutes**: ~5-10 minutes per full build currently
**Storage**: Build artifacts cleaned after upload
**APT repository size**: Grows with number of packages (managed by apt.hatlabs.fi)
**Network**: Minimal during build, higher during upstream sync

## Extension Points

### Adding New Sources

To add a new source:

**Step 1**: Copy sources/_template/ to sources/{newsource}/
**Step 2**: Configure upstream/source.yaml with source details
**Step 3**: Create store/{newsource}.yaml with store definition
**Step 4**: Create debian packaging in store/debian/
**Step 5**: Run converter to populate apps/
**Step 6**: Test build with tools/build-source.sh {newsource}
**Step 7**: Create PR with new source

The template system ensures consistency and reduces errors.

### Adding New Converters

Container-packaging-tools supports:

**Built-in converters**: CasaOS, Runtipi (future)
**Custom converters**: Plugin system for new formats
**Converter options**: Per-source configuration in upstream/source.yaml

New converters should:

- Follow container-packaging-tools plugin API
- Handle missing/invalid data gracefully
- Generate complete metadata.yaml, config.yml, docker-compose.yml
- Support batch conversion mode

### Custom Build Steps

Build process can be extended:

**Pre-build hooks**: Validation, linting, formatting
**Post-build hooks**: Testing, verification, notifications
**Per-source hooks**: Source-specific build customizations
**Integration tests**: Validate packages install correctly

## Maintenance and Operations

### Daily Operations

Automated daily tasks:

**Upstream sync**: Check all sources for changes (when implemented)
**Build on merge**: Automatic build and publish to unstable
**Dependency updates**: Dependabot for GitHub Actions and Python deps

Manual operations:

**PR review and merge**: Review automated PRs from sync
**Release creation**: Create GitHub release for stable channel
**Source addition**: Add new sources as needed

### Monitoring

Monitor these indicators:

**Conversion success rate**: Should remain 100% for valid apps
**Build success rate**: Should be >95%
**Sync latency**: Changes should appear within 24 hours
**CI/CD failures**: Investigate and fix promptly

### Troubleshooting

Common issues and resolution:

**Conversion failure**: Check upstream format changes, update converter
**Build failure**: Validate package metadata, check dependencies
**Sync failure**: Check upstream repository access, network issues
**Publication failure**: Verify APT_REPO_PAT is valid

## Future Architecture Evolution

### Planned Enhancements

**Automated upstream sync**: Daily scheduled workflow to check sources
**Incremental builds**: Build only changed apps to reduce CI time
**Parallel source builds**: Use GitHub Actions matrix for parallelism
**Template improvements**: More comprehensive source templates

### Potential Changes

**Alternative package formats**: Flatpak, Snap (if requested)
**Multi-repository support**: Publish to multiple APT repositories
**Metadata enhancement**: Richer app metadata, screenshots, ratings
**Search integration**: Cross-source app search capability

### Backward Compatibility

Architecture changes must:

**Maintain package names**: Existing packages continue to work
**Support existing stores**: Store packages remain compatible
**Preserve data**: User configurations and app data unaffected
**Document migrations**: Clear upgrade path for breaking changes

## Conclusion

This architecture provides a scalable, maintainable foundation for importing container applications from multiple upstream sources into the HaLOS ecosystem. The source-grouped design enables independent evolution of sources while maintaining consistency through shared tooling and workflows.

Key architectural strengths:

- **Scalability**: Supports unlimited sources and apps
- **Maintainability**: Source isolation and template system
- **Automation**: Minimal manual intervention required
- **Extensibility**: Clear extension points for new sources and converters
- **Quality**: Validation at every step ensures package quality

The architecture follows HaLOS patterns established in halos-marine-containers while extending them for multi-source scenarios. This consistency simplifies maintenance and enables reuse of tooling across projects.
