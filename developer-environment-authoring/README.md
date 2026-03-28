# Developer Environment Authoring

Audit, scaffold, and maintain dev environment configs — devcontainers, task runners, version managers, Docker Compose, editor settings, and CI alignment. This bundle finds the gaps in your developer experience setup, generates the missing configuration, and verifies everything works together.

## Quick Start

Install via the [Mush CLI](https://github.com/musher-dev/mush):

```sh
mush bundle install musher-dev/developer-environment-authoring
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Audit our dev environment setup and tell me what's missing
```

## What's Inside

The bundle ships three agents and ten skills following an audit → scaffold → verify workflow.

The **environment_auditor** agent runs a read-only assessment of your repository's developer environment configuration across eight dimensions — devcontainers, toolchain versions, task runners, environment variables, editor settings, Docker setup, CI alignment, and workspace initialization. It produces a gap analysis with a maturity score on a 5-level scale (Ad Hoc → CI-Aligned).

The **environment_scaffolder** agent generates complete configuration files from audit findings or from scratch — `devcontainer.json`, `Taskfile.yml`, `docker-compose.yml`, `.tool-versions`, `.env.example`, `.editorconfig`, and workspace setup scripts.

The **devcontainer_specialist** agent handles complex container environments — multi-container setups, feature composition, lifecycle hook debugging, and Docker-in-Docker configurations.

The `orchestrating-environment-setup` skill coordinates the full workflow: audit the current state, plan what to generate, delegate to the scaffolder, and verify the results.

## Usage Examples

**Audit environment completeness**
```
Audit this repo's developer environment — score devcontainer, toolchain versions, task runner, env vars, editor settings, Docker, and CI alignment
```
The environment_auditor produces a maturity-scored gap analysis with prioritized remediation steps.

**Scaffold a devcontainer**
```
Create a devcontainer configuration for this Python/Node project with PostgreSQL and Redis services
```
The devcontainer_specialist generates `devcontainer.json` with features, lifecycle hooks, and Docker Compose integration.

**Set up the full dev environment**
```
Run the full environment setup — audit what we have, scaffold what's missing, and verify everything works together
```
The orchestrator coordinates audit, planning, generation, and verification end-to-end.

**Align local and CI environments**
```
Make sure our local dev environment matches CI — check version files, lock files, and devcontainer reuse
```
Identifies parity gaps between local development and CI/CD environments.

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `configuring-devcontainers` | devcontainer.json design, features, lifecycle hooks, Docker integration |
| `managing-toolchain-versions` | `.tool-versions`, `.nvmrc`, asdf/mise, version pinning and conflict resolution |
| `authoring-task-runners` | Taskfile.yml, Makefile, justfile, npm scripts patterns |
| `managing-environment-variables` | `.env.example` templates, direnv, secret classification, startup validation |
| `configuring-editor-settings` | `.editorconfig`, VS Code workspace settings, extensions, debug configs |
| `scaffolding-workspace-setup` | Setup scripts, onboarding automation, prerequisite verification |
| `composing-docker-development` | Docker Compose for local dev, multi-stage Dockerfiles, volume strategy |
| `aligning-ci-environments` | Local/CI parity, version file reads, frozen lock files, devcontainer reuse |
| `auditing-environment-completeness` | 8-dimension gap analysis with maturity scoring |
| `orchestrating-environment-setup` | End-to-end workflow: audit → plan → generate → verify |

### Agents

| Agent | Role |
|-------|------|
| `environment_auditor` | Read-only maturity-scored gap analysis across 8 dimensions |
| `environment_scaffolder` | Generates complete environment configuration from audit findings |
| `devcontainer_specialist` | Deep-focus expert for complex container environments and troubleshooting |

### Rules

| Rule | Purpose |
|------|---------|
| `environment-config-patterns` | Enforcement rules for devcontainer, Docker, version, env var, and editor configs |

### References

| Reference | Purpose |
|-----------|---------|
| `configuration-file-catalog` | Canonical catalog of ~30 developer environment config files |
| `tooling-ecosystem` | Comparative analysis of version managers, task runners, container tools |
| `environment-maturity-model` | 5-level maturity framework (Ad Hoc → CI-Aligned) |

</details>

---

**Domain:** developer-environment · **Technologies:** Harness-agnostic · **Aliases:** dev-env, devcontainer-authoring, workspace-setup
