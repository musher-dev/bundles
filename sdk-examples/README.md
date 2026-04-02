# SDK Example Bundles

Musher registry bundles used as working reference material for the [Python SDK](https://github.com/musher-dev/python-sdk) and [TypeScript SDK](https://github.com/musher-dev/typescript-sdk) examples.

These are **Musher registry bundles** (structured with `musher.yaml` and asset files), not Claude Code bundles. They follow a different structure from the governance and authoring bundles at the repository root.

## Bundles

| Directory | Registry Name | Version | Purpose | SDK Coverage |
|-----------|--------------|---------|---------|--------------|
| `agent-toolkit/` | `musher-examples/agent-toolkit` | 2.0.0 | Showcase all asset types for enumeration and inspection | Python: `pull_bundle`, `inspect_manifest` |
| `code-review-kit/` | `musher-examples/code-review-kit` | 1.2.0 | Installable skills for Claude, OpenAI, and VS Code integration | TypeScript: `pull-bundle`, `resolve-bundle`, `install-project-skills`, `install-vscode-skills`; Python: `install_project_skills`, `export_plugin`, OpenAI examples |
| `data-workflows/` | `musher-examples/data-workflows` | 1.0.0 | CSV data analysis and summarization in hosted containers | Python: OpenAI `hosted_inline_skill` |
| `prompt-library/` | `musher-examples/prompt-library` | 1.2.0 | Typed resource access (prompts, toolsets, agent specs) | Python: `bundle_resources`, PydanticAI `instructions_from_bundle`, `dynamic_instructions` |

## Structure

Each bundle contains:

- `musher.yaml` — bundle metadata and asset manifest
- `README.md` — description, asset inventory, and SDK cross-references
- Asset files organized by type (`prompts/`, `skills/`, `toolsets/`, `agent_specs/`, `configs/`, `rules/`)

## Publishing

These bundles are published to the Musher registry under the `musher-examples` namespace. Version numbers match what the SDK example code expects.
