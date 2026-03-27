---
name: authoring-task-runners
version: 1.0.0
user-invocable: false
description: >-
  Design and configure task runner configurations including Taskfile.yml (Task),
  Makefile (Make), justfile (just), npm/pnpm scripts, and deno tasks. Covers task
  naming conventions, dependency ordering, cross-platform compatibility, variable
  passing, help/documentation generation, and common task patterns for development
  workflows. Use when creating or reviewing project task definitions, choosing
  between task runners, or standardizing development commands across a team.
  Triggered by: taskfile, Taskfile.yml, makefile, Makefile, justfile, npm scripts,
  task runner, make target, dev commands, build automation, pnpm scripts, deno task.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Task Runner Authoring

Canonical patterns for defining, organizing, and documenting development workflow tasks across popular task runners.

## Purpose

Every project needs a consistent set of commands: `setup`, `dev`, `test`, `lint`, `build`, `clean`. Without a task runner, these live in README snippets, tribal knowledge, and shell history. A well-structured task runner is the executable documentation of your development workflow — discoverable, composable, and portable.

Use this skill when adding a task runner to a project, migrating between runners, reviewing task definitions for completeness, or establishing team conventions for task naming and organization.

---

## Task Runner Selection

### Decision Matrix

| Runner | Config File | Language | Strengths | Weaknesses | Best For |
|---|---|---|---|---|---|
| **Task** | `Taskfile.yml` | YAML | Dependency graph, file-based caching, cross-platform | Requires Go binary install | Polyglot projects, CI/CD |
| **Make** | `Makefile` | Make DSL | Universal availability, battle-tested | Tab-sensitivity, limited cross-platform | Unix-centric teams, C/C++ |
| **just** | `justfile` | Custom DSL | Clean syntax, no tab issues, argument passing | Less ecosystem adoption | Developer ergonomics focus |
| **npm scripts** | `package.json` | JSON | Zero install in Node projects, lifecycle hooks | No dependency graph, verbose for complex tasks | Node-only projects |
| **pnpm scripts** | `package.json` | JSON | Same as npm + workspace support | Same as npm | pnpm monorepos |
| **deno task** | `deno.json` | JSON | Zero install in Deno projects | Deno-only ecosystem | Deno projects |

### Recommendation

- **New polyglot project**: Use **Task** (`Taskfile.yml`) — best balance of features and readability
- **Existing Node project**: Use **npm/pnpm scripts** for simple tasks, add **Task** when complexity grows
- **Unix/Linux team**: **Make** is fine if the team knows it — don't migrate working Makefiles without cause
- **Developer ergonomics priority**: **just** for its clean syntax and argument handling

---

## Universal Task Set

Every project should define these tasks regardless of runner:

| Task | Purpose | Example Command |
|---|---|---|
| `setup` | Bootstrap from zero to working environment | Install deps, create .env, seed DB |
| `dev` | Start development server with hot-reload | `uvicorn app:main --reload` |
| `test` | Run full test suite | `pytest` / `npm test` |
| `test:unit` | Run unit tests only | `pytest tests/unit` |
| `test:integration` | Run integration tests only | `pytest tests/integration` |
| `lint` | Run all linters | `ruff check . && eslint .` |
| `lint:fix` | Auto-fix linting issues | `ruff check --fix . && eslint --fix .` |
| `format` | Format all code | `ruff format . && prettier --write .` |
| `format:check` | Check formatting without modifying | `ruff format --check .` |
| `build` | Produce production artifacts | `docker build .` / `npm run build` |
| `clean` | Remove generated files and caches | `rm -rf dist/ .pytest_cache/` |
| `db:migrate` | Run pending database migrations | `atlas migrate apply` |
| `db:seed` | Seed database with development data | `python scripts/seed.py` |

---

## Task Naming Conventions

| Rule | Convention | Example |
|---|---|---|
| Verb-first | Start with an action verb | `test`, `build`, `lint`, `deploy` |
| Colon namespacing | Group related tasks with `:` | `test:unit`, `test:integration`, `db:migrate` |
| Lowercase | All lowercase, no camelCase | `lint:fix` not `lintFix` |
| Descriptive | Clear without reading the command | `db:migrate` not `dm` |

### Namespace Hierarchy

```
setup          # Top-level: environment bootstrapping
dev            # Top-level: start development
test           # Top-level: all tests
test:unit      # Namespaced: specific test scope
test:e2e       # Namespaced: specific test scope
lint           # Top-level: all linting
lint:fix       # Namespaced: linting variant
format         # Top-level: all formatting
format:check   # Namespaced: formatting variant
db:migrate     # Namespaced: database operations
db:seed        # Namespaced: database operations
db:reset       # Namespaced: database operations
docker:build   # Namespaced: container operations
docker:push    # Namespaced: container operations
```

---

## Taskfile.yml (Task)

### Structure

```yaml
version: '3'

vars:
  PYTHON: python3
  NODE: node

tasks:
  setup:
    desc: Bootstrap development environment
    cmds:
      - "{{.PYTHON}} -m pip install -e '.[dev]'"
      - npm install
      - task: db:migrate
    preconditions:
      - sh: command -v {{.PYTHON}}
        msg: "Python 3 is required. Install it via .tool-versions."

  dev:
    desc: Start development server
    deps: [db:migrate]
    cmds:
      - uvicorn app.main:app --reload --port 8000

  test:
    desc: Run full test suite
    cmds:
      - pytest {{.CLI_ARGS}}

  test:unit:
    desc: Run unit tests
    cmds:
      - pytest tests/unit {{.CLI_ARGS}}

  lint:
    desc: Run all linters
    cmds:
      - ruff check .
      - eslint .

  lint:fix:
    desc: Auto-fix linting issues
    cmds:
      - ruff check --fix .
      - eslint --fix .

  format:
    desc: Format all code
    cmds:
      - ruff format .
      - prettier --write .

  build:
    desc: Build production Docker image
    cmds:
      - docker build -t myapp:latest .
    sources:
      - Dockerfile
      - src/**/*
      - requirements.txt
    generates:
      - .task/checksum/build

  clean:
    desc: Remove generated files
    cmds:
      - rm -rf dist/ .pytest_cache/ .ruff_cache/ node_modules/.cache/

  db:migrate:
    desc: Apply pending database migrations
    cmds:
      - atlas migrate apply --url "$DATABASE_URL"

  db:seed:
    desc: Seed database with development data
    cmds:
      - "{{.PYTHON}} scripts/seed.py"
    deps: [db:migrate]
```

### Task Key Features

| Feature | Syntax | Purpose |
|---|---|---|
| `desc` | `desc: "description"` | Shown in `task --list` |
| `deps` | `deps: [task1, task2]` | Run dependencies before this task |
| `preconditions` | `preconditions: [{ sh: "cmd" }]` | Validate prerequisites |
| `sources` / `generates` | File globs | Skip task if inputs unchanged |
| `vars` | `vars: { KEY: value }` | Reusable variables |
| `{{.CLI_ARGS}}` | Template variable | Pass arguments from command line |
| `internal: true` | Boolean | Hide from `task --list` |

---

## Makefile

### Structure

```makefile
.PHONY: setup dev test lint lint-fix format build clean db-migrate db-seed help

PYTHON := python3
NODE := node

## setup: Bootstrap development environment
setup:
	$(PYTHON) -m pip install -e '.[dev]'
	npm install
	$(MAKE) db-migrate

## dev: Start development server
dev: db-migrate
	uvicorn app.main:app --reload --port 8000

## test: Run full test suite
test:
	pytest $(ARGS)

## test-unit: Run unit tests
test-unit:
	pytest tests/unit $(ARGS)

## lint: Run all linters
lint:
	ruff check .
	eslint .

## lint-fix: Auto-fix linting issues
lint-fix:
	ruff check --fix .
	eslint --fix .

## format: Format all code
format:
	ruff format .
	prettier --write .

## build: Build production Docker image
build:
	docker build -t myapp:latest .

## clean: Remove generated files
clean:
	rm -rf dist/ .pytest_cache/ .ruff_cache/ node_modules/.cache/

## db-migrate: Apply pending database migrations
db-migrate:
	atlas migrate apply --url "$$DATABASE_URL"

## db-seed: Seed database with development data
db-seed: db-migrate
	$(PYTHON) scripts/seed.py

## help: Show this help message
help:
	@grep -E '^## ' $(MAKEFILE_LIST) | sed 's/^## //' | column -t -s ':'
```

### Makefile Rules

| Rule | Rationale |
|---|---|
| Always declare `.PHONY` | Prevents conflicts with files named like targets |
| Use `##` comments for self-documenting help | Enables auto-generated `make help` |
| Use variables for tool paths | `$(PYTHON)` allows override: `make test PYTHON=python3.11` |
| Use `$(MAKE)` for recursive make | Ensures correct make binary is used |
| Escape `$` as `$$` in shell commands | Make interprets `$` as variable expansion |

---

## justfile

### Structure

```just
# Bootstrap development environment
setup:
    python3 -m pip install -e '.[dev]'
    npm install
    just db-migrate

# Start development server
dev: db-migrate
    uvicorn app.main:app --reload --port 8000

# Run full test suite
test *args:
    pytest {{ args }}

# Run unit tests
test-unit *args:
    pytest tests/unit {{ args }}

# Run all linters
lint:
    ruff check .
    eslint .

# Auto-fix linting issues
lint-fix:
    ruff check --fix .
    eslint --fix .

# Format all code
format:
    ruff format .
    prettier --write .

# Build production Docker image
build:
    docker build -t myapp:latest .

# Remove generated files
clean:
    rm -rf dist/ .pytest_cache/ .ruff_cache/ node_modules/.cache/

# Apply pending database migrations
db-migrate:
    atlas migrate apply --url "$DATABASE_URL"

# Seed database with development data
db-seed: db-migrate
    python3 scripts/seed.py
```

### just Advantages Over Make

| Feature | Make | just |
|---|---|---|
| Indentation | Must use tabs | Spaces or tabs |
| Arguments | Awkward `$(ARGS)` pattern | Native `*args` parameter |
| String interpolation | `$$` escaping required | Standard `{{ var }}` |
| Cross-platform | Shell-dependent | Consistent across OS |
| Self-documentation | Requires `help` target hack | Built-in `just --list` |

---

## npm / pnpm Scripts

### Structure

```json
{
  "scripts": {
    "setup": "npm install && npm run db:migrate",
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "test": "vitest",
    "test:unit": "vitest run tests/unit",
    "test:e2e": "playwright test",
    "lint": "eslint . && tsc --noEmit",
    "lint:fix": "eslint --fix .",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "clean": "rm -rf .next/ dist/ node_modules/.cache/",
    "db:migrate": "prisma migrate dev",
    "db:seed": "prisma db seed"
  }
}
```

### npm Script Rules

| Rule | Rationale |
|---|---|
| Use `:` for namespacing | `test:unit`, `test:e2e` — consistent with npm conventions |
| Avoid `pre`/`post` hooks for critical tasks | Implicit execution order surprises developers |
| Use `cross-env` for environment variables | `cross-env NODE_ENV=test vitest` works on Windows |
| Keep commands short | Complex logic goes in a script file, not a JSON string |

---

## Anti-Patterns

### 1. No Self-Documentation

Tasks without descriptions. Running `task --list` or `make help` shows nothing useful. Always add `desc:` (Task) or `##` comments (Make).

### 2. Long Inline Commands

```json
"scripts": {
  "deploy": "docker build -t app . && docker tag app registry.example.com/app:$(git rev-parse --short HEAD) && docker push registry.example.com/app:$(git rev-parse --short HEAD) && kubectl set image deployment/app app=registry.example.com/app:$(git rev-parse --short HEAD)"
}
```

Move complex commands to a shell script and call it from the task runner.

### 3. Missing Dependencies Between Tasks

`db:seed` runs without first running `db:migrate`. Use task runner dependency features (`deps:` in Task, prerequisites in Make) to enforce ordering.

### 4. Platform-Specific Commands Without Alternatives

Using `rm -rf` in npm scripts fails on Windows. Use `rimraf` or `shx rm -rf` for cross-platform compatibility when the team includes Windows developers.

### 5. Duplicated Commands Across Runners

Having both a Makefile and a Taskfile.yml with the same tasks but subtly different commands. Choose one runner and make it authoritative.

---

## Checklist

Before committing task runner configuration:

- [ ] All universal tasks defined (setup, dev, test, lint, format, build, clean)
- [ ] Every task has a description
- [ ] Task naming follows verb-first, colon-namespaced convention
- [ ] Dependencies between tasks declared explicitly
- [ ] Preconditions validate required tools
- [ ] Complex commands extracted to scripts
- [ ] Cross-platform compatibility considered
- [ ] `help` target or equivalent documentation available
- [ ] CI uses the same task runner commands as local development

---

## Related Skills and Agents

- **`scaffolding-workspace-setup` skill**: The `setup` task is the executable counterpart to the setup script — this skill covers the broader bootstrapping context
- **`managing-environment-variables` skill**: Task runners often need environment variables — this skill covers `.env` management and direnv integration
- **`aligning-ci-environments` skill**: CI should call the same task runner commands — prevents drift between local and CI definitions
- **`environment_scaffolder` agent**: Generates task runner configuration as part of complete environment setup
- **`environment_auditor` agent**: Checks for missing tasks, undocumented commands, and runner consistency
