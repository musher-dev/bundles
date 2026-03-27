---
name: aligning-ci-environments
version: 1.0.0
user-invocable: false
description: >-
  Ensure local development environments match CI/CD environments for consistent
  behavior including shared version sources, devcontainer reuse in CI, reproducible
  dependency installation, environment variable parity, Docker image sharing, and
  lock file enforcement. Use when diagnosing works-on-my-machine issues, designing
  CI pipelines that mirror local dev, or auditing environment parity.
  Triggered by: CI alignment, CI environment, works on my machine, CI parity,
  environment parity, CI local match, GitHub Actions, CI pipeline, CI docker,
  reproducible builds, lock file, frozen lockfile.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# CI Environment Alignment

Strategies for eliminating divergence between local development, CI/CD, and production environments.

## Purpose

"Works on my machine" is the symptom of environment divergence. The developer uses Node 22.11.0, CI installs 22.9.0, and a subtle API difference breaks the build. Or dependencies install cleanly locally but fail in CI because the lock file is stale. This skill provides the alignment patterns that prevent these failures.

Use this skill when designing CI pipelines that should match local development, diagnosing CI failures that don't reproduce locally, or auditing an existing pipeline for environment drift.

---

## Alignment Dimensions

| Dimension | Source of Truth | Local Reads From | CI Reads From |
|---|---|---|---|
| Runtime versions | `.tool-versions` / `.nvmrc` | Version manager (mise, nvm) | Setup action (mise-action, setup-node) |
| Dependencies | Lock file | `npm install` / `pip install` | `npm ci` / `pip install -r` |
| System packages | Dockerfile or devcontainer | Docker / devcontainer build | Same Docker image or equivalent action |
| Environment variables | `.env.example` | `.env` (local copy) | CI secrets / env configuration |
| Database schema | Migrations | `docker compose up db` | Test database in CI |

---

## Version Alignment

### Single Source: Version File → CI Setup Action

```yaml
# GitHub Actions — read from .tool-versions
- uses: jdx/mise-action@v2
  # Automatically reads .tool-versions

# Or read from .nvmrc
- uses: actions/setup-node@v4
  with:
    node-version-file: '.nvmrc'

# Or read from .python-version
- uses: actions/setup-python@v5
  with:
    python-version-file: '.python-version'
```

### Anti-Pattern: Hardcoded Version in CI

```yaml
# BAD — version drifts from .tool-versions
- uses: actions/setup-node@v4
  with:
    node-version: '22'

# GOOD — reads from version file
- uses: actions/setup-node@v4
  with:
    node-version-file: '.nvmrc'
```

**Rule:** CI must never hardcode a version that is also declared in a version file. Always read from the file.

---

## Dependency Alignment

### Lock File Enforcement

| Package Manager | Local Install | CI Install | Enforces Lock File? |
|---|---|---|---|
| npm | `npm install` | `npm ci` | Yes (`ci` fails if lock diverges) |
| pnpm | `pnpm install` | `pnpm install --frozen-lockfile` | Yes |
| yarn | `yarn install` | `yarn install --immutable` | Yes |
| pip | `pip install -r requirements.txt` | `pip install -r requirements.txt` | Partial (no built-in freeze check) |
| uv | `uv sync` | `uv sync --frozen` | Yes |
| Poetry | `poetry install` | `poetry install --no-interaction` | Yes (uses poetry.lock) |

**Rules:**
- CI must use the frozen/immutable install mode
- If CI install succeeds but local install differs, the lock file is stale — fail the build
- Commit lock files for applications (always) and libraries (per ecosystem convention)

### Caching Dependencies in CI

```yaml
# GitHub Actions — cache node_modules
- uses: actions/setup-node@v4
  with:
    node-version-file: '.nvmrc'
    cache: 'npm'

# Or manual cache with hash
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ hashFiles('package-lock.json') }}
```

---

## Devcontainer Reuse in CI

The same devcontainer used for local development can run CI tasks, ensuring exact environment parity.

### GitHub Actions with devcontainers/ci

```yaml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests in devcontainer
        uses: devcontainers/ci@v0.3
        with:
          runCmd: |
            npm ci
            npm test
            npm run lint
```

### When to Reuse vs Replicate

| Approach | Pros | Cons | Use When |
|---|---|---|---|
| **Reuse devcontainer** | Exact parity | Slower build (container build overhead) | Environment is complex, parity is critical |
| **Replicate with actions** | Faster builds | May drift from local | Environment is simple, speed matters |
| **Shared Docker image** | Exact parity + fast CI | Requires image registry | Large team, complex toolchain |

### Shared Docker Image Pattern

```dockerfile
# Build a CI image from the same Dockerfile used for development
FROM myapp:development AS ci
# CI-specific tools
RUN npm ci --production=false
```

```yaml
# CI uses the pre-built image
jobs:
  test:
    runs-on: ubuntu-latest
    container: ghcr.io/myorg/myapp-ci:latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```

---

## Environment Variable Parity

### CI Secret Configuration

```yaml
# GitHub Actions
env:
  DATABASE_URL: postgresql://test:test@localhost:5432/test_db
  REDIS_URL: redis://localhost:6379/0
  STRIPE_SECRET_KEY: ${{ secrets.STRIPE_TEST_KEY }}
  APP_ENV: test
```

### Parity Checklist

| Variable Type | Local Source | CI Source | Parity Rule |
|---|---|---|---|
| App config | `.env` | Workflow `env:` | Same variable names |
| Service URLs | `.env` (localhost) | Workflow `env:` (service container) | Different hosts, same format |
| Secrets | `.env` (test values) | GitHub Secrets | Same variable names, different values |
| Feature flags | `.env` | Workflow `env:` | Match local defaults |

**Rule:** Every variable in `.env.example` should have a corresponding entry in CI configuration. Use the `.env.example` as the checklist.

---

## Database Parity

### GitHub Actions Service Containers

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      db:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 5s
          --health-timeout 3s
          --health-retries 5
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 5s
          --health-timeout 3s
          --health-retries 5
```

**Rules:**
- Use the same database image version in CI as in docker-compose.yml
- Apply the same migrations in CI as in local development
- Use health checks in CI service containers just like in Docker Compose

---

## Task Runner Alignment

### Same Commands, Both Environments

```yaml
# CI uses the same task runner commands as local dev
steps:
  - run: task lint
  - run: task test
  - run: task build
```

**Rule:** If developers run `task test` locally, CI should run `task test` — not a bespoke `pytest -v --cov` command. This ensures the exact same arguments, environment setup, and behavior.

### When CI Needs Extra Steps

CI may need additional steps not in the task runner (artifact upload, deployment). These are CI-specific — don't add them to the task runner:

```yaml
steps:
  - run: task test             # Same as local
  - run: task build            # Same as local
  - uses: actions/upload-artifact@v4  # CI-specific
    with:
      name: build
      path: dist/
```

---

## Drift Detection

### Automated Parity Checks

Add a CI job that detects drift between local and CI configurations:

```yaml
  check-parity:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Check version file consistency
      - name: Verify version files
        run: |
          if [[ -f .tool-versions ]] && [[ -f .nvmrc ]]; then
            TV_NODE=$(grep '^nodejs' .tool-versions | awk '{print $2}')
            NVMRC_NODE=$(cat .nvmrc)
            if [[ "$TV_NODE" != "$NVMRC_NODE" ]]; then
              echo "::error::Version mismatch: .tool-versions ($TV_NODE) vs .nvmrc ($NVMRC_NODE)"
              exit 1
            fi
          fi

      # Check lock file freshness
      - name: Verify lock file
        run: npm ci  # Fails if lock file doesn't match package.json
```

---

## Anti-Patterns

### 1. Different Tool Versions in CI

CI uses Node 20, local uses Node 22. Tests pass in both but with subtly different behavior that surfaces in production.

### 2. `npm install` Instead of `npm ci` in CI

`npm install` may modify the lock file. `npm ci` fails if the lock file is out of sync — which is what you want in CI.

### 3. Missing Service Health Checks in CI

```yaml
services:
  db:
    image: postgres:16
    # No health check — tests may run before DB is ready
```

Tests intermittently fail because PostgreSQL isn't accepting connections when they start.

### 4. Different Commands in CI vs Local

Developers run `task test`, CI runs `pytest -xvs --tb=short`. The arguments differ, so behavior differs. Use the same task runner commands.

### 5. Hardcoded Secrets in CI Config

```yaml
env:
  API_KEY: sk_live_actual_key_here  # Committed to repo!
```

Use CI secrets management, never inline values.

---

## Checklist

When reviewing CI/local environment alignment:

- [ ] CI reads runtime versions from version files (not hardcoded)
- [ ] CI uses frozen/immutable dependency installation
- [ ] CI service containers use the same image versions as docker-compose.yml
- [ ] CI service containers have health checks
- [ ] CI uses the same task runner commands as local development
- [ ] Every `.env.example` variable has a CI equivalent
- [ ] Lock files are committed and enforced
- [ ] No secrets hardcoded in CI configuration files

---

## Related Skills and Agents

- **`managing-toolchain-versions` skill**: Establishes the version files that CI reads from
- **`composing-docker-development` skill**: Docker Compose services should match CI service containers
- **`managing-environment-variables` skill**: `.env.example` is the checklist for CI env var configuration
- **`authoring-task-runners` skill**: CI should call the same task runner commands
- **`environment_auditor` agent**: Can detect CI/local drift as part of a comprehensive audit
- **`environment_scaffolder` agent**: Generates CI-aligned configuration when scaffolding environments
