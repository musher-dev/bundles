---
name: writing-readme
version: 1.0.0
user-invocable: false
description: Author compelling README.md files with structured sections for project overview, quick start, installation, usage examples, and contribution links. Adapts structure by project type (library, application, framework, CLI tool). Use when writing or improving a project README, creating first impressions, or optimizing repo discoverability. Triggered by: README, readme, project overview, repo description, quick start, getting started, project introduction.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# README Authoring

## Purpose

Author README.md files that serve as the front door to a repository. A good README answers three questions in under 30 seconds: what is this, who is it for, and how do I start. Every choice -- section order, heading depth, code example length -- should accelerate a new visitor toward that understanding.

## Instructions

Follow these steps when authoring or updating a project README:

1. **Read existing README or stub**
   - Look for `README.md`, `README`, or `readme.md` at the repository root
   - If no file exists, suggest running the `scaffolding-repo-documentation` skill first
   - Note any existing content the user wants to preserve

2. **Gather project context**
   - Inspect package manifests (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`)
   - Review source files to determine language, framework, purpose, and audience
   - Check for existing docs (`CONTRIBUTING.md`, `LICENSE`, `SECURITY.md`, `CHANGELOG.md`)
   - Identify project type: library, application, framework, or CLI tool

3. **Author sections following the framework**
   - Use the section-by-section guide in `./references/readme-framework.md`
   - Adapt section depth by project type:
     - **Library** emphasizes API surface, import examples, and type signatures
     - **Application** emphasizes deployment, environment variables, and architecture overview
     - **Framework** emphasizes concepts, getting started journey, and plugin/extension system
     - **CLI Tool** emphasizes command reference, flag documentation, and shell completion

4. **Apply quality criteria per section**

   | Section | Quality Test |
   |---|---|
   | Title | Matches package/repo name? Includes relevant badges (build, version, license)? |
   | Description | One paragraph? States what + who + why? Understandable without domain knowledge? |
   | Quick Start | Achievable in under 5 minutes? Copy-pasteable commands? Shows expected output? |
   | Installation | Covers all supported package managers? Version-pinned examples? |
   | Usage | Covers the 3 most common use cases? Runnable code examples? |
   | Configuration | Documents key options? Links to full reference if extensive? |
   | Contributing | Links to CONTRIBUTING.md? Welcoming tone? |

5. **Cross-reference other repo docs**
   - `CONTRIBUTING.md` → link from the Contributing section
   - `LICENSE` → reference in the License section, state the license type explicitly
   - `SECURITY.md` → ensure consistency with any security-related claims in the README
   - `CHANGELOG.md` → link when discussing versioning or recent changes

6. **Write the file**
   - Preserve any existing content the user wants to keep
   - Use ATX-style headings (`#`, `##`, `###`) consistently
   - Keep line lengths readable -- wrap prose at natural sentence boundaries

## Project-Type Adaptations

- **Library**: Emphasize API surface, import examples, type signatures, and compatibility matrix. Include a minimal "hello world" that demonstrates the primary export. Add a version compatibility table if the library supports multiple runtimes or language versions.
- **Application**: Emphasize deployment, environment variables, and architecture overview. Lead Quick Start with `docker run` or equivalent single-command launch. Include a system requirements section and link to operational runbooks.
- **Framework**: Emphasize concepts, getting started journey, and plugin/extension system. Walk through the mental model before showing code. Include a "Key Concepts" section that defines framework-specific terminology.
- **CLI Tool**: Emphasize command reference, flag documentation, and shell completion setup. Show the most common invocation first. Include a "Commands" section with a summary table linking to detailed usage.

## Guidelines

- **DO**: Lead with what the reader can accomplish, not what the project contains
- **DO**: Include copy-pasteable commands that work on a fresh machine
- **DO**: Show expected output for commands so readers can verify success
- **DO**: Use badges sparingly -- build status, latest version, and license are sufficient
- **DON'T**: Document every API method in the README -- link to API docs instead
- **DON'T**: Include a "Table of Contents" for READMEs under 200 lines
- **DON'T**: Use screenshots without alt text
- **DON'T**: Start the description with "This is a..." -- describe what it does, not what it is

## Edge Cases

- **Pre-release project**: Focus on vision, roadmap, and how to contribute. Quick Start may link to a development setup instead of installation. Mark the project status prominently with a banner or badge.
- **Mature project with breaking changes**: Include a migration guide section or link to CHANGELOG. Add a version compatibility matrix. Call out the latest stable version explicitly in the Installation section.
- **Monorepo root README**: Serve as a project map with a table of packages and links to their individual READMEs. Keep the root README navigational rather than instructional -- each package owns its own depth.
- **Internal/private project**: Skip badges, add an "Internal Use Only" notice, emphasize onboarding and team contacts. Include links to internal wikis or runbooks rather than public documentation.

## Supporting Files

- [README Section-by-Section Framework](./references/readme-framework.md)
