---
name: configuring-devcontainers
version: 1.0.0
user-invocable: false
description: >-
  Design and configure VS Code Dev Container environments including
  devcontainer.json schema, feature selection and pinning, lifecycle hooks
  (postCreateCommand, postStartCommand, postAttachCommand), Docker and Docker
  Compose integration, multi-container setups, port forwarding, mount
  configuration, and GPU passthrough. Use when creating or reviewing
  .devcontainer/ configurations, selecting features, troubleshooting container
  startup, or designing team-shared development containers.
  Triggered by: devcontainer, dev container, devcontainer.json, container
  development, codespace, development container, remote container, container
  feature, postCreateCommand, postStartCommand.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Devcontainer Configuration

Comprehensive guidance for designing reproducible, performant Dev Container environments that work across VS Code, GitHub Codespaces, and Dev Container CLI.

## Purpose

A well-configured devcontainer eliminates "works on my machine" by packaging the entire development environment — runtime, tools, extensions, settings — into a declarative specification. This skill covers the full lifecycle: image selection, feature composition, lifecycle hook design, Docker integration, and performance optimization.

Use this skill when creating a new `.devcontainer/devcontainer.json`, reviewing an existing configuration for completeness, troubleshooting container startup issues, or designing a multi-service development environment.

---

## Image Selection

The base image determines the container's OS, pre-installed tools, and size.

### Decision Matrix

| Project Type | Recommended Image | Size | Notes |
|---|---|---|---|
| Node.js / TypeScript | `mcr.microsoft.com/devcontainers/javascript-node:22` | ~800MB | Includes nvm, yarn, common build tools |
| Python | `mcr.microsoft.com/devcontainers/python:3.12` | ~900MB | Includes pip, venv, common build deps |
| Go | `mcr.microsoft.com/devcontainers/go:1.22` | ~1.1GB | Includes gopls, dlv, common Go tools |
| Rust | `mcr.microsoft.com/devcontainers/rust:1` | ~1.3GB | Includes rustup, cargo, clippy |
| Polyglot / Custom | `mcr.microsoft.com/devcontainers/base:ubuntu` | ~400MB | Minimal base — add languages via features |
| Custom Dockerfile | `build.dockerfile` reference | Varies | Full control; use multi-stage for size |

### Image Pinning Rules

| Rule | Example | Rationale |
|---|---|---|
| Pin to major.minor | `python:3.12` not `python:latest` | Reproducibility across team members |
| Use MCR images when available | `mcr.microsoft.com/devcontainers/...` | Optimized for Dev Container runtime |
| Pin digest for CI | `image@sha256:abc123...` | Exact reproducibility in automated builds |

---

## Feature Selection

Features are composable units of dev tooling installed on top of the base image.

### Core Features

| Feature | ID | When to Include |
|---|---|---|
| Git | `ghcr.io/devcontainers/features/git:1` | Always (usually in base image) |
| GitHub CLI | `ghcr.io/devcontainers/features/github-cli:1` | Repos hosted on GitHub |
| Docker-in-Docker | `ghcr.io/devcontainers/features/docker-in-docker:2` | Projects that build/run containers |
| Docker-outside-of-Docker | `ghcr.io/devcontainers/features/docker-outside-of-docker:1` | Projects that need host Docker socket |
| Node.js | `ghcr.io/devcontainers/features/node:1` | Frontend tooling, npm scripts |
| Python | `ghcr.io/devcontainers/features/python:1` | Python runtime when not in base |
| AWS CLI | `ghcr.io/devcontainers/features/aws-cli:1` | AWS-integrated projects |
| Terraform | `ghcr.io/devcontainers/features/terraform:1` | Infrastructure-as-code projects |
| kubectl + Helm | `ghcr.io/devcontainers/features/kubectl-helm-minikube:1` | Kubernetes projects |

### Feature Pinning Rules

```jsonc
// GOOD: Pin to major version
"features": {
  "ghcr.io/devcontainers/features/node:1": {
    "version": "22"
  }
}

// BAD: Unpinned — breaks reproducibility
"features": {
  "ghcr.io/devcontainers/features/node:latest": {}
}
```

**Rule:** Always pin features to at least a major version. Pin runtime versions within the feature options (e.g., `"version": "22"` for Node).

### Feature Ordering

Features install in the order listed. Place dependencies first:

1. Language runtimes (node, python, go)
2. Build tools (docker, cmake)
3. CLI tools (gh, aws, terraform)
4. Convenience tools (starship, zsh-plugins)

---

## Lifecycle Hooks

Lifecycle hooks run at specific points in the container lifecycle. Choose the right hook for each task.

### Hook Selection Matrix

| Hook | Runs When | Runs As | Use For |
|---|---|---|---|
| `initializeCommand` | Before container build | Host machine | Validating host prerequisites |
| `onCreateCommand` | After container creation (once) | Container user | Installing project dependencies |
| `updateContentCommand` | After create + on content update | Container user | Rebuilding when source changes in prebuilds |
| `postCreateCommand` | After create + updateContent | Container user | One-time setup: git config, db seed, etc. |
| `postStartCommand` | Every container start | Container user | Starting background services |
| `postAttachCommand` | Every VS Code attach | Container user | UI-dependent setup (terminal greeting) |

### Idempotency Rule

Every lifecycle hook command must be safe to run multiple times. Use guard clauses:

```jsonc
{
  "postCreateCommand": "test -f .env || cp .env.example .env && npm install",
  "postStartCommand": "docker compose -f docker-compose.dev.yml up -d"
}
```

### Multi-Command Pattern

For complex setup, use a shell script instead of inline commands:

```jsonc
{
  "postCreateCommand": "bash .devcontainer/setup.sh"
}
```

The script should:
- Set `#!/usr/bin/env bash` and `set -euo pipefail`
- Print progress messages for each step
- Skip steps that are already complete (idempotent)
- Exit non-zero on failure (container build will report the error)

---

## Docker Compose Integration

For projects requiring databases, caches, or other services alongside the dev container.

### Single-Service (Simple)

```jsonc
// .devcontainer/devcontainer.json
{
  "image": "mcr.microsoft.com/devcontainers/python:3.12",
  "features": { /* ... */ },
  "forwardPorts": [8000, 5432],
  "postStartCommand": "docker compose up -d db redis"
}
```

### Multi-Container (Docker Compose)

```jsonc
// .devcontainer/devcontainer.json
{
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  "forwardPorts": [8000, 5432, 6379],
  "shutdownAction": "stopCompose"
}
```

```yaml
# .devcontainer/docker-compose.yml
services:
  app:
    build:
      context: ..
      dockerfile: .devcontainer/Dockerfile
    volumes:
      - ..:/workspace:cached
    command: sleep infinity
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: app_dev
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U dev"]
      interval: 5s
      timeout: 3s
      retries: 5
  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  pgdata:
```

### Volume Mount Performance

| Mount Type | Syntax | Performance | Use For |
|---|---|---|---|
| Cached | `..:/workspace:cached` | Good on macOS | Source code (host → container) |
| Named volume | `node_modules:/workspace/node_modules` | Best | Dependencies (avoid host sync) |
| tmpfs | `tmpfs: { /tmp: { size: '1g' } }` | Fastest | Temporary build artifacts |

**Rule:** Always use named volumes for `node_modules`, `vendor`, `__pycache__`, and other dependency directories to avoid cross-platform filesystem performance issues.

---

## Port Forwarding

```jsonc
{
  "forwardPorts": [3000, 5432, 6379],
  "portsAttributes": {
    "3000": { "label": "App", "onAutoForward": "openBrowser" },
    "5432": { "label": "PostgreSQL", "onAutoForward": "silent" },
    "6379": { "label": "Redis", "onAutoForward": "silent" }
  }
}
```

**Rules:**
- Forward all ports the developer interacts with directly
- Use `"onAutoForward": "silent"` for infrastructure ports (databases, caches)
- Use `"onAutoForward": "openBrowser"` for web app ports
- Label every forwarded port for discoverability

---

## VS Code Customizations

```jsonc
{
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "ms-python.vscode-pylance",
        "charliermarsh.ruff",
        "mtxr.sqltools",
        "mtxr.sqltools-driver-pg"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "charliermarsh.ruff",
        "python.defaultInterpreterPath": "/usr/local/bin/python"
      }
    }
  }
}
```

**Rules:**
- Only include extensions the entire team needs — personal preferences go in user settings
- Pin formatter and linter extensions to ensure consistent behavior
- Set `editor.formatOnSave: true` to catch formatting issues before commit
- Configure language-specific settings within the devcontainer, not in the repo's `.vscode/settings.json`

---

## Complete Example

```jsonc
// .devcontainer/devcontainer.json
{
  "name": "Project Dev Environment",
  "image": "mcr.microsoft.com/devcontainers/python:3.12",
  "features": {
    "ghcr.io/devcontainers/features/node:1": { "version": "22" },
    "ghcr.io/devcontainers/features/github-cli:1": {},
    "ghcr.io/devcontainers/features/docker-in-docker:2": {}
  },
  "forwardPorts": [8000, 5432],
  "portsAttributes": {
    "8000": { "label": "API", "onAutoForward": "notify" },
    "5432": { "label": "PostgreSQL", "onAutoForward": "silent" }
  },
  "postCreateCommand": "pip install -e '.[dev]' && npm install",
  "postStartCommand": "docker compose up -d db",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-python.python",
        "charliermarsh.ruff",
        "mtxr.sqltools"
      ],
      "settings": {
        "editor.formatOnSave": true
      }
    }
  },
  "remoteUser": "vscode"
}
```

---

## Anti-Patterns

### 1. Unpinned Base Image

Using `python:latest` or `node:lts`. Team members get different environments depending on when they build. Always pin to `major.minor`.

### 2. Heavy postCreateCommand

Installing system packages, compiling native extensions, and seeding databases all in one inline command. Use a dedicated setup script or move heavy installations into the Dockerfile/features.

### 3. Missing Named Volumes for Dependencies

Mounting the entire workspace including `node_modules` from the host. On macOS/Windows, filesystem operations on host-mounted volumes are 10-100x slower. Use named volumes for dependency directories.

### 4. Docker-in-Docker When Docker-outside-of-Docker Suffices

`docker-in-docker` runs a full Docker daemon inside the container. `docker-outside-of-Docker` shares the host daemon via socket mount. Use the lighter option unless you need isolated Docker builds.

### 5. No Health Checks on Infrastructure Services

Starting PostgreSQL in `postStartCommand` without waiting for it to be ready. Application code fails on first run. Use `docker compose` with health checks and `depends_on: { condition: service_healthy }`.

### 6. Personal Extensions in Team Config

Adding personal productivity extensions (themes, vim bindings) to the shared devcontainer. These belong in the user's VS Code settings sync, not the team configuration.

---

## Checklist

Before committing a devcontainer configuration:

- [ ] Base image pinned to major.minor version
- [ ] All features pinned to at least major version
- [ ] Runtime versions specified in feature options
- [ ] Lifecycle hooks are idempotent
- [ ] Named volumes used for dependency directories
- [ ] All forwarded ports labeled
- [ ] Infrastructure services have health checks
- [ ] Only team-required extensions included
- [ ] `remoteUser` set (not running as root)
- [ ] Configuration tested with `devcontainer build` and `devcontainer up`

---

## Related Skills and Agents

- **`composing-docker-development` skill**: Deeper guidance on Docker Compose patterns for development — use when the Docker setup goes beyond what devcontainer integration covers
- **`managing-toolchain-versions` skill**: Coordinates runtime version pinning outside of devcontainers — ensures `.tool-versions` and devcontainer feature versions stay aligned
- **`aligning-ci-environments` skill**: Ensures CI uses the same devcontainer or equivalent Docker image — prevents local/CI drift
- **`configuring-editor-settings` skill**: Covers VS Code settings beyond what `customizations.vscode` handles — `.editorconfig`, workspace settings, debug configurations
- **`devcontainer_specialist` agent**: Deep-focus agent for complex devcontainer design, multi-container troubleshooting, and performance optimization
- **`environment_scaffolder` agent**: Generates complete devcontainer configurations as part of full environment scaffolding
