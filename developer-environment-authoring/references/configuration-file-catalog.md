# Configuration File Catalog

Canonical reference of developer environment configuration files — what each file does, which tools read it, whether to commit it, and relationships between files.

## Purpose

Developer environment configuration is spread across dozens of dotfiles and config files. This catalog is the single reference for what each file does and how it relates to others. Skills and agents consult this catalog to avoid giving contradictory advice about which files to create, commit, or gitignore.

---

## Version Management Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `.tool-versions` | asdf, mise | Yes | Polyglot runtime version pinning |
| `.nvmrc` | nvm, fnm | Yes | Node.js version pinning |
| `.node-version` | fnm, nodenv, Vercel, Netlify | Yes | Node.js version pinning (alternative) |
| `.python-version` | pyenv, mise | Yes | Python version pinning |
| `.ruby-version` | rbenv, mise | Yes | Ruby version pinning |
| `.java-version` | jenv | Yes | Java version pinning |
| `.go-version` | goenv | Yes | Go version pinning |
| `mise.toml` | mise | Yes | mise-specific config with environment variables |
| `.sdkmanrc` | SDKMAN | Yes | JVM ecosystem version pinning |

**Conflict risk:** `.tool-versions` and runtime-specific files (`.nvmrc`, `.python-version`) can declare different versions for the same runtime. Choose one as authoritative and document the choice.

---

## Task Runner Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `Taskfile.yml` | Task (go-task) | Yes | YAML-based task definitions |
| `Makefile` | GNU Make | Yes | Make-based task definitions |
| `justfile` | just | Yes | just command runner definitions |
| `package.json` `scripts` | npm, pnpm, yarn | Yes | Node.js task definitions |
| `deno.json` `tasks` | Deno | Yes | Deno task definitions |
| `Rakefile` | Ruby Rake | Yes | Ruby task definitions |
| `Earthfile` | Earthly | Yes | Containerized build definitions |

**Rule:** One authoritative task runner per project. Secondary runners (e.g., npm scripts wrapping Task) are acceptable as convenience aliases.

---

## Container & Docker Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `Dockerfile` | Docker, Podman | Yes | Container image definition |
| `docker-compose.yml` | Docker Compose | Yes | Multi-service orchestration |
| `docker-compose.override.yml` | Docker Compose | Maybe | Local overrides (commit if team-shared) |
| `docker-compose.dev.yml` | Docker Compose | Yes | Development-specific services |
| `docker-compose.test.yml` | Docker Compose | Yes | Test-specific services |
| `.dockerignore` | Docker, Podman | Yes | Build context exclusion rules |
| `compose.yaml` | Docker Compose v2 | Yes | Modern naming (replaces docker-compose.yml) |

**Note:** `docker-compose.override.yml` is auto-loaded by Docker Compose. If used for personal overrides, gitignore it and document the pattern. If used for shared dev overrides, commit it.

---

## Devcontainer Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `.devcontainer/devcontainer.json` | VS Code, Codespaces, DevContainer CLI | Yes | Container dev environment specification |
| `.devcontainer/Dockerfile` | DevContainer build | Yes | Custom dev container image |
| `.devcontainer/docker-compose.yml` | DevContainer multi-service | Yes | Multi-container dev environment |
| `.devcontainer/setup.sh` | DevContainer lifecycle hooks | Yes | Post-create setup script |

**Rule:** Everything in `.devcontainer/` should be committed. Personal customizations go in VS Code user settings, not devcontainer files.

---

## Editor & IDE Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `.editorconfig` | Most editors (native or plugin) | Yes | Cross-editor formatting basics |
| `.vscode/settings.json` | VS Code | Yes | Workspace settings (team-shared) |
| `.vscode/extensions.json` | VS Code | Yes | Recommended extensions |
| `.vscode/launch.json` | VS Code | Yes | Debug configurations |
| `.vscode/tasks.json` | VS Code | Maybe | VS Code task integration |
| `.vscode/*.code-workspace` | VS Code | Maybe | Multi-root workspace (commit if team pattern) |
| `.idea/` | JetBrains IDEs | Partial | Project settings — see below |
| `.idea/*.xml` (select files) | JetBrains IDEs | Maybe | Code style, inspections (commit shared ones) |

### JetBrains `.idea/` Guidance

| Commit | Gitignore |
|---|---|
| `codeStyles/`, `inspectionProfiles/` | `workspace.xml`, `tasks.xml` |
| `dictionaries/` (shared) | `usage.statistics.xml` |
| `.idea/*.iml` (if standardized) | `shelf/`, `httpRequests/` |

---

## Environment Variable Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `.env` | dotenv, Docker Compose, direnv | **No** | Active environment variables (may contain secrets) |
| `.env.example` | Documentation | Yes | Template with all variables, no real values |
| `.env.local` | dotenv | **No** | Personal environment overrides |
| `.env.test` | Test frameworks | Maybe | Test-specific variables (no secrets) |
| `.env.development` | Framework-specific (Next.js, Vite) | Maybe | Framework dev defaults (no secrets) |
| `.envrc` | direnv | Yes | direnv shell hooks (auto-load .env, layout) |

**Critical rule:** `.env` files containing actual values must never be committed. Only `.env.example` (template) and `.envrc` (direnv hooks) belong in version control.

---

## Linter & Formatter Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `pyproject.toml` | Ruff, Black, mypy, pytest | Yes | Python tool configuration |
| `ruff.toml` | Ruff | Yes | Ruff-specific configuration |
| `.eslintrc.*` / `eslint.config.*` | ESLint | Yes | JavaScript/TypeScript linting |
| `.prettierrc` / `prettier.config.*` | Prettier | Yes | Code formatting rules |
| `.prettierignore` | Prettier | Yes | Files to skip formatting |
| `biome.json` | Biome | Yes | Biome lint + format config |
| `.stylelintrc.*` | Stylelint | Yes | CSS/SCSS linting |
| `rustfmt.toml` | rustfmt | Yes | Rust formatting |
| `clippy.toml` | Clippy | Yes | Rust linting |
| `.golangci.yml` | golangci-lint | Yes | Go linting |
| `.sqlfluff` | SQLFluff | Yes | SQL linting |

**Rule:** All linter and formatter configurations are committed. Personal overrides (if needed) go in editor settings, not config files.

---

## Git Configuration Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `.gitignore` | Git | Yes | Files excluded from version control |
| `.gitattributes` | Git | Yes | Path-specific Git settings (line endings, LFS, diff) |
| `.gitmodules` | Git | Yes | Submodule definitions |
| `.github/` | GitHub | Yes | GitHub-specific workflows, templates, config |
| `.husky/` | Husky | Yes | Git hook scripts |
| `.pre-commit-config.yaml` | pre-commit | Yes | Pre-commit hook definitions |
| `.lintstagedrc.*` | lint-staged | Yes | Staged-file linting configuration |

---

## CI/CD Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `.github/workflows/*.yml` | GitHub Actions | Yes | CI/CD pipeline definitions |
| `.gitlab-ci.yml` | GitLab CI | Yes | GitLab pipeline definition |
| `.circleci/config.yml` | CircleCI | Yes | CircleCI pipeline definition |
| `Jenkinsfile` | Jenkins | Yes | Jenkins pipeline definition |
| `.buildkite/pipeline.yml` | Buildkite | Yes | Buildkite pipeline definition |
| `bitbucket-pipelines.yml` | Bitbucket Pipelines | Yes | Bitbucket pipeline definition |

---

## Dependency & Package Files

| File | Tool(s) | Commit? | Purpose |
|---|---|---|---|
| `package.json` | npm, pnpm, yarn | Yes | Node.js project manifest |
| `package-lock.json` | npm | Yes | Exact dependency tree |
| `pnpm-lock.yaml` | pnpm | Yes | Exact dependency tree |
| `yarn.lock` | Yarn | Yes | Exact dependency tree |
| `requirements.txt` | pip | Yes | Python dependencies (pinned) |
| `requirements-dev.txt` | pip | Yes | Python dev dependencies |
| `pyproject.toml` | pip, Poetry, PDM | Yes | Python project manifest |
| `poetry.lock` | Poetry | Yes | Exact dependency tree |
| `uv.lock` | uv | Yes | Exact dependency tree |
| `go.mod` | Go | Yes | Go module definition |
| `go.sum` | Go | Yes | Go dependency checksums |
| `Gemfile` | Bundler | Yes | Ruby dependencies |
| `Gemfile.lock` | Bundler | Yes | Exact dependency tree |
| `Cargo.toml` | Cargo | Yes | Rust project manifest |
| `Cargo.lock` | Cargo | Yes (apps) / Maybe (libs) | Exact dependency tree |

**Rule:** Lock files are always committed for applications. For libraries, check the ecosystem convention (Rust: commit for binaries, skip for libraries).

---

## Relationship Map

Key dependency and interaction relationships between config files:

```
.tool-versions ──────► CI setup actions (version source)
       │
       ├──────► .devcontainer/devcontainer.json (feature versions must match)
       │
       └──────► .nvmrc / .python-version (must not conflict)

.env.example ────────► .env (developer copies and fills in values)
       │
       └──────► .envrc (direnv loads .env automatically)

Taskfile.yml ────────► CI workflows (same commands)
       │
       └──────► .devcontainer postCreateCommand (calls task setup)

.editorconfig ───────► .vscode/settings.json (overlapping concerns)
       │
       └──────► .prettierrc (EditorConfig settings as fallback)

Dockerfile ──────────► .dockerignore (must be kept in sync)
       │
       └──────► docker-compose.yml (references Dockerfile)
```
