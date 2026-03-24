---
name: researching-codebases
description: >
  Explore repository structure, dependencies, and conventions to build
  deep understanding of a codebase. Use when onboarding to a new project,
  investigating unfamiliar code, or mapping dependencies before making changes.
---

# Researching Codebases

When asked to explore or understand a codebase, follow this structured approach to build a comprehensive mental model before answering questions or suggesting changes.

## Phase 1: Orientation

Identify the project type and technology stack:

1. Read the root-level configuration files (package.json, pyproject.toml, go.mod, Cargo.toml, etc.)
2. Check for a README or CONTRIBUTING guide
3. Identify the build system, test framework, and linting configuration
4. Note the language version and major dependency versions

## Phase 2: Structure Mapping

Understand how the codebase is organized:

1. Map the top-level directory layout and identify the purpose of each directory
2. Identify entry points (main files, route definitions, handler registrations)
3. Trace the dependency graph between major modules
4. Look for shared utilities, types, or constants that other modules depend on

## Phase 3: Convention Discovery

Learn the project's established patterns:

1. Read 2-3 representative files to understand naming conventions, error handling patterns, and code style
2. Check for linting rules or formatter configurations that enforce conventions
3. Identify recurring patterns (repository pattern, service layer, middleware chains, etc.)
4. Note any custom abstractions or domain-specific patterns unique to this project

## Phase 4: Synthesis

Compile findings into a concise summary:

- Technology stack and versions
- Directory structure with purpose annotations
- Key entry points and data flow
- Established conventions and patterns
- Areas of complexity or technical debt worth noting

## Guidelines

- Prioritize reading actual code over documentation — docs can be outdated
- Focus on understanding the current state, not the git history
- When the codebase is large, start from the entry point the user cares about and expand outward
- Surface surprising or non-obvious patterns — the user likely already knows the obvious ones
