# Agent Toolkit

Reference Musher bundle demonstrating the full breadth of asset types. Designed as the primary bundle for SDK pull, inspect, and resolve examples.

## Registry

- **Name:** `musher-examples/agent-toolkit`
- **Version:** 2.0.0
- **Replaces:** `acme/agent-toolkit:2.0.0` (SDK placeholder)

## Assets

| Logical Path | Type | Description |
|---|---|---|
| `prompts/system.md` | prompt | System prompt defining an AI coding assistant |
| `prompts/safety-guidelines.md` | prompt | Safety and content policy guidelines |
| `skills/researching-codebases/SKILL.md` | skill | Codebase exploration and mapping |
| `skills/explaining-architecture/SKILL.md` | skill | Architecture explanation generation |
| `toolsets/default-tools.json` | toolset | Default MCP tool permissions |
| `toolsets/restricted-tools.json` | toolset | Read-only locked-down toolset |
| `agent_specs/code-assistant.json` | agent_spec | Coding agent definition |
| `agent_specs/review-assistant.json` | agent_spec | Code review agent definition |
| `configs/model-settings.json` | config | Model selection and parameters |
| `rules/output-formatting.md` | rule | Output formatting conventions |
| `rules/code-style.md` | rule | Code style conventions |

## Used By SDK Examples

### Python SDK
- `examples/basics/pull_bundle.py` — pulls this bundle and enumerates files
- `examples/basics/inspect_manifest.py` — resolves manifest layers without downloading

### TypeScript SDK
- Referenced as an alternative bundle in basic pull/resolve examples
