---
name: scaffolding-workspace-setup
version: 1.0.0
user-invocable: false
description: >-
  Design workspace initialization scripts and onboarding automation including
  setup.sh/setup scripts, dependency bootstrapping, prerequisite verification,
  health checks, cross-platform considerations (macOS/Linux/WSL), and first-run
  developer experience. Use when creating setup scripts, reviewing onboarding
  flows, or designing developer-friendly bootstrap experiences.
  Triggered by: setup script, workspace setup, onboarding, bootstrap, dev setup,
  first run, getting started, developer onboarding, setup.sh, project setup,
  workspace initialization.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Workspace Setup Scaffolding

Patterns for designing initialization scripts and onboarding automation that take a developer from fresh clone to running application.

## Purpose

The first 30 minutes with a new codebase determine whether a developer feels productive or frustrated. A well-designed setup process installs dependencies, configures the environment, seeds data, and verifies everything works — all with a single command. This skill covers the patterns that make `./setup.sh` (or `task setup`) reliable, idempotent, and informative.

Use this skill when creating a setup script for a new project, improving an existing onboarding flow, or designing the "Getting Started" section of a project's README.

---

## Setup Script Anatomy

### Standard Structure

```bash
#!/usr/bin/env bash
set -euo pipefail

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

info()  { echo -e "${GREEN}[INFO]${NC} $1"; }
warn()  { echo -e "${YELLOW}[WARN]${NC} $1"; }
error() { echo -e "${RED}[ERROR]${NC} $1"; exit 1; }

# ─── Prerequisites ──────────────────────────────────────────────
info "Checking prerequisites..."

command -v git >/dev/null 2>&1 || error "git is required but not installed"
command -v docker >/dev/null 2>&1 || error "docker is required but not installed"
command -v node >/dev/null 2>&1 || error "node is required — install via .tool-versions"

NODE_VERSION=$(node --version | sed 's/v//')
EXPECTED_NODE=$(grep '^nodejs' .tool-versions 2>/dev/null | awk '{print $2}')
if [[ -n "$EXPECTED_NODE" && "$NODE_VERSION" != "$EXPECTED_NODE" ]]; then
  warn "Node version mismatch: have $NODE_VERSION, expected $EXPECTED_NODE"
fi

# ─── Dependencies ───────────────────────────────────────────────
info "Installing dependencies..."

npm install
pip install -e '.[dev]' 2>/dev/null || true

# ─── Environment ────────────────────────────────────────────────
info "Configuring environment..."

if [[ ! -f .env ]]; then
  cp .env.example .env
  info "Created .env from .env.example — edit with your values"
else
  info ".env already exists — skipping"
fi

# ─── Infrastructure ─────────────────────────────────────────────
info "Starting infrastructure services..."

docker compose up -d db redis
info "Waiting for PostgreSQL..."
until docker compose exec db pg_isready -U dev -q 2>/dev/null; do
  sleep 1
done

# ─── Database ───────────────────────────────────────────────────
info "Running database migrations..."

task db:migrate 2>/dev/null || npm run db:migrate 2>/dev/null || true

info "Seeding development data..."
task db:seed 2>/dev/null || npm run db:seed 2>/dev/null || true

# ─── Verification ───────────────────────────────────────────────
info "Verifying setup..."

CHECKS_PASSED=0
CHECKS_TOTAL=0

check() {
  CHECKS_TOTAL=$((CHECKS_TOTAL + 1))
  if eval "$2" >/dev/null 2>&1; then
    info "  ✓ $1"
    CHECKS_PASSED=$((CHECKS_PASSED + 1))
  else
    warn "  ✗ $1"
  fi
}

check "Node dependencies installed" "test -d node_modules"
check "Python dependencies installed" "python -c 'import app' 2>/dev/null"
check "Environment file exists" "test -f .env"
check "PostgreSQL accepting connections" "docker compose exec db pg_isready -U dev -q"
check "Redis accepting connections" "docker compose exec redis redis-cli ping"

echo ""
info "Setup complete: $CHECKS_PASSED/$CHECKS_TOTAL checks passed"

if [[ $CHECKS_PASSED -lt $CHECKS_TOTAL ]]; then
  warn "Some checks failed — review warnings above"
fi

echo ""
info "Next steps:"
info "  1. Edit .env with your secret values"
info "  2. Run 'task dev' to start the development server"
```

---

## Design Principles

### 1. Idempotency

Every step must be safe to run multiple times. Use guard clauses:

```bash
# Good: Skip if already done
[[ -f .env ]] || cp .env.example .env

# Good: npm install is naturally idempotent
npm install

# Bad: Duplicates data on every run
psql -f seed.sql
```

### 2. Fail Fast with Clear Messages

```bash
set -euo pipefail  # Exit on error, undefined vars, pipe failures

command -v docker >/dev/null 2>&1 || error "docker is required but not installed"
```

**Rule:** Check all prerequisites before doing any work. Don't install half the dependencies and then fail because Docker isn't installed.

### 3. Progress Communication

Print what's happening at each step. Silent scripts feel broken:

```bash
info "Installing Node dependencies..."
npm install

info "Starting PostgreSQL..."
docker compose up -d db
```

### 4. Graceful Degradation

Optional steps should warn, not fail:

```bash
# Optional: Seed data
if command -v task >/dev/null 2>&1; then
  task db:seed
else
  warn "Task not installed — skipping database seeding"
  warn "Run 'npm run db:seed' manually"
fi
```

---

## Bootstrapping Order

Steps must execute in dependency order:

```
1. Check prerequisites     (tools installed? right versions?)
2. Install dependencies    (npm install, pip install)
3. Configure environment   (copy .env.example → .env)
4. Start infrastructure    (docker compose up -d)
5. Wait for services       (health check loop)
6. Run migrations          (database schema)
7. Seed data               (development fixtures)
8. Verify setup            (health checks on everything)
9. Print next steps        (what to do after setup)
```

**Rule:** Never run migrations before infrastructure is ready. Never seed data before migrations are applied.

---

## Cross-Platform Considerations

### macOS / Linux / WSL Detection

```bash
OS="$(uname -s)"
case "$OS" in
  Darwin) PLATFORM="macos" ;;
  Linux)
    if grep -qi microsoft /proc/version 2>/dev/null; then
      PLATFORM="wsl"
    else
      PLATFORM="linux"
    fi
    ;;
  *) error "Unsupported platform: $OS" ;;
esac
```

### Platform-Specific Handling

| Concern | macOS | Linux | WSL |
|---|---|---|---|
| Package manager | `brew install` | `apt-get install` | `apt-get install` |
| Docker performance | Use OrbStack or cached mounts | Native performance | Enable WSL2 backend |
| File permissions | Generally fine | Watch for root-owned Docker files | Watch for NTFS/ext4 mismatch |
| Line endings | LF (default) | LF (default) | May need `.gitattributes` |

### When to Skip Platform Detection

If the project uses devcontainers or Docker for development, skip platform-specific logic. The container provides a consistent environment across all platforms:

```bash
if [[ -f .devcontainer/devcontainer.json ]]; then
  info "Devcontainer detected — use 'Dev Containers: Reopen in Container' in VS Code"
  exit 0
fi
```

---

## README Integration

The setup script should be referenced from the README:

```markdown
## Getting Started

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) (or [OrbStack](https://orbstack.dev/) on macOS)
- [mise](https://mise.jdx.dev/) (or [nvm](https://github.com/nvm-sh/nvm) for Node-only)

### Setup

```bash
git clone <repo-url>
cd <project>
./scripts/setup.sh
```

### Development

```bash
task dev    # Start dev server (http://localhost:8000)
task test   # Run tests
task lint   # Run linters
```
```

**Rules:**
- Prerequisites section lists what must be installed before setup
- Setup is a single command
- Development section shows the 3-5 most common commands

---

## Anti-Patterns

### 1. Non-Idempotent Setup

Running `./setup.sh` twice inserts duplicate seed data, reinstalls system packages, or errors on existing directories. Every step must handle the "already done" case.

### 2. Silent Failure

```bash
npm install 2>/dev/null
psql -f migrate.sql 2>/dev/null || true
```

Swallowing errors makes debugging impossible. Let errors surface with clear messages.

### 3. Interactive Prompts

```bash
read -p "Enter your database password: " DB_PASS
```

Setup scripts should work unattended. Use `.env` files for configuration, not interactive prompts.

### 4. Missing Prerequisites Check

Script runs for 5 minutes installing dependencies, then fails because Python 3.12 isn't installed. Check all prerequisites first.

### 5. Hardcoded Absolute Paths

```bash
source /Users/alice/.nvm/nvm.sh
```

Breaks for every other developer. Use `command -v` and portable paths.

### 6. No Verification Step

Setup "completes" but the developer has no idea if it actually worked. Always verify and report results.

---

## Checklist

Before committing a setup script:

- [ ] Starts with `#!/usr/bin/env bash` and `set -euo pipefail`
- [ ] Checks all prerequisites before installing anything
- [ ] Every step is idempotent (safe to run multiple times)
- [ ] Prints progress at each major step
- [ ] Optional steps warn on failure rather than abort
- [ ] Bootstraps in dependency order
- [ ] Waits for infrastructure services to be healthy
- [ ] Ends with verification checks
- [ ] Prints "next steps" for the developer
- [ ] Referenced from README Getting Started section
- [ ] Works without interactive prompts

---

## Related Skills and Agents

- **`authoring-task-runners` skill**: The `setup` task runner command often calls this script — coordinate the two
- **`managing-environment-variables` skill**: Setup scripts create `.env` from `.env.example` — ensure the template is complete
- **`configuring-devcontainers` skill**: For devcontainer projects, setup runs inside `postCreateCommand` — adapt the script for container context
- **`environment_scaffolder` agent**: Generates setup scripts as part of complete environment bootstrapping
- **`environment_auditor` agent**: Checks whether a setup script exists and follows these patterns
