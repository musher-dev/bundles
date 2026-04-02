# Demo Starter

Demonstration bundle showcasing the Musher platform with practical skills and agents for everyday development workflows. Used as the primary bundle for the Python and TypeScript SDK examples in the [demo repo](https://github.com/musher-dev/demo).

## Registry

- **Name:** `musher-dev/demo-starter`
- **Version:** 1.0.0
- **Repository:** https://github.com/musher-dev/demo

## Assets

| Logical Path | Type | Description |
|---|---|---|
| `skills/summarizing-changes/SKILL.md` | skill | Summarize git history, staged diffs, or PR changes into clear summaries grouped by theme |
| `skills/writing-commit-messages/SKILL.md` | skill | Generate Conventional Commits-compliant commit messages from staged changes |
| `agent_specs/code-reviewer.json` | agent_spec | Review code for quality, correctness, security, and best-practice adherence |

## Used By SDK Examples

### Python SDK
- `examples/` — `musher.pull("musher-dev/demo-starter:1.0.0")`

### TypeScript SDK
- `examples/` — `musher.pull("musher-dev/demo-starter:1.0.0")`
