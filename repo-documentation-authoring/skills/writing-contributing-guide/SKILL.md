---
name: writing-contributing-guide
version: 1.0.0
user-invocable: false
description: Author CONTRIBUTING.md files that lower the barrier to first contribution with development setup instructions, workflow guidance, coding standards, testing requirements, and PR process documentation. Adapts to project toolchain and language ecosystem. Use when writing contributor guidelines, improving onboarding, or defining PR workflows. Triggered by: CONTRIBUTING, contributing guide, contributor guidelines, development setup, PR process, how to contribute.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Contributing Guide Authoring

## Purpose

Author CONTRIBUTING.md files that transform interested observers into active contributors. The primary goal is reducing the time from "I want to help" to "my first PR is merged" by providing clear, tested setup instructions and unambiguous process documentation.

## Instructions

Follow these steps when authoring or updating a contributing guide:

1. **Read existing file or stub**
   - Find the existing `CONTRIBUTING.md` at the repository root
   - Read its current contents if present
   - If no file exists, suggest running the `scaffolding-repo-documentation` skill first

2. **Detect project toolchain**
   - Inspect package manifests (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Gemfile`)
   - Read CI workflow files (`.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`)
   - Determine: language, package manager, test framework, linter, formatter, and CI system
   - Note any containerized development setup (`Dockerfile`, `docker-compose.yml`, `.devcontainer/`)

3. **Author sections following the framework**
   - Work through each section using the structure in `./references/contributing-framework.md`
   - Tailor the Development Setup section to the detected toolchain
   - Write commands that are specific to the project, not generic placeholders

4. **Define the contribution ladder**
   - Establish clear levels with escalating expectations and access:

   | Level | Who | Expectations | Access |
   |---|---|---|---|
   | First-timer | Anyone | Follow the setup guide, submit a PR | Fork access |
   | Occasional | Has merged 1+ PR | Follow coding standards, respond to review feedback | Fork access |
   | Regular | Has merged 5+ PRs | Review others' PRs, help triage issues | Triage role |
   | Maintainer | Nominated by existing maintainer | Merge PRs, make releases, mentor others | Write access |

5. **Cross-reference related docs**
   - Link to `CODE_OF_CONDUCT.md` in the Welcome section
   - Reference the PR template (`.github/pull_request_template.md`) in the PR Process section
   - Ensure coding standards align with the project's linter and formatter configuration
   - Link to `SECURITY.md` for vulnerability reporting if it exists

6. **Write the file**
   - Verify that every command in the Development Setup section is copy-pasteable and produces the expected result
   - Confirm all referenced files and paths actually exist in the repository
   - Set a consistent tone: welcoming but direct, helpful but not condescending

## Quality Criteria

| Section | Quality Test |
|---|---|
| Welcome | Names specific contribution types? Welcoming tone without being patronizing? |
| Prerequisites | Lists exact tool versions? Includes install links? |
| Development Setup | Every command copy-pasteable? Includes verification step? |
| Workflow | Branch naming convention clear? Commit message format specified? |
| Coding Standards | References automated tooling (linter, formatter)? Not just "follow best practices"? |
| Testing | Explains how to run tests? States coverage expectations? |
| PR Process | Defines merge criteria? Sets review timeline expectations? |

## Toolchain-Specific Patterns

Adapt the Development Setup section to the project's ecosystem:

- **Node.js**: nvm for version management, npm/yarn/pnpm for dependencies, jest/vitest for testing, eslint/prettier for linting and formatting. Include `node -v` and `npm test` verification steps.
- **Python**: pyenv for version management, pip/poetry/uv for dependencies, pytest for testing, ruff/black for formatting. Include `python --version` and `pytest` verification steps.
- **Go**: `go install` for tooling, `go test ./...` for testing, golangci-lint for linting. Include `go version` and `go test ./...` verification steps.
- **Rust**: rustup for toolchain management, `cargo test` for testing, clippy for linting, rustfmt for formatting. Include `rustc --version` and `cargo test` verification steps.

## Guidelines

- **DO**: Test every command in the Development Setup section on a clean machine
- **DO**: Include a "verification" step after setup (e.g., "run tests to confirm everything works")
- **DO**: Link to the CODE_OF_CONDUCT.md
- **DO**: Specify response time expectations for PR reviews
- **DON'T**: Assume the reader has the project's toolchain pre-installed
- **DON'T**: Write "follow best practices" without specifying which practices
- **DON'T**: Require CLA signing without explaining what it is and why
- **DON'T**: Gate first-time contributions behind excessive process

## Edge Cases

- **Documentation-only contributions**: Provide a separate, simplified workflow that doesn't require full dev setup -- contributors should be able to edit docs with just a text editor and Git
- **Monorepo**: Explain which package(s) to set up and how to scope tests to the relevant package rather than running the entire suite
- **External dependencies (databases, APIs)**: Document how to set up local development dependencies using Docker Compose or provide mock/stub alternatives
- **No CI yet**: Note that tests are run manually and explain exactly how; include the expected output so contributors can self-verify

## Supporting Files

- [Contributing Framework](./references/contributing-framework.md)
