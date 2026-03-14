---
name: governing-spec-metadata
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of OpenAPI specification metadata including Info object completeness, openapi version field vs info.version distinction, servers array configuration, organizational extensions (x-api-id, x-audience), LLM-readable descriptions, and externalDocs linkage. Use when reviewing OpenAPI spec metadata, auditing Info object completeness, configuring servers arrays, adding organizational extensions, or improving spec discoverability. Triggered by: spec metadata, Info object, openapi version, info.version, servers array, x-api-id, x-audience, externalDocs, API metadata, spec completeness, LLM-readable, API description, spec info, API identity.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Spec Metadata Governance

Comprehensive governance for the OpenAPI specification's metadata layer -- the Info object, version fields, servers array, organizational extensions, and external documentation linkage.

## Purpose

The metadata layer of an OpenAPI spec is the first thing humans, tools, and LLMs encounter. Incomplete metadata causes SDK generators to produce unnamed packages, documentation portals to display blank headers, and API catalogs to reject registration. This skill enforces completeness and precision in the metadata layer, treating it as the API's identity card.

**Boundary:** This skill governs the OpenAPI specification document itself, framework-agnostic. For FastAPI-specific code configuration (root_path, tags_metadata), see `api-route-governance/governing-openapi-contracts`.

Use this skill when authoring or reviewing the `info`, `openapi`, `servers`, and `externalDocs` sections of an OpenAPI document.

---

## Version Fields: `openapi` vs `info.version`

### Two Distinct Version Fields

OpenAPI documents contain two version fields that serve entirely different purposes:

| Field | Purpose | Example | Who Consumes It |
|-------|---------|---------|-----------------|
| `openapi` | Specification version -- which OAS grammar the document follows | `"3.1.0"` | Parsers, validators, tooling |
| `info.version` | API version -- the release version of the described API | `"2.4.1"` | Consumers, SDK generators, changelogs |

### `openapi` Field Rules

```yaml
openapi: "3.1.0"
```

- Target `3.1.0` as the minimum for all new specs
- Never omit -- parsers cannot validate without it
- Treat as infrastructure, not a design choice -- upgrade when tooling supports it

**3.1.0 over 3.0.x advantages:**
- Full JSON Schema 2020-12 alignment (`type: ["string", "null"]` replaces `nullable: true`)
- Webhooks as first-class objects
- `$ref` alongside sibling properties (no `allOf` workarounds)
- `pathItems` for reusable path definitions

**3.2.0 awareness:**
- Introduces QUERY method, hierarchical tags, Overlay specification
- Do not use 3.2.0 features until tooling adoption reaches critical mass

### `info.version` Field Rules

```yaml
info:
  version: "2.4.1"
```

- Follow semantic versioning (semver): `MAJOR.MINOR.PATCH`
- Increment MAJOR for breaking changes, MINOR for additive changes, PATCH for fixes
- This version appears in generated SDKs, documentation headers, and API catalogs
- Keep synchronized with the actual API release -- stale versions erode consumer trust

---

## Info Object Completeness

### Required Fields

Every OpenAPI spec must populate these Info object fields:

```yaml
info:
  title: "Acme Platform API"
  summary: "Multi-tenant REST API for project and organization management"
  description: |
    The Acme Platform API provides programmatic access to projects,
    organizations, members, and billing. All endpoints require
    authentication via Bearer token or API key.

    ## Authentication
    Include `Authorization: Bearer <token>` on every request.

    ## Rate Limiting
    All endpoints enforce 1000 requests/minute per API key.
    Exceeded limits return 429 with `Retry-After` header.
  version: "2.4.1"
  contact:
    name: "API Support"
    url: "https://developer.acme.com/support"
    email: "api-support@acme.com"
  license:
    name: "Proprietary"
    identifier: "LicenseRef-Acme-Proprietary"
```

### Completeness Checklist

| Field | Required | Purpose | Impact if Missing |
|-------|----------|---------|-------------------|
| `title` | Yes | API name in docs, SDK package names | SDK generates unnamed package |
| `summary` | Yes | One-line description for catalogs | Catalog entry shows blank |
| `description` | Yes | Detailed overview with Markdown | Portal landing page is empty |
| `version` | Yes | API version for consumers | SDK versioning breaks |
| `contact` | Recommended | Support channel for consumers | No escalation path |
| `license` | Recommended | Legal terms for API usage | Legal ambiguity |
| `termsOfService` | Optional | URL to terms of service | Acceptable to omit for internal APIs |

### Description Quality Standards

The `description` field is the API's landing page. It must:

1. **Open with a one-sentence summary** of what the API does
2. **List authentication requirements** -- how consumers authenticate
3. **State rate limiting policy** -- limits, headers, behavior on exceed
4. **Use Markdown formatting** -- headers, lists, code blocks for readability
5. **Be LLM-readable** -- write for both humans and AI assistants consuming the spec

**LLM-readable descriptions:**
- Use complete sentences, not telegraphic fragments
- Define domain-specific terms on first use
- State constraints explicitly ("all timestamps are UTC, RFC 3339 format")
- Avoid ambiguous pronouns -- repeat the noun

---

## Servers Array

### Configuration

The `servers` array tells consumers where to send requests:

```yaml
servers:
  - url: "https://api.acme.com/v2"
    description: "Production"
  - url: "https://staging-api.acme.com/v2"
    description: "Staging"
  - url: "https://sandbox-api.acme.com/v2"
    description: "Sandbox (test data, rate limits relaxed)"
```

### Servers Rules

| Rule | Rationale |
|------|-----------|
| Include at least one server | Tooling defaults to localhost without it |
| Production server listed first | Most tools use the first entry as default |
| Include description for every server | Consumers must know which environment they're targeting |
| Include version prefix in URL if URI-versioned | Ensures generated clients hit the correct version |
| Use variables for parameterized URLs | Multi-region or tenant-specific base URLs |

### Server Variables

For multi-region or parameterized deployments:

```yaml
servers:
  - url: "https://{region}.api.acme.com/v2"
    description: "Regional endpoint"
    variables:
      region:
        default: "us-east-1"
        enum:
          - "us-east-1"
          - "eu-west-1"
          - "ap-southeast-1"
        description: "AWS region for the API endpoint"
```

---

## Organizational Extensions

### `x-api-id`

A stable, unique identifier for the API across the organization's catalog:

```yaml
info:
  x-api-id: "acme-platform-api"
```

- Use kebab-case, globally unique within the organization
- Never reuse -- even if the API is rebuilt from scratch
- Used by API catalogs, governance dashboards, and dependency graphs

### `x-audience`

Declares the intended consumer audience:

```yaml
info:
  x-audience: "external-public"
```

| Value | Meaning |
|-------|---------|
| `external-public` | Available to any registered developer |
| `external-partner` | Available to contracted partners only |
| `internal` | Available within the organization |
| `team-internal` | Available within the owning team only |

**Governance impact:** Audience level determines documentation requirements, breaking change policies, and deprecation timelines. External-public APIs require stricter governance than team-internal APIs.

### Other Common Organizational Extensions

| Extension | Purpose | Example |
|-----------|---------|---------|
| `x-api-owner` | Team or individual responsible | `"platform-team"` |
| `x-api-lifecycle` | Current lifecycle stage | `"production"`, `"deprecated"`, `"experimental"` |
| `x-api-tier` | SLA tier | `"tier-1"`, `"tier-2"` |

**Rule:** Prefix all organizational extensions with `x-`. Document them in a central extension registry. See `governing-spec-extensions` for extension governance.

---

## External Documentation

### `externalDocs` Configuration

Link the spec to its human-readable documentation:

```yaml
externalDocs:
  description: "Full API documentation with guides and tutorials"
  url: "https://developer.acme.com/docs/platform-api"
```

### Rules

- Every public API spec must include `externalDocs`
- The URL must resolve to a live page (not a placeholder)
- The description should indicate what the external docs contain beyond the spec
- Operation-level `externalDocs` can link to specific guides:

```yaml
paths:
  /projects:
    post:
      externalDocs:
        description: "Project creation guide with examples"
        url: "https://developer.acme.com/guides/create-project"
```

---

## Review Dimensions

When reviewing or auditing spec metadata, evaluate these dimensions:

### 1. Version Accuracy
- Is `openapi` set to `3.1.0` (not defaulting to 3.0.x)?
- Does `info.version` match the actual deployed API version?
- Are the two version fields correctly distinguished?

### 2. Info Object Completeness
- Are all required fields populated (title, summary, description, version)?
- Does the description include authentication, rate limiting, and key constraints?
- Is the description LLM-readable (complete sentences, defined terms)?

### 3. Servers Configuration
- Is at least one server listed?
- Is production the first entry?
- Do all servers have descriptions?
- Are variables used for parameterized URLs?

### 4. Organizational Identity
- Is `x-api-id` set with a stable, unique identifier?
- Is `x-audience` declared with a valid value?
- Are organizational extensions documented in a central registry?

### 5. External Documentation
- Does `externalDocs` link to a live documentation page?
- Do complex operations link to specific guides?

---

## Anti-Patterns

### 1. Missing `openapi` Field
Parsers cannot validate the document. Some tools silently default to 3.0.0, causing subtle schema incompatibilities.

### 2. Stale `info.version`
The spec says `1.0.0` but the API is on version 3.2.1. Consumers generate clients against the wrong version expectations.

### 3. Empty Description
The `description` field is blank or contains only the title repeated. Documentation portals render an empty landing page.

### 4. Localhost-Only Servers
The `servers` array contains only `http://localhost:8000`. Consumers cannot discover the production URL from the spec.

### 5. Missing `x-api-id`
Without a stable identifier, API catalogs cannot track the API across version changes, renames, or repository moves.

### 6. Telegraphic Descriptions
`"Project API"` instead of `"REST API for creating, managing, and archiving projects within an Acme organization. Requires Bearer token authentication."` -- LLMs and humans both need context.

---

## Guardrails

1. **Never confuse `openapi` with `info.version`** -- they serve different purposes and change on different cadences
2. **Always validate server URLs** -- ensure they resolve and match deployed environments
3. **Always include authentication in the description** -- consumers need this before reading any endpoint
4. **Treat extensions as contracts** -- once published, `x-api-id` and `x-audience` cannot change without migration
5. **Write descriptions for both humans and LLMs** -- complete sentences, defined terms, explicit constraints
