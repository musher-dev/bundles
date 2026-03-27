---
paths:
  - "**/.devcontainer/**"
  - "**/devcontainer.json"
  - "**/.tool-versions"
  - "**/.nvmrc"
  - "**/.python-version"
  - "**/.editorconfig"
  - "**/.env.example"
  - "**/.envrc"
  - "**/Taskfile.yml"
  - "**/Makefile"
  - "**/justfile"
  - "**/docker-compose*.yml"
  - "**/compose.yaml"
  - "**/.vscode/*.json"
  - "**/.dockerignore"
---

# Environment Configuration Patterns

Enforcement rules applied when editing developer environment configuration files.

## Environment Variables

- **MUST NOT** commit `.env` files containing actual values — only `.env.example` (template) and `.envrc` (direnv hooks) belong in version control
- **MUST** use placeholder values for secrets in `.env.example` (e.g., `sk_test_replace_me`, not empty strings)
- **MUST** include a descriptive comment above each variable in `.env.example`

## Version Pinning

- **MUST** pin versions to exact `major.minor.patch` in `.tool-versions`, `.nvmrc`, and `.python-version` — never use `latest`, `lts`, or version ranges
- **MUST** ensure devcontainer feature versions are pinned to at least major version
- **SHOULD** keep version declarations consistent across multiple version files (e.g., `.tool-versions` and `.nvmrc` must agree on Node version)

## Devcontainer

- **MUST** set `remoteUser` to a non-root user in devcontainer.json
- **MUST** pin base image to `major.minor` version (not `latest`)
- **MUST** pin features to at least major version with runtime versions in options
- **SHOULD** use named volumes for dependency directories (node_modules, .venv) to avoid host filesystem performance issues

## Docker Compose

- **MUST** include health checks on all infrastructure services (database, cache, message queue)
- **MUST** use `depends_on` with `condition: service_healthy` for service ordering
- **MUST** set `start_period` on health checks for slow-starting services
- **MUST NOT** hardcode secrets in docker-compose files — use `env_file` or `${VARIABLE}` interpolation
- **SHOULD** use named volumes for persistent data (database, cache) and dependency directories

## Task Runner

- **MUST** include descriptions for all tasks (Taskfile `desc:`, Makefile `##` comments)
- **SHOULD** define the universal task set: setup, dev, test, lint, format, build, clean

## Editor Configuration

- **MUST** set `root = true` in `.editorconfig`
- **MUST** set `insert_final_newline = true` in `.editorconfig`
- **MUST** set `trim_trailing_whitespace = true` in `.editorconfig` (with `false` override for Markdown)
- **MUST** set `indent_style = tab` for Makefile sections in `.editorconfig`
- **MUST NOT** include user-specific paths (home directories, personal tool paths) in `.vscode/settings.json`
- **MUST NOT** include personal preference settings (themes, fonts, keybindings) in `.vscode/settings.json`

## Dockerignore

- **MUST** exclude `.git`, `node_modules`, `.env`, and IDE directories from Docker build context
- **SHOULD** exclude test directories and documentation from production image builds

## Taskfile.yml Specific

- **MUST** use `version: '3'` in Taskfile.yml
- **SHOULD** use `preconditions` to validate required tools before running tasks
