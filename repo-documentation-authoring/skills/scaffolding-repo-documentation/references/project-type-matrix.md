# Project Type Matrix

Maps each community health file to its recommendation level per project type. Use this matrix to determine which files to scaffold for a given repository.

## Recommendation Levels

- **Required**: Every project of this type should have this file
- **Recommended**: Most projects benefit; scaffold by default but note it can be removed
- **Optional**: Scaffold only if the user explicitly requests it
- **N/A**: Not applicable for this project type

## Matrix

| File | Library | Application | Framework | CLI Tool | Org `.github` |
|---|---|---|---|---|---|
| README.md | Required | Required | Required | Required | Required |
| CONTRIBUTING.md | Required | Recommended | Required | Required | Required |
| CODE_OF_CONDUCT.md | Required | Optional | Required | Required | Required |
| SECURITY.md | Required | Required | Required | Required | Required |
| SUPPORT.md | Recommended | Optional | Recommended | Recommended | Required |
| CHANGELOG.md | Required | Recommended | Required | Required | N/A |
| LICENSE | Required | Required | Required | Required | Recommended |
| GOVERNANCE.md | Optional | Optional | Recommended | Optional | Required |
| ISSUE_TEMPLATE/ | Required | Optional | Required | Recommended | Required |
| PR_TEMPLATE | Required | Recommended | Required | Recommended | Required |
| CODEOWNERS | Recommended | Recommended | Recommended | Recommended | Required |
| FUNDING.yml | Optional | Optional | Optional | Optional | Optional |

## Project Type Detection Signals

### Library
A reusable package published to a registry and consumed by other projects.

**Signals**: `package.json` with no `bin` field, `setup.py`/`pyproject.toml` with package metadata, `Cargo.toml` with `[lib]`, `go.mod` without `main.go` in root, presence of `src/` or `lib/` without `app/` or `server/`.

### Application
A deployable service, web app, or standalone program.

**Signals**: `Dockerfile`, `docker-compose.yml`, `Procfile`, deployment configs (`vercel.json`, `fly.toml`, `render.yaml`), `package.json` with `start` script, database migration directories.

### Framework
A foundational toolkit that others build upon, with plugin/extension architecture.

**Signals**: `examples/` directory, plugin/extension system, `docs/` with guide structure, extensive `README.md` with "Getting Started" and "Concepts" sections, template/generator commands.

### CLI Tool
A command-line program installed globally or run via `npx`/`pipx`.

**Signals**: `package.json` with `bin` field, `setup.py` with `console_scripts`, `Cargo.toml` with `[[bin]]`, `main.go` in root, argument parser imports (`argparse`, `clap`, `commander`).

### Org `.github` Repository
A repository named `.github` under a GitHub organization, providing default community health files for all repos in the org.

**Signals**: Repository name is literally `.github`, typically contains only community health files and profile README.

## Ambiguity Resolution

When a project matches multiple types:
1. Ask the user which type best describes their project
2. If they are unsure, default to **Library** (the most documentation-heavy type)
3. A project can combine types (e.g., a framework that is also published as a library) — use the union of Required files from both types
