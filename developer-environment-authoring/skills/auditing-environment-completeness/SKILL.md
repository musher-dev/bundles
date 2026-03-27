---
name: auditing-environment-completeness
version: 1.0.0
user-invocable: false
description: >-
  Audit a repository for developer environment configuration completeness covering
  version pinning, editor config, task runner, env var documentation, setup scripts,
  devcontainer presence, Docker configuration, and CI alignment. Produces a gap
  analysis with maturity scoring and prioritized recommendations. Use when assessing
  a project's developer experience maturity, onboarding to a new codebase, or
  evaluating developer environment quality.
  Triggered by: environment audit, dev experience audit, DX audit, environment
  completeness, environment review, onboarding review, dev setup review, environment
  maturity, environment assessment, developer experience.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Environment Completeness Auditing

Structured framework for evaluating the completeness, consistency, and maturity of a project's developer environment configuration.

## Purpose

A comprehensive developer environment audit answers two questions: "What's missing?" and "What should we fix first?" This skill provides the detection methods, scoring rubric, and output format for assessing environment configuration quality. It feeds directly into the `environment_auditor` agent's workflow and produces actionable gap analyses.

Use this skill when joining a new project (to understand its environment quality), conducting periodic environment reviews, or preparing a remediation roadmap for developer experience improvements.

---

## Audit Dimensions

The audit evaluates 8 dimensions, each with specific detection methods and scoring criteria.

### 1. Version Pinning

**What to look for:**

| Check | Detection | Severity if Missing |
|---|---|---|
| Any version file exists | Glob `.tool-versions`, `.nvmrc`, `.python-version` | High |
| Versions are exact (major.minor.patch) | Read version files, check for `latest`, `lts`, ranges | Medium |
| Version files are consistent | Compare values across multiple version files | High |
| `engines` / `requires-python` set | Read `package.json`, `pyproject.toml` | Low |
| CI reads from version files | Grep CI config for `node-version-file`, `python-version-file` | Medium |

### 2. Task Runner

**What to look for:**

| Check | Detection | Severity if Missing |
|---|---|---|
| Task runner config exists | Glob `Taskfile.yml`, `Makefile`, `justfile`, check `package.json` scripts | High |
| Universal tasks defined | Check for setup, dev, test, lint, build, clean | Medium |
| Tasks have descriptions | Read task runner config, check for desc/comments | Low |
| CI uses same task commands | Grep CI config for task runner invocations | Medium |

### 3. Environment Variables

**What to look for:**

| Check | Detection | Severity if Missing |
|---|---|---|
| `.env.example` exists | Glob `.env.example` | High |
| Variables are documented | Read `.env.example`, check for comments | Medium |
| `.env` is gitignored | Read `.gitignore`, check for `.env` entry | Critical |
| Secrets use placeholders | Read `.env.example`, check for real-looking secrets | Critical |
| direnv configured | Glob `.envrc` | Low |

### 4. Editor Configuration

**What to look for:**

| Check | Detection | Severity if Missing |
|---|---|---|
| `.editorconfig` exists | Glob `.editorconfig` | Medium |
| VS Code settings exist | Glob `.vscode/settings.json` | Low |
| Recommended extensions listed | Glob `.vscode/extensions.json` | Low |
| Debug configurations exist | Glob `.vscode/launch.json` | Low |
| No personal settings in workspace | Read `.vscode/settings.json`, check for themes/fonts | Medium |

### 5. Container Configuration

**What to look for:**

| Check | Detection | Severity if Missing |
|---|---|---|
| Devcontainer exists | Glob `.devcontainer/devcontainer.json` | Low-Medium |
| Docker Compose exists (if infra needed) | Glob `docker-compose*.yml`, `compose.yaml` | Medium |
| `.dockerignore` exists (if Dockerfile exists) | Glob `.dockerignore` | Medium |
| Health checks on services | Read docker-compose, check for `healthcheck` | High |
| Named volumes for dependencies | Read docker-compose, check volume config | Medium |

### 6. Setup & Onboarding

**What to look for:**

| Check | Detection | Severity if Missing |
|---|---|---|
| Setup script or setup task exists | Glob `scripts/setup*`, check task runner for `setup` | Medium |
| README has Getting Started section | Read `README.md`, search for "Getting Started" or "Setup" | Medium |
| Prerequisites documented | Read README, check for prerequisites/requirements list | Medium |

### 7. Git Configuration

**What to look for:**

| Check | Detection | Severity if Missing |
|---|---|---|
| `.gitignore` covers common patterns | Read `.gitignore`, check for node_modules, .env, __pycache__ | High |
| `.gitattributes` sets line endings | Glob `.gitattributes` | Low |
| Pre-commit hooks configured | Glob `.pre-commit-config.yaml`, `.husky/`, `lefthook.yml` | Low |

### 8. CI Alignment

**What to look for:**

| Check | Detection | Severity if Missing |
|---|---|---|
| CI config exists | Glob `.github/workflows/*.yml`, `.gitlab-ci.yml` | Medium |
| CI reads version files | Grep CI config for version-file references | Medium |
| CI uses frozen dependency install | Grep for `npm ci`, `--frozen-lockfile`, `--immutable` | High |
| CI service versions match local | Compare docker-compose image versions with CI service images | Medium |

---

## Scoring Rubric

### Per-Dimension Scoring

| Score | Level | Criteria |
|---|---|---|
| 0 | Absent | Dimension not addressed at all |
| 1 | Minimal | Basic file exists but incomplete or misconfigured |
| 2 | Adequate | Covers the essentials, minor gaps |
| 3 | Complete | Fully configured, follows best practices |

### Maturity Levels

| Level | Score Range | Description |
|---|---|---|
| **1 — Ad Hoc** | 0-6 | Tribal knowledge, no standardized environment |
| **2 — Documented** | 7-12 | README instructions exist, some config files |
| **3 — Automated** | 13-18 | Setup script, task runner, env template |
| **4 — Containerized** | 19-22 | Devcontainer or Docker-based development |
| **5 — CI-Aligned** | 23-24 | Full parity between local, container, and CI |

---

## Output Format

```
# Developer Environment Audit Report

**Repository:** [name]
**Date:** YYYY-MM-DD
**Languages:** [detected]
**Infrastructure:** [detected services]

---

## Maturity Score: [X]/24 — Level [N]: [Level Name]

| Dimension | Score | Notes |
|---|---|---|
| Version Pinning | X/3 | [brief note] |
| Task Runner | X/3 | [brief note] |
| Environment Variables | X/3 | [brief note] |
| Editor Configuration | X/3 | [brief note] |
| Container Configuration | X/3 | [brief note] |
| Setup & Onboarding | X/3 | [brief note] |
| Git Configuration | X/3 | [brief note] |
| CI Alignment | X/3 | [brief note] |

---

## Findings

### Critical
- [Findings that pose security or data risk]

### High
- [Findings that cause developer friction daily]

### Medium
- [Findings that slow onboarding or cause occasional issues]

### Low
- [Nice-to-have improvements]

---

## Recommendations

### Immediate (Fix This Week)
1. [Critical and high-severity items]

### Short-Term (This Sprint)
2. [Medium-severity items with high impact]

### Backlog
3. [Low-severity improvements]

---

## Files to Create or Modify

| Action | File | Purpose |
|---|---|---|
| Create | `.editorconfig` | Cross-editor formatting |
| Create | `.env.example` | Environment variable documentation |
| Modify | `.gitignore` | Add missing exclusions |
| ... | ... | ... |
```

---

## Detection Shortcuts

Quick commands for initial assessment:

```bash
# Version files
ls -la .tool-versions .nvmrc .python-version .ruby-version 2>/dev/null

# Task runner
ls -la Taskfile.yml Makefile justfile 2>/dev/null

# Environment
ls -la .env .env.example .envrc 2>/dev/null

# Editor
ls -la .editorconfig .vscode/settings.json .vscode/extensions.json 2>/dev/null

# Container
ls -la .devcontainer/devcontainer.json docker-compose*.yml Dockerfile 2>/dev/null

# Git
ls -la .gitignore .gitattributes .pre-commit-config.yaml 2>/dev/null

# CI
ls -la .github/workflows/*.yml .gitlab-ci.yml 2>/dev/null
```

---

## Related Skills and Agents

- **All other skills in this bundle**: This skill references rules and checklists from every other skill to evaluate completeness
- **`environment_auditor` agent**: Executes this skill as part of a comprehensive read-only audit
- **`environment_scaffolder` agent**: Receives this skill's findings and generates the missing files
- **`references/environment-maturity-model.md`**: Provides the 5-level maturity framework used for scoring
