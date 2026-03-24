---
name: researching-repos
description: >
  Explore repository structure, map dependencies, and surface conventions
  before making changes. Use when onboarding to a new repo, investigating
  unfamiliar code, or preparing context for a review.
---

# Researching Repos

Systematically explore a repository to build the context needed for effective code review or contribution.

## Steps

1. **Identify the stack** — read root config files (package.json, pyproject.toml, go.mod) to determine languages, frameworks, and dependency versions.

2. **Map the structure** — walk the top-level directories and identify their roles (source, tests, config, docs, scripts, CI).

3. **Find entry points** — locate main files, route registrations, CLI entrypoints, or event handlers that anchor the request/data flow.

4. **Trace dependencies** — follow imports from entry points to understand which modules depend on which, and where shared types or utilities live.

5. **Discover conventions** — read 2-3 representative source files to learn naming patterns, error handling style, test organization, and commit message format.

6. **Note pain points** — flag areas with high complexity, outdated dependencies, missing tests, or inconsistent patterns.

## Output

Produce a concise summary covering:
- Technology stack and versions
- Directory layout with purpose annotations
- Key entry points and data flow paths
- Established conventions worth preserving
- Areas of concern or technical debt
