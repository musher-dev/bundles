# Developer Environment Authoring

Provides knowledge and templates for authoring developer environment configurations including devcontainers, task runners, and workspace setup.

## Metadata

- **Domain:** developer-environment
- **Risk Level:** low
- **Technologies:** None (vendor-agnostic)
- **Aliases:** dev-env, devcontainer-authoring, workspace-setup

## Components

### Skills
- `configuring-devcontainers` — devcontainer.json design, features, lifecycle hooks, Docker integration
- `managing-toolchain-versions` — .tool-versions, .nvmrc, asdf/mise, version pinning and conflict resolution
- `authoring-task-runners` — Taskfile.yml, Makefile, justfile, npm scripts patterns and conventions
- `managing-environment-variables` — .env.example templates, direnv, secret classification, startup validation
- `configuring-editor-settings` — .editorconfig, VS Code workspace settings, extensions, debug configs
- `scaffolding-workspace-setup` — setup scripts, onboarding automation, prerequisite verification
- `composing-docker-development` — Docker Compose for local dev, multi-stage Dockerfiles, volume strategy
- `aligning-ci-environments` — local/CI parity, version file reads, frozen lock files, devcontainer reuse
- `auditing-environment-completeness` — gap analysis, 8-dimension scoring, maturity assessment
- `orchestrating-environment-setup` — end-to-end workflow: audit, plan, generate, verify

### Agents
- `environment_auditor` — read-only auditor producing maturity-scored gap analysis reports
- `environment_scaffolder` — generates complete environment configuration from audit findings or from scratch
- `devcontainer_specialist` — deep-focus expert for complex container environments and troubleshooting

### Rules
- `environment-config-patterns` — enforcement rules for devcontainer, Docker, version, env var, and editor config files

### References
- `configuration-file-catalog` — canonical catalog of ~30 developer environment config files
- `tooling-ecosystem` — comparative analysis of version managers, task runners, container tools
- `environment-maturity-model` — 5-level maturity framework (Ad Hoc → CI-Aligned)

### Tools
- None
