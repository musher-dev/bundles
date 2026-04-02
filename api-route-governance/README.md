# API Route Governance

Stop debating REST conventions. This bundle encodes API design standards so every endpoint follows the same naming, versioning, error, and payload rules — enforced by automated audits and ready-made fixes.

## Quick Start

Install via the [Musher CLI](https://github.com/musher-dev/musher-cli):

```sh
musher bundle install musher-dev/api-route-governance
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Audit the API routes in src/api/ for compliance with our REST conventions
```

## What's Inside

The bundle ships two agents and thirteen governance skills that cover the full API design lifecycle — from URI structure through field serialization to schema implementation.

The **route_auditor** agent runs read-only compliance checks against your route designs and payload contracts. It scores endpoints across URI naming, HTTP semantics, error responses, versioning, pagination, tenant routes, payload structure, field formats, and mutation patterns, then produces a severity-prioritized findings report.

The **route_designer** agent picks up where the auditor leaves off. It implements fixes for governance violations, scaffolds new compliant endpoints, and refactors non-conforming routes to match your standards.

Each governance skill encodes a specific domain of API design — route naming conventions, HTTP method semantics per RFC 9110, RFC 9457 Problem Details for errors, cursor-based pagination envelopes, field serialization formats (RFC 3339 timestamps, ISO 8601 durations, snake_case naming), null-vs-absent lifecycle semantics, JSON Merge Patch mutation rules, and Pydantic V2 schema implementation patterns.

## Usage Examples

**Run a full compliance audit**
```
Run a route compliance audit on our API — check naming, HTTP semantics, error responses, pagination, and payload structure
```
The route_auditor scans your endpoints and produces a scored findings report with severity levels and remediation guidance.

**Design new endpoints**
```
Design the CRUD routes for a new "projects" resource following our API governance standards
```
The route_designer scaffolds compliant routes with proper naming, status codes, error responses, and pagination.

**Review field serialization**
```
Review the field formats in our API responses — timestamps, money fields, enums, identifiers
```
Checks that your fields follow serialization conventions: RFC 3339 timestamps with `_time` suffix, embedded money objects, UPPER_SNAKE_CASE enums, and opaque string identifiers.

## Boundaries

This bundle governs the **route and payload layer** — URI design, HTTP semantics, request/response shapes, and framework-level schema implementation (e.g., Pydantic models).

For **OpenAPI specification document** standards (metadata, operationIds, tags, component reuse, multi-file structure), see [openapi-specification-governance](../openapi-specification-governance/).

For **contract enforcement pipeline** concerns (breaking change detection, conformance testing, SDK generation, CI gates), see [contract-enforcement-governance](../contract-enforcement-governance/).

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `governing-route-naming` | URI design — resource nouns, pluralization, nesting depth, kebab-case paths, UUIDs, `/me` alias |
| `governing-http-semantics` | HTTP method safety, idempotency, status code selection per RFC 9110 |
| `governing-error-responses` | RFC 9457 Problem Details compliance, field-level validation errors |
| `governing-api-versioning` | Expand-and-Contract migration, deprecation headers, breaking change classification |
| `governing-openapi-contracts` | OpenAPI metadata, operation IDs, response models, FastAPI configuration |
| `governing-collection-endpoints` | Cursor-based pagination, page size limits, filter/sort parameters, response envelopes |
| `governing-payload-structure` | Top-level JSON object rules, collection wrapping, I-JSON compliance per RFC 7493 |
| `governing-field-serialization` | RFC 3339 timestamps, ISO 8601 durations, money objects, enum formats, snake_case naming |
| `governing-field-lifecycle` | Null-vs-absent semantics, OUTPUT_ONLY/IMMUTABLE annotations, input-output model separation |
| `governing-mutation-semantics` | PUT full replacement, JSON Merge Patch (RFC 7396), array mutation via sub-resources |
| `governing-schema-implementation` | Pydantic V2 config, response_model enforcement, discriminated unions, model naming |
| `governing-tenant-routes` | Multi-tenant path design, dual-validation middleware, BOLA prevention |
| `auditing-route-compliance` | 60-question compliance audit across 5 categories with severity scoring |

### Agents

| Agent | Role |
|-------|------|
| `route_auditor` | Read-only compliance audits producing severity-prioritized findings reports |
| `route_designer` | Implements fixes, scaffolds compliant endpoints, refactors non-conforming routes |

</details>

---

**Domain:** api-route · **Technologies:** Harness-agnostic · **Aliases:** api-standards, endpoint-conventions, api-design-rules
