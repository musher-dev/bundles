# Bundles

Portable agent bundles for the [Musher](https://musher.dev) platform. Used in platform development, available as reference examples, and published to the [Hub](https://hub.musher.dev).

## Bundles

| Bundle | Domain | Purpose |
|--------|--------|---------|
| `agent-asset-authoring` | authoring | Templates for creating skills, hooks, rules, and other agent assets |
| `api-route-governance` | governance | API endpoint design standards — naming, HTTP methods, versioning, errors |
| `contract-enforcement-governance` | governance | Contract verification across the pipeline — breaking changes, conformance, CI gates |
| `database-schema-governance` | governance | Database design standards — naming, data types, indexing, structural patterns |
| `developer-docs-authoring` | authoring | Generate, maintain, and audit developer documentation |
| `developer-environment-authoring` | authoring | Devcontainer configs, task runners, and workspace setup |
| `marketing-site-authoring` | authoring | Multi-agent pipeline for marketing site content production |
| `musher-bundle-scaffolding` | scaffolding | Scaffold new bundle directories with naming validation |
| `openapi-specification-governance` | governance | OpenAPI specification standards and contract quality |
| `product-profile-authoring` | authoring | Canonical product metadata, positioning, and roadmaps |
| `project-shaping-orchestration` | orchestration | Problem framing, solution design, risk assessment, and scoping |
| `repo-documentation-authoring` | authoring | Repository documentation files for discoverability and community health |

## SDK Examples

The [`sdk-examples/`](sdk-examples/) directory contains Musher registry bundles (with `musher.yaml`) that demonstrate SDK integration across harnesses. These follow a different structure from the bundles above. See the [SDK examples README](sdk-examples/README.md).

## Bundle Structure

Each bundle contains a `README.md` and may include `skills/`, `agents/`, `references/`, and `rules/` directories. Bundles use a portable-core + adapter architecture so they work across harnesses. See [CLAUDE.md](CLAUDE.md) for naming conventions and component rules.

## Usage

**From the Hub** — install bundles with the [Mush CLI](https://github.com/musher-dev/mush):

```sh
mush bundle install namespace/slug
```

**Local development** — clone the repo and load bundles into your harness workspace.

## Ecosystem

| Resource | Description |
|----------|-------------|
| [Hub](https://hub.musher.dev) | Browse and discover published bundles |
| [Docs](https://docs.musher.dev) | Platform documentation |
| [Mush CLI](https://github.com/musher-dev/mush) | Install and run bundles locally |
| [Musher CLI](https://github.com/musher-dev/musher-cli) | Create, validate, and publish bundles |
| [TypeScript SDK](https://github.com/musher-dev/typescript-sdk) | Programmatic bundle access for Node.js |
| [Python SDK](https://github.com/musher-dev/python-sdk) | Programmatic bundle access for Python |
| [Specs](https://github.com/musher-dev/specs) | Canonical JSON schemas for the bundle format |
