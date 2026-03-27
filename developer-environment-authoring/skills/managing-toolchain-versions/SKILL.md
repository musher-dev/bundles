---
name: managing-toolchain-versions
version: 1.0.0
user-invocable: false
description: >-
  Configure and enforce toolchain version pinning using .tool-versions (asdf/mise),
  .nvmrc, .python-version, .ruby-version, .node-version, and runtime-specific
  version files. Covers version manager selection, pinning granularity, conflict
  resolution across multiple version files, CI alignment, and monorepo versioning
  strategies. Use when standardizing runtime versions across a team, resolving
  version conflicts, or auditing version file consistency.
  Triggered by: tool-versions, asdf, mise, nvm, nvmrc, node-version,
  python-version, ruby-version, version pinning, runtime version, fnm, pyenv,
  rbenv, version manager.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Toolchain Version Management

Canonical rules for declaring, pinning, and synchronizing runtime and tool versions across local development, CI, and container environments.

## Purpose

Version drift is a silent productivity killer. Developer A uses Node 20, Developer B uses Node 22, and a dependency silently changes behavior between versions. CI uses a third version, and the bug surfaces only in production. This skill eliminates version drift by establishing a single source of truth for every runtime and tool version in the project.

Use this skill when adding version files to a project, auditing existing version specifications for consistency, choosing between version managers, or synchronizing versions across devcontainers and CI.

---

## Version Manager Selection

### Decision Matrix

| Manager | Scope | Languages/Tools | Config File | Best For |
|---|---|---|---|---|
| **mise** | Polyglot | 400+ plugins (node, python, go, terraform, etc.) | `.tool-versions` or `mise.toml` | Teams wanting one tool for all runtimes |
| **asdf** | Polyglot | 400+ plugins (same ecosystem as mise) | `.tool-versions` | Established teams already on asdf |
| **nvm** | Node only | Node.js | `.nvmrc` | Node-only projects, wide adoption |
| **fnm** | Node only | Node.js | `.nvmrc` or `.node-version` | Fast alternative to nvm |
| **pyenv** | Python only | Python | `.python-version` | Python-focused projects |
| **rbenv** | Ruby only | Ruby | `.ruby-version` | Ruby-focused projects |

### Recommendation

- **Polyglot projects**: Use `mise` with `.tool-versions` — one file, one tool, all runtimes
- **Single-language projects**: Use the language-specific manager if the team already has it, but prefer `mise` for new projects
- **Devcontainer-first projects**: Pin versions in devcontainer features and use `.tool-versions` as documentation for non-container use

---

## Version File Formats

### `.tool-versions` (asdf / mise)

```
nodejs 22.11.0
python 3.12.7
golang 1.22.5
terraform 1.9.5
```

**Rules:**
- One tool per line: `<plugin> <version>`
- Exact versions only — no ranges, no `lts`, no `latest`
- Alphabetical ordering for easy scanning
- Comments with `#` for context when pinning a non-latest version

### `.nvmrc`

```
22.11.0
```

**Rules:**
- Single line, version number only
- No `v` prefix (nvm accepts it but `.nvmrc` convention omits it)
- Exact version — never `lts/*` or `node`

### `.python-version`

```
3.12.7
```

**Rules:**
- Single line, exact version
- Must match a version installable by pyenv

### `.node-version`

```
22.11.0
```

**Rules:**
- Single line, exact version
- Read by fnm, nodenv, and some CI systems
- Functionally equivalent to `.nvmrc` for version declaration

---

## Pinning Granularity

| Granularity | Example | When to Use |
|---|---|---|
| Exact (major.minor.patch) | `22.11.0` | Default for all projects — maximum reproducibility |
| Minor (major.minor) | `22.11` | Only when a tool's plugin doesn't support patch-level resolution |
| Major only | `22` | Never in version files — only acceptable in devcontainer features |

**Rule:** Always pin to exact `major.minor.patch` in version files. The 30 seconds to look up the latest patch version prevents hours of debugging version-specific behavior.

---

## Conflict Resolution

When multiple version files exist for the same runtime, resolution order varies by tool:

### Node.js Version Precedence

| Priority | File | Read By |
|---|---|---|
| 1 | `mise.toml` / `.tool-versions` | mise |
| 2 | `.nvmrc` | nvm, fnm |
| 3 | `.node-version` | fnm, nodenv, Vercel, Netlify |
| 4 | `package.json` `engines.node` | npm (advisory only) |

### Rules for Avoiding Conflicts

1. **Choose one primary version file** — `.tool-versions` for polyglot, `.nvmrc` for Node-only
2. **If multiple files must exist** (e.g., `.tool-versions` for local + `.nvmrc` for CI), they must specify the same version
3. **Document the source of truth** in `README.md` or `CONTRIBUTING.md`
4. **Add a CI check** that verifies version file consistency

### Consistency Check Script

```bash
#!/usr/bin/env bash
# Verify Node version consistency across version files
set -euo pipefail

TOOL_VERSIONS_NODE=$(grep '^nodejs' .tool-versions | awk '{print $2}')
NVMRC_NODE=$(cat .nvmrc 2>/dev/null || echo "")
PKG_ENGINE=$(node -e "console.log(require('./package.json').engines?.node || '')" 2>/dev/null || echo "")

if [[ -n "$NVMRC_NODE" && "$TOOL_VERSIONS_NODE" != "$NVMRC_NODE" ]]; then
  echo "MISMATCH: .tool-versions ($TOOL_VERSIONS_NODE) != .nvmrc ($NVMRC_NODE)"
  exit 1
fi

echo "Version files consistent: Node $TOOL_VERSIONS_NODE"
```

---

## CI Alignment

The CI environment must use the same version source as local development.

### GitHub Actions

```yaml
# Read from .tool-versions or .nvmrc
- uses: actions/setup-node@v4
  with:
    node-version-file: '.nvmrc'

# Or for mise-managed polyglot
- uses: jdx/mise-action@v2
```

### Docker / Devcontainer

```dockerfile
# Read version from .tool-versions at build time
ARG NODE_VERSION
RUN mise install nodejs@${NODE_VERSION}
```

### Alignment Rules

| Source of Truth | CI Reads From | Devcontainer Feature Version |
|---|---|---|
| `.tool-versions` | `.tool-versions` via mise-action | Must match `.tool-versions` entry |
| `.nvmrc` | `.nvmrc` via setup-node | Must match `.nvmrc` entry |
| `.python-version` | `.python-version` via setup-python | Must match `.python-version` entry |

**Rule:** Never hardcode a version in CI YAML that is also declared in a version file. Always read from the file.

---

## Monorepo Versioning

### Root-Level Versions

Place `.tool-versions` at the repository root for shared runtimes:

```
# .tool-versions (root)
nodejs 22.11.0
python 3.12.7
```

### Package-Level Overrides

If a specific package requires a different version, place a `.tool-versions` in that package directory:

```
# packages/legacy-api/.tool-versions
nodejs 18.20.4
```

**Rules:**
- Root `.tool-versions` is the default for all packages
- Package-level overrides must be documented with a comment explaining why
- Minimize overrides — version divergence within a monorepo increases maintenance burden
- CI matrix builds should respect package-level version files

---

## Anti-Patterns

### 1. Using `latest` or `lts` in Version Files

```
# BAD
nodejs lts
python latest
```

These resolve to different versions over time, defeating the purpose of version pinning.

### 2. Version in CI but Not in Version File

```yaml
# BAD: Hardcoded in CI, not in .tool-versions
- uses: actions/setup-node@v4
  with:
    node-version: '22.11.0'
```

Creates a second source of truth that drifts from local development.

### 3. Multiple Conflicting Version Files

Having `.tool-versions` with `nodejs 22.11.0` and `.nvmrc` with `20.10.0`. Different team members get different versions depending on which tool they use.

### 4. Missing `engines` Field in package.json

```json
{
  "name": "my-app"
}
```

Without `engines.node`, npm won't warn developers using the wrong Node version. Add it as a safety net (advisory, not authoritative):

```json
{
  "engines": {
    "node": ">=22.11.0"
  }
}
```

### 5. Pinning to Major Only

```
nodejs 22
```

Node 22.0.0 and 22.11.0 can have different behavior. Patch-level pinning prevents subtle inconsistencies.

---

## Examples

### Example 1: New Python + Node Project

**.tool-versions:**
```
nodejs 22.11.0
python 3.12.7
```

**package.json (advisory):**
```json
{
  "engines": { "node": ">=22.11.0" }
}
```

**pyproject.toml (advisory):**
```toml
[project]
requires-python = ">=3.12"
```

**CI:**
```yaml
- uses: jdx/mise-action@v2
# mise reads .tool-versions automatically
```

### Example 2: Node-Only Project with nvm

**.nvmrc:**
```
22.11.0
```

**package.json:**
```json
{
  "engines": { "node": ">=22.11.0" }
}
```

**CI:**
```yaml
- uses: actions/setup-node@v4
  with:
    node-version-file: '.nvmrc'
```

---

## Checklist

Before committing version configuration:

- [ ] One primary version file chosen (`.tool-versions` or runtime-specific)
- [ ] All versions pinned to exact `major.minor.patch`
- [ ] No duplicate version declarations with conflicting values
- [ ] CI reads versions from the version file (not hardcoded)
- [ ] Devcontainer feature versions match version file entries
- [ ] `engines` field set in `package.json` (if Node project)
- [ ] `requires-python` set in `pyproject.toml` (if Python project)
- [ ] Version source of truth documented in README or CONTRIBUTING

---

## Related Skills and Agents

- **`configuring-devcontainers` skill**: Devcontainer features also pin runtime versions — this skill ensures those stay synchronized with `.tool-versions`
- **`aligning-ci-environments` skill**: CI must read from the same version source — this skill provides the alignment patterns
- **`auditing-environment-completeness` skill**: Checks whether version files exist and are consistent
- **`environment_scaffolder` agent**: Generates version files as part of complete environment setup
- **`environment_auditor` agent**: Detects version file conflicts and missing pinning
