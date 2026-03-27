---
name: managing-environment-variables
version: 1.0.0
user-invocable: false
description: >-
  Design environment variable management patterns including .env.example templates,
  direnv (.envrc) configuration, secret vs non-secret classification, startup
  validation, naming conventions, and Docker Compose env_file integration. Use
  when establishing env var conventions, reviewing secret handling, configuring
  direnv layouts, or documenting environment requirements.
  Triggered by: env, .env, environment variables, env.example, envrc, direnv,
  dotenv, secrets, environment configuration, env template, env vars.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Environment Variable Management

Canonical patterns for declaring, documenting, securing, and validating environment variables across development, CI, and production.

## Purpose

Environment variables are the primary mechanism for runtime configuration. Mismanaged env vars cause three categories of failure: missing variables crash the application at startup (or worse, at runtime), leaked secrets compromise security, and undocumented variables block new developer onboarding. This skill establishes patterns that prevent all three.

Use this skill when creating `.env.example` templates, configuring direnv for automatic environment loading, classifying variables as secret vs non-secret, or adding startup validation for required variables.

---

## File Hierarchy

| File | Purpose | Commit? | Contains Secrets? |
|---|---|---|---|
| `.env.example` | Template documenting all variables | Yes | Never |
| `.env` | Active local values | **No** | Usually |
| `.env.local` | Personal overrides | **No** | Usually |
| `.env.test` | Test-specific values | Maybe | Never |
| `.env.development` | Framework dev defaults | Maybe | Never |
| `.envrc` | direnv auto-loading hooks | Yes | Never (loads from .env) |

### Critical Rule

**`.env` files with actual values must never be committed.** Add to `.gitignore`:

```gitignore
# Environment variables
.env
.env.local
.env.*.local
!.env.example
!.env.test
```

---

## .env.example Template

The `.env.example` is the single source of truth for what environment variables the project needs.

### Format Rules

```bash
# =============================================================================
# Application
# =============================================================================

# Server bind address and port
HOST=0.0.0.0
PORT=8000

# Application environment: development | staging | production
APP_ENV=development

# Log level: debug | info | warning | error
LOG_LEVEL=info

# =============================================================================
# Database
# =============================================================================

# PostgreSQL connection URL
# Format: postgresql://user:password@host:port/database
DATABASE_URL=postgresql://dev:dev@localhost:5432/app_dev

# =============================================================================
# External Services (SECRETS — replace with real values)
# =============================================================================

# Stripe API key — get from https://dashboard.stripe.com/apikeys
STRIPE_SECRET_KEY=sk_test_replace_me

# SendGrid API key — get from https://app.sendgrid.com/settings/api_keys
SENDGRID_API_KEY=SG.replace_me

# =============================================================================
# Optional
# =============================================================================

# Redis URL (defaults to localhost if not set)
# REDIS_URL=redis://localhost:6379/0

# Sentry DSN for error tracking (disabled if not set)
# SENTRY_DSN=
```

### Template Rules

| Rule | Rationale |
|---|---|
| Group variables by section with headers | Scannable, organized by concern |
| Include a comment above each variable | Describes purpose, format, and where to get the value |
| Use safe default values for non-secrets | `DATABASE_URL=postgresql://dev:dev@localhost:5432/app_dev` |
| Use obvious placeholder values for secrets | `sk_test_replace_me` — clearly not a real key |
| Comment out optional variables | Distinguishes required from optional at a glance |
| Include links to where secrets are obtained | `get from https://dashboard.stripe.com/apikeys` |

---

## Naming Conventions

| Rule | Convention | Example |
|---|---|---|
| UPPER_SNAKE_CASE | Always | `DATABASE_URL`, `API_KEY` |
| Prefix by service/domain | When multiple services share variables | `STRIPE_SECRET_KEY`, `SENDGRID_API_KEY` |
| Boolean values | `true` / `false` (lowercase) | `ENABLE_CACHE=true` |
| URL values | Full URL with scheme | `DATABASE_URL=postgresql://...` |
| `_URL` suffix | Connection strings | `DATABASE_URL`, `REDIS_URL` |
| `_KEY` / `_SECRET` suffix | API credentials | `API_KEY`, `CLIENT_SECRET` |
| `_PATH` suffix | File system paths | `CERT_PATH=/etc/ssl/cert.pem` |
| `_PORT` suffix | Port numbers | `APP_PORT=8000` |

---

## Secret Classification

### Classification Matrix

| Category | Examples | Storage | Rotation |
|---|---|---|---|
| **Public config** | `APP_ENV`, `PORT`, `LOG_LEVEL` | `.env.example` with real defaults | N/A |
| **Internal secrets** | `DATABASE_URL`, `REDIS_URL` | `.env` locally, secrets manager in prod | Low frequency |
| **External secrets** | `STRIPE_SECRET_KEY`, `AWS_SECRET_KEY` | `.env` locally, secrets manager in prod | Per vendor policy |
| **Signing secrets** | `JWT_SECRET`, `SESSION_SECRET` | `.env` locally, secrets manager in prod | Regular rotation |

### Rules

1. **Never commit actual secret values** — not in `.env`, not in CI config, not in code
2. **Use placeholder values in `.env.example`** — `replace_me`, `sk_test_xxx`
3. **Document where to obtain each secret** — link to dashboard, vault path, or team contact
4. **Separate secret and non-secret variables visually** — use section headers in `.env.example`

---

## direnv Configuration

direnv automatically loads/unloads environment variables when entering/leaving a directory.

### Basic .envrc

```bash
# Load .env file if it exists
dotenv_if_exists

# Or explicitly:
# dotenv .env
# dotenv_if_exists .env.local
```

### Layout-Based .envrc

```bash
# Auto-activate Python virtual environment
layout python3

# Auto-use .tool-versions
use asdf

# Load environment variables
dotenv_if_exists
dotenv_if_exists .env.local

# Computed variables
export DATABASE_URL="postgresql://dev:dev@localhost:5432/${PWD##*/}_dev"
```

### direnv Rules

| Rule | Rationale |
|---|---|
| Commit `.envrc` | It configures the development environment, contains no secrets |
| Use `dotenv_if_exists` not `dotenv` | Doesn't error if `.env` doesn't exist yet |
| Add `.direnv/` to `.gitignore` | direnv caches go here |
| Run `direnv allow` after cloning | Required one-time approval for security |

---

## Startup Validation

Applications should validate required environment variables at startup, failing fast with clear error messages.

### Python (Pydantic Settings)

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    stripe_secret_key: str
    app_env: str = "development"
    port: int = 8000
    log_level: str = "info"

    model_config = {"env_file": ".env"}

# Fails at import time with clear error if required vars missing
settings = Settings()
```

### Node.js (envalid)

```typescript
import { cleanEnv, str, port, url } from 'envalid';

const env = cleanEnv(process.env, {
  DATABASE_URL: url(),
  STRIPE_SECRET_KEY: str(),
  APP_ENV: str({ choices: ['development', 'staging', 'production'], default: 'development' }),
  PORT: port({ default: 8000 }),
  LOG_LEVEL: str({ choices: ['debug', 'info', 'warning', 'error'], default: 'info' }),
});
```

### Validation Rules

| Rule | Rationale |
|---|---|
| Validate at startup, not at use | Fail-fast prevents runtime surprises |
| Type-check values | `PORT` should be a number, `DATABASE_URL` should be a URL |
| Provide sensible defaults for non-secrets | `PORT=8000`, `LOG_LEVEL=info` |
| Never default secrets | Force explicit configuration |
| List all missing variables in one error | Don't make developers fix-and-restart one at a time |

---

## Docker Compose Integration

```yaml
services:
  app:
    env_file:
      - .env
    environment:
      # Override or add variables not in .env
      APP_ENV: development
      DATABASE_URL: postgresql://dev:dev@db:5432/app_dev
```

**Rules:**
- Use `env_file` to load `.env` into the container
- Use `environment` for container-specific overrides (e.g., service hostnames change inside Docker network)
- Container services reference each other by service name (`db`, `redis`), not `localhost`

---

## Anti-Patterns

### 1. Committing .env Files

Secrets in Git history persist forever, even after deletion. Use `.gitignore` and pre-commit hooks to prevent accidental commits.

### 2. Missing .env.example

New developers have no idea what variables are needed. They clone, run the app, and get a cryptic crash. Always maintain a complete `.env.example`.

### 3. Undocumented Variables

```bash
# .env.example
API_KEY=
WEBHOOK_SECRET=
X_CUSTOM_HEADER=
```

No descriptions, no format hints, no links to where values come from. Every variable needs a comment.

### 4. Secrets in Docker Compose

```yaml
environment:
  STRIPE_KEY: sk_live_actual_secret_key_here
```

Docker Compose files are committed. Use `env_file` or interpolation (`${STRIPE_KEY}`) instead.

### 5. No Startup Validation

The app starts successfully with missing env vars and crashes 30 minutes later when a request hits the Stripe integration. Validate all required variables at startup.

---

## Checklist

Before committing environment variable configuration:

- [ ] `.env.example` documents every variable with description and section grouping
- [ ] `.env` and `.env.local` are in `.gitignore`
- [ ] Secret variables have placeholder values, not empty strings
- [ ] Each secret includes a comment on where to obtain it
- [ ] direnv `.envrc` committed (if using direnv)
- [ ] `.direnv/` in `.gitignore`
- [ ] Application validates required variables at startup
- [ ] Docker Compose uses `env_file`, not inline secrets
- [ ] CI uses secrets manager or encrypted env vars, not hardcoded values

---

## Related Skills and Agents

- **`configuring-devcontainers` skill**: Devcontainer lifecycle hooks may need environment variables — coordinate `.env` loading with `postCreateCommand`
- **`authoring-task-runners` skill**: Task runners often pass environment variables — ensure task definitions reference `.env` consistently
- **`aligning-ci-environments` skill**: CI environment variables must match what `.env.example` documents
- **`environment_scaffolder` agent**: Generates `.env.example` and `.envrc` as part of complete environment setup
- **`environment_auditor` agent**: Detects missing `.env.example`, undocumented variables, and committed secrets
