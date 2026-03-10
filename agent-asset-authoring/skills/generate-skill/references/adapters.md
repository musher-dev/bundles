# Adapter Configuration Reference

Vendor-specific configuration for portable-core skills. Each harness reads skill instructions differently -- the adapter layer bridges the gap.

---

## adapter.yaml Schema

```yaml
# Top-level keys are vendor identifiers
claude:
  # Claude Code specific fields
copilot:
  # GitHub Copilot specific fields
codex:
  # OpenAI Codex CLI specific fields
openhands:
  # OpenHands specific fields
gemini:
  # Gemini CLI specific fields
```

---

## Claude Code Adapter

Claude Code reads frontmatter directly from SKILL.md. For Claude Code projects, adapter fields are **merged into the SKILL.md frontmatter** rather than kept in a separate file.

### Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `version` | string | `1.0.0` | Semantic version |
| `user-invocable` | boolean | `false` | Enable `/skill-name` invocation |
| `allowed-tools` | string[] | All | Restrict to listed tools only |
| `context` | `fork` | shared | Run in isolated context window |
| `agent` | `Explore` \| `Plan` | none | Delegate to specialized agent |
| `disable-model-invocation` | boolean | `false` | Prevent auto-discovery (explicit `/invoke` only) |
| `argument-hint` | string | none | Hint shown in `/skill-name` completion |
| `hooks` | object | none | Pre/post tool execution scripts |

### Example: Merged SKILL.md Frontmatter

```yaml
---
name: auditing-security
version: 1.0.0
description: Audit code for security vulnerabilities...
user-invocable: true
allowed-tools:
  - Read
  - Grep
  - Glob
context: fork
---
```

### Example: adapter.yaml (for documentation/portability)

```yaml
claude:
  version: 1.0.0
  user-invocable: true
  allowed-tools:
    - Read
    - Grep
    - Glob
  context: fork
```

---

## GitHub Copilot Adapter

Copilot reads instructions from `.github/copilot-instructions.md` and prompt files.

### Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `user-invocable` | boolean | `true` | Whether user can invoke directly |
| `disable-model-invocation` | boolean | `false` | Prevent auto-discovery |

### Example

```yaml
copilot:
  user-invocable: true
  disable-model-invocation: false
```

### Notes
- Copilot uses `.github/copilot-instructions.md` for persistent instructions
- Prompt files in `.github/prompts/` serve as on-demand skills
- No tool restriction mechanism -- Copilot manages tool access internally

---

## OpenAI Codex CLI Adapter

Codex reads from `AGENTS.md` files and has its own tool/dependency model.

### Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `allow_implicit_invocation` | boolean | `true` | Whether Codex can auto-discover |
| `dependencies.tools` | string[] | `[]` | MCP tool server dependencies |

### Example

```yaml
codex:
  allow_implicit_invocation: true
  dependencies:
    tools:
      - filesystem
      - grep
```

### Notes
- Codex uses `AGENTS.md` for persistent instructions (equivalent to CLAUDE.md)
- Skills map to sections within AGENTS.md or referenced markdown files
- MCP tool servers provide tool access

---

## OpenHands Adapter

OpenHands uses keyword-based trigger matching.

### Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `triggers` | string[] | `[]` | Keywords that activate this skill |

### Example

```yaml
openhands:
  triggers:
    - security audit
    - vulnerability scan
    - code review security
```

### Notes
- OpenHands reads instructions from markdown files
- Trigger keywords drive skill selection
- No built-in tool restriction mechanism

---

## Gemini CLI Adapter

Gemini CLI reads from `GEMINI.md` and supports instruction files.

### Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `user-invocable` | boolean | `true` | Whether user can invoke directly |

### Example

```yaml
gemini:
  user-invocable: true
```

### Notes
- Gemini CLI uses `GEMINI.md` for persistent instructions
- Instruction files in project directories serve as skills
- Tool access managed by Gemini's sandbox model

---

## Multi-Harness Example

A complete adapter.yaml supporting multiple harnesses:

```yaml
# adapter.yaml for auditing-security skill

claude:
  version: 1.0.0
  user-invocable: true
  allowed-tools:
    - Read
    - Grep
    - Glob
  context: fork

copilot:
  user-invocable: true

codex:
  allow_implicit_invocation: true
  dependencies:
    tools:
      - filesystem
      - grep

openhands:
  triggers:
    - security audit
    - vulnerability check
    - OWASP review

gemini:
  user-invocable: true
```

---

## Migration Guide: Commands to Skills + Adapter

If you have existing Claude Code command files (`.claude/commands/*.md`), here's how to convert them:

### Step 1: Identify Portable Content

Separate the command into:
- **Portable instructions** -- the workflow steps, guidelines, examples
- **Vendor-specific features** -- `$ARGUMENTS`, `$1`, `!cmd`, `@file`, `disable-model-invocation`

### Step 2: Create Portable Core

Move the instructions into a SKILL.md with only `name` + `description` in frontmatter. Replace variable references with natural language:

| Old (Command) | New (Portable Core) |
|---------------|---------------------|
| `$ARGUMENTS` | "The user's input describing..." |
| `$1` | "The first argument provided by the user..." |
| `` !`git status` `` | "Check the current git status" |
| `@src/file.ts` | "Read the file at the path provided by the user" |

### Step 3: Create Adapter Config

Move vendor-specific fields to adapter.yaml:

```yaml
claude:
  user-invocable: true
  argument-hint: <migration-name> [--dry-run]
  allowed-tools:
    - Read
    - Grep
    - Glob
    - Bash
  disable-model-invocation: true  # If it was a command (explicit-only)
```

### Step 4: Generate Claude Code SKILL.md

For Claude Code, merge the adapter fields back into the SKILL.md frontmatter (Claude Code reads frontmatter directly). The adapter.yaml serves as documentation for portability.
