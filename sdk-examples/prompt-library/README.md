# Prompt Library

Musher bundle with a curated collection of prompts, toolsets, and agent specs. Designed as the primary bundle for SDK typed-access and PydanticAI integration examples.

## Registry

- **Name:** `musher-examples/prompt-library`
- **Version:** 1.2.0
- **Replaces:** `acme/prompt-library:1.2.0` (SDK placeholder)

## Assets

| Logical Path | Type | Description |
|---|---|---|
| `prompts/system.md` | prompt | Software engineering assistant system prompt |
| `prompts/coding-guidelines.md` | prompt | Language-agnostic coding standards |
| `prompts/documentation-standards.md` | prompt | Technical documentation standards |
| `toolsets/code-analysis-tools.json` | toolset | Static analysis and linting tool configs |
| `toolsets/documentation-tools.json` | toolset | Documentation generation tool configs |
| `agent_specs/coding-assistant.json` | agent_spec | Coding-focused assistant definition |
| `agent_specs/documentation-agent.json` | agent_spec | Documentation-focused agent definition |

## Used By SDK Examples

### Python SDK
- `examples/basics/bundle_resources.py` — accesses prompts, toolsets, and agent specs by type
- `examples/pydantic_ai/instructions_from_bundle.py` — loads `prompts/system.md` as PydanticAI system instructions
- `examples/pydantic_ai/dynamic_instructions.py` — uses toolsets for runtime-loaded agent configuration
