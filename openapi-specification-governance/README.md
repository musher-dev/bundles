# OpenAPI Specification Governance

Keep your OpenAPI specs consistent, complete, and SDK-generation-ready. This bundle governs the specification document itself — metadata, operation identifiers, tag taxonomy, component reuse, multi-file structure, and change workflows — with automated audits and hands-on fixes.

## Quick Start

Install via the [Musher CLI](https://github.com/musher-dev/musher-cli):

```sh
musher bundle install musher-dev/openapi-specification-governance
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Audit our OpenAPI spec for compliance — check metadata, operationIds, tags, component reuse, and structure
```

## What's Inside

The bundle ships two agents and ten governance skills focused exclusively on the OpenAPI specification document.

The **spec_auditor** agent performs read-only analysis across nine compliance categories — spec metadata, operation identifiers, tag hierarchy, component reuse, advanced operations, extension governance, spec structure, authoring model, and change workflow — then produces a scored findings report with a compliance scorecard.

The **spec_designer** agent implements fixes for audit findings, scaffolds compliant specification files, and refactors non-conforming specs to meet governance standards.

The governance skills cover the full specification lifecycle: Info object completeness and LLM-readable descriptions, camelCase operationId naming with SDK generation impact, hierarchical tag taxonomy (OpenAPI 3.2.0), `$ref` patterns and schema composition with `allOf`/`oneOf`/discriminator, multi-file directory layout with Redocly bundling, design-first vs code-first authoring evaluation, and the Golden Path change workflow from design through deployment.

## Usage Examples

**Audit an existing spec**
```
Run a spec compliance audit on openapi.yaml — score metadata, operationIds, tags, components, and structure
```
The spec_auditor produces a 9-category compliance scorecard with severity-prioritized findings.

**Organize a multi-file spec**
```
Restructure our monolithic openapi.yaml into a multi-file layout with separate paths/, schemas/, and parameters/ directories
```
The spec_designer splits the spec following directory conventions and wires up `$ref` pointers.

**Evaluate your authoring model**
```
Should we use design-first or code-first for our API spec? Evaluate the trade-offs for our team
```
Assesses maintenance tax, authority flow, and source-of-truth governance to recommend the right approach.

## Boundaries

This bundle governs the **OpenAPI specification document** — the YAML/JSON files that define your API contract.

For **route and payload implementation** concerns (URI naming, HTTP semantics, error responses, Pydantic models), see [api-route-governance](../api-route-governance/).

For **contract enforcement pipeline** concerns (breaking change detection, conformance testing, SDK generation, CI gates), see [contract-enforcement-governance](../contract-enforcement-governance/).

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `governing-spec-metadata` | Info object completeness, servers array, organizational extensions, LLM-readable descriptions |
| `governing-operation-identifiers` | operationId naming (camelCase verbNoun), uniqueness, SDK generation impact |
| `governing-tag-hierarchy` | OpenAPI 3.2.0 hierarchical tags, classification types, migration from flat tags |
| `governing-component-reuse` | `$ref` patterns, inline schema anti-patterns, schema composition, RFC 9457 ProblemDetails |
| `governing-advanced-operations` | QUERY method, webhooks vs callbacks, streaming media types, link objects, Overlay spec |
| `governing-spec-extensions` | `x-` prefix conventions, portability assessment, extension registry governance |
| `governing-spec-structure` | Multi-file directory layout, file-level `$ref` strategies, Redocly CLI bundling |
| `governing-authoring-model` | Design-first vs code-first vs hybrid evaluation, source-of-truth governance |
| `governing-change-workflow` | Golden Path sequence, cross-team coordination, PR conventions, backward compatibility |
| `auditing-spec-compliance` | 9-category compliance audit with severity scoring and Spectral/Redocly/Vacuum recommendations |

### Agents

| Agent | Role |
|-------|------|
| `spec_auditor` | Read-only specification audits with compliance scorecard and improvement roadmap |
| `spec_designer` | Implements spec fixes, scaffolds compliant files, refactors non-conforming specifications |

</details>

---

**Domain:** specification · **Technologies:** OpenAPI (harness-agnostic) · **Aliases:** openapi-gov, oas-governance, openapi-spec
