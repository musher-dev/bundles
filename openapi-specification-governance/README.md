# OpenAPI Specification Governance

Audits, authors, and enforces OpenAPI specifications for standards compliance and contract quality.

## Metadata

- **Domain:** specification
- **Risk Level:** medium
- **Technologies:** OpenAPI
- **Aliases:** openapi-gov, oas-governance, openapi-spec

## Boundary

This bundle governs the OpenAPI specification document itself, framework-agnostic. For framework-specific code configuration (e.g., FastAPI route decorators, Pydantic models, `root_path`), see `api-route-governance`.

## Components

### Skills

| Name | Scope |
|------|-------|
| `governing-spec-metadata` | Info object completeness, `openapi` vs `info.version` versioning, servers array, organizational extensions, LLM-readable descriptions, `externalDocs` |
| `governing-operation-identifiers` | `operationId` naming (camelCase verbNoun), uniqueness, component key regex, schema naming (UpperCamelCase), parameter naming, SDK generation impact |
| `governing-tag-hierarchy` | 3.2.0 hierarchical tags, tag classification (navigation/badge/lifecycle/audience), migration from flat tags and `x-tagGroups` |
| `governing-component-reuse` | `$ref` patterns, inline schema anti-patterns, schema composition (`allOf`/`oneOf`/`discriminator`), RFC 9457 ProblemDetails, SDK generation impact |
| `governing-advanced-operations` | QUERY method via `additionalOperations`, webhooks vs callbacks, streaming media types, `itemSchema`, link objects, Overlay specification |
| `governing-spec-extensions` | `x-` prefix rules, extension governance, portability impact, standards-over-extensions philosophy, deprecating extensions |
| `auditing-spec-compliance` | 6-category question bank, severity model (Critical/Major/Minor/Cosmetic), compliance scorecard, Spectral/Redocly/Vacuum rulesets, CI integration |

### Agents

| Name | Purpose | Model | Tools |
|------|---------|-------|-------|
| `spec_auditor` | Read-only specification audits with compliance scoring | opus | Read, Grep, Glob |
| `spec_designer` | Specification authoring, fixing violations, scaffolding | sonnet | Read, Grep, Glob, Edit, Write |

### Tools
- None
