# Bundles

Portable agent bundles for the [Musher](https://musher.dev) platform. Used in platform development, available as reference examples, and published to the [Hub](https://hub.musher.dev).

## Bundles

| Bundle | Purpose |
|--------|---------|
| [`agent-asset-authoring`](agent-asset-authoring/) | Generate well-structured agent and skill files with portable-core architecture |
| [`api-route-governance`](api-route-governance/) | Encode API design standards — naming, versioning, errors, and payload rules |
| [`contract-enforcement-governance`](contract-enforcement-governance/) | Catch breaking changes, enforce conformance tests, and gate CI |
| [`database-schema-governance`](database-schema-governance/) | Enforce PostgreSQL schema standards from naming through migration safety |
| [`developer-docs-authoring`](developer-docs-authoring/) | Design, scaffold, and audit developer documentation end-to-end |
| [`developer-environment-authoring`](developer-environment-authoring/) | Audit, scaffold, and maintain dev environment configs |
| [`marketing-site-authoring`](marketing-site-authoring/) | Multi-stage content pipeline from brand strategy through design audit |
| [`musher-bundle-scaffolding`](musher-bundle-scaffolding/) | Scaffold new bundle directories with validated naming |
| [`openapi-specification-governance`](openapi-specification-governance/) | Keep OpenAPI specs consistent, complete, and SDK-generation-ready |
| [`product-profile-authoring`](product-profile-authoring/) | Create canonical product profiles — metadata, positioning, roadmaps |
| [`project-shaping-orchestration`](project-shaping-orchestration/) | Shape projects before committing to build — problem framing through scoping |
| [`repo-documentation-authoring`](repo-documentation-authoring/) | Scaffold and maintain GitHub community health files |

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
