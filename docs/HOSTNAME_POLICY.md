# Hard-coded Hostname Policy

This document defines the policy for using `halos.local` and `halos.hal` hostnames in HaLOS codebases.

## Policy Summary

**Rule**: Hard-coded references to `halos.local` or `halos.hal` are NOT allowed except in:

1. **Documentation files** (`*.md`) - Examples and explanations are fine
2. **halos-pi-gen repository** - This is the image builder where the default system hostname is configured

## What This Means

### NOT Allowed

- Source code with hard-coded hostnames (even with env var fallbacks like `${HALOS_DOMAIN:-halos.local}`)
- Test files with hard-coded hostnames
- Configuration files with hard-coded hostnames
- Shell scripts with hard-coded default values

### Allowed

- Documentation files (README.md, AGENTS.md, SPEC.md, etc.) can use these hostnames for examples
- `halos-pi-gen` can configure the default system hostname

## Enforcement

Each repository has a pre-commit hook (`lefthook`) that runs `.github/scripts/check-hardcoded-hostnames.sh` to detect violations.

### Installing Hooks

```bash
cd <repository>
lefthook install
```

### Running Manually

```bash
.github/scripts/check-hardcoded-hostnames.sh
```

## How to Fix Violations

### Source Code

Replace hard-coded hostnames with environment variables or configuration:

```python
# Bad
hostname = "halos.local"

# Good
hostname = os.environ.get("HALOS_HOSTNAME")
if not hostname:
    raise ValueError("HALOS_HOSTNAME environment variable is required")
```

### Test Files

Read test configuration from environment:

```python
# Bad
BASE_URL = "https://halos.local:9090"

# Good
TEST_HOST = os.environ.get("HALOS_TEST_HOST", "localhost")
BASE_URL = f"https://{TEST_HOST}:9090"
```

For Playwright tests:
```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    baseURL: process.env.HALOS_TEST_URL || 'https://localhost:9090',
  },
});
```

### Shell Scripts / Run Scripts

Use environment variables with NO default fallback, or make the target explicit:

```bash
# Bad
DEPLOY_TARGET="${HALOS_TEST_HOST:-halos.local}"

# Good - require explicit configuration
DEPLOY_TARGET="${HALOS_TEST_HOST:?HALOS_TEST_HOST is required}"

# Good - pass as argument
deploy_to "$1"  # User must pass target
```

### Docker Compose

Use required environment variables:

```yaml
# Bad
- traefik.http.routers.app.rule=Host(`app.${HALOS_DOMAIN:-halos.local}`)

# Good
- traefik.http.routers.app.rule=Host(`app.${HALOS_DOMAIN:?HALOS_DOMAIN is required}`)
```

## Current Violations

As of the lint check implementation, the following repositories have violations that need to be fixed:

| Repository | Files with Violations |
|-----------|----------------------|
| container-packaging-tools | registry.py, test_registry.py |
| cockpit-apt | E2E test files, playwright.config.ts |
| homarr-container-adapter | run script, authelia.rs, homarr.rs, registry.rs |
| halos-mdns-publisher | avahi_manager.rs, config.rs |
| halos-marine-containers | run script, test-*.sh scripts |
| halos-core-containers | docker-compose.yml, metadata.yaml, run script |
| halos-homarr-branding | run script, generate-seed-db.py, debian/changelog |

**Clean repositories** (no violations):
- cockpit-networkmanager-halos
- HALPI2-rust-daemon
- halos-imported-containers

## Rationale

Hard-coded hostnames cause problems:

1. **Test fragility** - Tests only work with specific network configurations
2. **Deployment issues** - Assume specific hostnames instead of using configuration
3. **Security** - May leak internal infrastructure details
4. **Flexibility** - Prevents using different hostnames for different environments

By requiring explicit configuration, we ensure that all deployments are intentional and configurable.
