# Tooling Ecosystem Reference

Comparative analysis of developer environment tools — version managers, task runners, container tools, and environment loaders. Consumed by skills when recommending tools and by agents when auditing tool choices.

## Purpose

Developers and teams must choose between competing tools for the same concern. This reference provides objective comparisons to support informed decisions. It does not prescribe a single stack — the right choice depends on team context, existing tooling, and project requirements.

---

## Version Managers

### Polyglot Managers

| Tool | Config File | Speed | Plugin Count | Maintenance Status | Notes |
|---|---|---|---|---|---|
| **mise** | `.tool-versions`, `mise.toml` | Fast (Rust) | 400+ (asdf-compatible) | Active | asdf successor, adds env var management |
| **asdf** | `.tool-versions` | Moderate (Bash) | 400+ | Active | Established ecosystem, wide CI support |
| **aqua** | `aqua.yaml` | Fast (Go) | 2000+ (different registry) | Active | Focused on CLI tools, not runtimes |

### Language-Specific Managers

| Tool | Language | Config File | Speed | Notes |
|---|---|---|---|---|
| **nvm** | Node.js | `.nvmrc` | Slow (Bash) | Most widely adopted for Node |
| **fnm** | Node.js | `.nvmrc`, `.node-version` | Fast (Rust) | Drop-in nvm replacement |
| **volta** | Node.js | `package.json` `volta` | Fast (Rust) | Pins version per project via package.json |
| **pyenv** | Python | `.python-version` | Moderate (C shims) | De facto Python version manager |
| **rbenv** | Ruby | `.ruby-version` | Moderate (C shims) | Standard Ruby version manager |
| **rustup** | Rust | `rust-toolchain.toml` | Fast | Official Rust toolchain manager |
| **sdkman** | JVM (Java, Kotlin, Scala) | `.sdkmanrc` | Moderate | Multi-JVM language support |

### Selection Guidance

| Scenario | Recommendation |
|---|---|
| New polyglot project | **mise** — fastest, most flexible, asdf-compatible |
| Team already on asdf | Stay with **asdf** unless performance is an issue |
| Node-only project, simple needs | **fnm** — fast, .nvmrc compatible |
| Node-only project, wide CI support needed | **nvm** — broadest CI action support |
| Python data science team | **pyenv** + **conda** for complex environments |
| Rust project | **rustup** — official, no alternative needed |

---

## Task Runners

| Tool | Config | Language | Dependencies | Caching | Cross-Platform | Self-Documenting |
|---|---|---|---|---|---|---|
| **Task** | `Taskfile.yml` | YAML | Yes (task deps) | Yes (sources/generates) | Yes | Yes (`task --list`) |
| **Make** | `Makefile` | Make DSL | Yes (file deps) | Yes (file-based) | Partial (Unix-centric) | Manual (`help` target) |
| **just** | `justfile` | Custom DSL | Yes (recipe deps) | No | Yes | Yes (`just --list`) |
| **npm scripts** | `package.json` | JSON | No | No | Partial | Yes (`npm run`) |
| **pnpm scripts** | `package.json` | JSON | No | No | Partial | Yes (`pnpm run`) |
| **deno task** | `deno.json` | JSON | No | No | Yes | Yes (`deno task`) |
| **Earthly** | `Earthfile` | Custom DSL | Yes | Yes (Docker-based) | Yes (containerized) | Yes |

### Selection Guidance

| Scenario | Recommendation |
|---|---|
| Polyglot project, need caching | **Task** — best feature set for the complexity |
| Unix team, C/C++ project | **Make** — native, everyone knows it |
| Developer ergonomics priority | **just** — cleanest syntax |
| Node-only, simple needs | **npm/pnpm scripts** — zero additional tooling |
| Reproducible builds across machines | **Earthly** — containerized execution |

---

## Container Tools

| Tool | License | Platform | Docker API Compatible | Performance | Notes |
|---|---|---|---|---|---|
| **Docker Desktop** | Freemium | macOS, Windows, Linux | Yes (is Docker) | Baseline | Free for personal/small business |
| **Podman** | Apache 2.0 | macOS, Windows, Linux | CLI-compatible | Similar | Rootless by default, no daemon |
| **OrbStack** | Freemium | macOS only | Yes | Faster than Docker Desktop on Mac | Significantly better macOS performance |
| **Rancher Desktop** | Apache 2.0 | macOS, Windows, Linux | Yes (via dockerd or nerdctl) | Similar | Kubernetes built-in |
| **Colima** | MIT | macOS, Linux | Yes (via Lima) | Good | Lightweight, CLI-only |

### Selection Guidance

| Scenario | Recommendation |
|---|---|
| macOS team, performance matters | **OrbStack** — measurably faster file I/O |
| Cross-platform team | **Docker Desktop** — widest support and documentation |
| License-sensitive organization | **Podman** or **Colima** — fully open source |
| Need local Kubernetes | **Rancher Desktop** — Kubernetes included |

---

## Environment Loaders

| Tool | Config File | Auto-Load | Shell Support | Features |
|---|---|---|---|---|
| **direnv** | `.envrc` | Yes (on `cd`) | bash, zsh, fish | Shell hooks, layouts, polyglot support |
| **dotenv-cli** | `.env` | Manual | Any (wraps commands) | Simple env injection |
| **dotenvx** | `.env` | Manual | Any (wraps commands) | Encryption, multi-env support |
| **mise** | `mise.toml` | Yes (on `cd`) | bash, zsh, fish | Env vars alongside tool versions |

### Selection Guidance

| Scenario | Recommendation |
|---|---|
| Auto-load env vars on directory change | **direnv** — battle-tested, supports layouts |
| Already using mise for versions | **mise** — add env vars to `mise.toml` |
| Simple env injection for a command | **dotenv-cli** — `dotenv -- npm test` |
| Need encrypted .env files | **dotenvx** — encryption at rest |

---

## Pre-Commit Frameworks

| Tool | Config File | Language | Speed | Hook Ecosystem |
|---|---|---|---|---|
| **pre-commit** | `.pre-commit-config.yaml` | Python | Moderate | Largest (language-agnostic) |
| **Husky** | `.husky/` | Node.js | Fast | Node ecosystem |
| **lefthook** | `lefthook.yml` | Go | Fast | Growing, parallel execution |
| **lint-staged** | `.lintstagedrc` | Node.js | Fast | Staged-files-only linting |

### Selection Guidance

| Scenario | Recommendation |
|---|---|
| Polyglot project | **pre-commit** — widest hook ecosystem |
| Node-only project | **Husky** + **lint-staged** — standard Node pattern |
| Performance priority | **lefthook** — parallel execution, fast binary |

---

## Tool Combinations by Project Type

### Python API Project
```
Versions:    mise (.tool-versions)
Tasks:       Task (Taskfile.yml) or Make
Container:   Docker Compose + devcontainer
Env vars:    direnv (.envrc) + .env.example
Pre-commit:  pre-commit (.pre-commit-config.yaml)
```

### Node.js Web App
```
Versions:    fnm (.nvmrc)
Tasks:       npm scripts (package.json)
Container:   Docker Compose + devcontainer
Env vars:    dotenv (built into framework) + .env.example
Pre-commit:  Husky + lint-staged
```

### Go Microservice
```
Versions:    mise (.tool-versions) or go.mod only
Tasks:       Task (Taskfile.yml) or Make
Container:   Docker Compose + devcontainer
Env vars:    direnv (.envrc) + .env.example
Pre-commit:  pre-commit or lefthook
```

### Polyglot Monorepo
```
Versions:    mise (.tool-versions at root)
Tasks:       Task (Taskfile.yml) with includes per package
Container:   Docker Compose + devcontainer
Env vars:    direnv (.envrc at root) + .env.example per package
Pre-commit:  pre-commit or lefthook
```
