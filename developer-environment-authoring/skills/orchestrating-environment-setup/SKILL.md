---
name: orchestrating-environment-setup
version: 1.0.0
user-invocable: true
description: >-
  Orchestrate end-to-end developer environment scaffolding for a repository by
  first auditing existing configuration, then delegating to specialized skills for
  devcontainers, task runners, version management, editor config, env vars, Docker,
  and setup scripts. Use when bootstrapping a complete developer environment from
  scratch, upgrading an existing setup, or running a full environment audit followed
  by remediation.
  Triggered by: scaffold environment, bootstrap environment, setup dev environment,
  create dev environment, environment setup, dev env, full environment, workspace
  bootstrap, project environment.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Environment Setup Orchestration

Top-level workflow for auditing and scaffolding complete developer environment configurations.

## Purpose

This is the entry point for developer environment work. Rather than requiring the user to invoke individual skills for devcontainers, task runners, version management, and editor settings separately, this skill coordinates the full process: assess what exists, identify what's missing, and generate what's needed — all in one workflow.

Use this skill when a user asks to "set up the dev environment," "bootstrap the project," or "improve the developer experience." It delegates to specialized skills for each concern area.

---

## Workflow

### Phase 1: Audit

Before creating anything, understand what already exists.

1. **Detect project technology profile:**
   - Read `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, or `Gemfile`
   - Identify languages, frameworks, and their versions
   - Detect infrastructure dependencies (database, cache, queue references in code)

2. **Scan existing environment configuration:**
   - Version files: `.tool-versions`, `.nvmrc`, `.python-version`
   - Task runner: `Taskfile.yml`, `Makefile`, `justfile`, `package.json` scripts
   - Environment variables: `.env.example`, `.envrc`
   - Editor: `.editorconfig`, `.vscode/`
   - Containers: `.devcontainer/`, `docker-compose*.yml`, `Dockerfile`
   - Setup: `scripts/setup*`, README "Getting Started"
   - Git: `.gitignore`, `.gitattributes`, `.pre-commit-config.yaml`
   - CI: `.github/workflows/`, `.gitlab-ci.yml`

3. **Score current maturity:**
   - Apply the `auditing-environment-completeness` skill
   - Score across 8 dimensions (0-3 each, 24 total)
   - Map to maturity level (1-5)

4. **Present audit findings** to the user before proceeding:

```
## Current Environment Status

**Maturity:** Level [N] — [Name] ([X]/24)
**Missing:** [brief list of gaps]

### Recommended Actions

1. [Highest priority action]
2. [Next priority action]
3. ...

Shall I proceed with generating the missing configuration?
```

### Phase 2: Plan

After user approval, plan the generation:

1. **Determine target maturity level** based on project context:
   - Solo/small project → Level 3 (Automated)
   - Team project → Level 4 (Containerized)
   - Enterprise/open source → Level 5 (CI-Aligned)

2. **Identify files to create/modify** in dependency order:
   1. `.editorconfig` (no dependencies)
   2. `.tool-versions` or runtime-specific version files (no dependencies)
   3. `.gitignore` updates (no dependencies)
   4. `.env.example` (references detected env vars)
   5. Task runner (references tools and scripts)
   6. `docker-compose.yml` (if infrastructure needed)
   7. `.devcontainer/devcontainer.json` (references compose, versions)
   8. `.vscode/settings.json`, `extensions.json`, `launch.json`
   9. Setup script or setup task
   10. CI alignment updates

3. **Skip files that already exist** unless they have quality issues

### Phase 3: Generate

Delegate to specialized skills in order:

| Step | Skill | Creates |
|---|---|---|
| 1 | `configuring-editor-settings` | `.editorconfig`, `.vscode/*` |
| 2 | `managing-toolchain-versions` | `.tool-versions` or `.nvmrc` |
| 3 | `managing-environment-variables` | `.env.example`, `.envrc` |
| 4 | `authoring-task-runners` | `Taskfile.yml` or equivalent |
| 5 | `composing-docker-development` | `docker-compose.yml`, `Dockerfile`, `.dockerignore` |
| 6 | `configuring-devcontainers` | `.devcontainer/devcontainer.json` |
| 7 | `scaffolding-workspace-setup` | Setup script or task |
| 8 | `aligning-ci-environments` | CI configuration updates |

Each skill provides the rules and patterns; this orchestrator coordinates their execution.

### Phase 4: Verify

After generation:

1. **Cross-file consistency check:**
   - Version in `.tool-versions` matches devcontainer feature version
   - Variables in `.env.example` match `docker-compose.yml` environment entries
   - Task runner commands reference correct tools
   - CI reads from version files
   - `.gitignore` covers `.env`, `.direnv/`, and IDE-specific exclusions

2. **Re-score maturity** to confirm improvement

3. **Present summary** with next steps for the developer

---

## Interaction Modes

### Full Bootstrap (Default)

User asks to set up the environment from scratch. Run the full workflow: audit → plan → generate → verify.

### Gap Fill

User has a partial setup. Run audit, identify specific gaps, and generate only the missing pieces.

### Audit Only

User wants to assess current state without changes. Run Phase 1 only, present the maturity score and findings.

### Targeted Upgrade

User wants to improve a specific dimension (e.g., "add a devcontainer"). Skip the full audit, focus on the requested area and its dependencies.

---

## Related Skills and Agents

- **`auditing-environment-completeness` skill**: Provides the audit framework used in Phase 1
- **All other skills**: Each handles one dimension of environment configuration
- **`environment_auditor` agent**: Can run the audit portion independently
- **`environment_scaffolder` agent**: Can run the generation portion independently
- **`devcontainer_specialist` agent**: Handles complex container scenarios beyond basic scaffolding
