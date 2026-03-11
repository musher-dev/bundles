---
name: governing-payload-structure
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of API payload document structure including top-level JSON object requirements, collection wrapping with items arrays, I-JSON compliance per RFC 7493, contract type taxonomy for state representations and commands and mutations, and semantic field ontology for identity attributes relationships metadata control and links. Use when designing request or response body shapes, reviewing JSON payload structure, auditing API contract types, or enforcing envelope conventions. Triggered by: payload structure, JSON envelope, top-level object, collection wrapper, I-JSON, RFC 7493, contract type, state representation, command payload, field ontology, request body shape, response body shape, payload design, document structure.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Payload Structure Governance

Comprehensive governance for API request and response body document structure, enforcing consistent top-level shapes, collection wrapping, I-JSON compliance, and semantic field organization.

## Purpose

The shape of a JSON payload is the most fundamental contract between client and server. Inconsistent envelope structures -- bare arrays in some endpoints, wrapped objects in others, missing metadata fields -- force consumers to write per-endpoint parsing logic instead of a single generic client. This skill encodes canonical document structure rules that make payloads predictable, extensible, and safe.

Use this skill when designing new request or response body shapes, reviewing existing payload structures, auditing API contract types, or implementing envelope conventions.

---

## Top-Level Document Shape

### Rule: Always a JSON Object

Every API request and response body must be a top-level JSON object (`{}`). Never return or accept a bare JSON array, string, number, or null at the top level.

| Correct | Wrong | Why |
|---------|-------|-----|
| `{"items": [...]}` | `[...]` | Bare arrays cannot be extended with metadata |
| `{"status": "ok"}` | `"ok"` | Bare strings cannot carry additional context |
| `{}` | *(empty body)* | Commands with no parameters still use empty object |

**Rationale:**
- Top-level arrays cannot be extended without breaking changes (no room for `meta`, `links`, pagination)
- Top-level arrays are vulnerable to JSON hijacking in older browsers (array constructor override)
- Top-level objects provide a stable extensibility point -- new fields can be added without breaking existing clients

### Single Resources

Return the resource directly as the top-level object:

```json
{
  "id": "a1b2c3d4-...",
  "name": "My Project",
  "created_at_time": "2024-01-15T09:30:00Z",
  "status": "ACTIVE"
}
```

No wrapping in a `data` key for single resources -- the resource *is* the document.

### Collections

Wrap collection results in an `items` array within the top-level object:

```json
{
  "items": [
    {"id": "a1b2c3d4-...", "name": "Project A"},
    {"id": "e5f6g7h8-...", "name": "Project B"}
  ],
  "next_page_token": "eyJjdXJzb3IiOi...",
  "total_count": 42
}
```

**Rules:**
- The array key is `items` (not `data`, `results`, or `records`)
- Pagination metadata lives as sibling keys (not nested in `meta`)
- An empty collection returns `{"items": [], ...}` -- never `null` or absent
- Cross-reference: `governing-collection-endpoints` defines pagination, filtering, and sorting conventions for collection responses

### Commands and Actions

Command payloads (custom operations via `:verb` syntax) use a top-level JSON object even when no parameters are needed:

```json
POST /projects/{id}:archive
Content-Type: application/json

{}
```

Commands with parameters include them as top-level fields:

```json
POST /projects/{id}:transfer
Content-Type: application/json

{
  "new_owner_id": "u9v8w7x6-..."
}
```

---

## I-JSON Compliance (RFC 7493)

### Internet JSON Profile

All payloads must conform to I-JSON (RFC 7493), a restricted profile of JSON designed for maximum interoperability:

| Requirement | Rationale |
|-------------|-----------|
| UTF-8 encoding (no BOM) | Universal encoding support |
| Unique object keys | Duplicate keys have undefined behavior across parsers |
| Numbers within IEEE 754 double range | Avoid precision loss in JavaScript and other IEEE 754 environments |
| No top-level values other than objects or arrays | Ensure extensibility (this skill further restricts to objects only) |

### Must-Ignore Policy

Consumers must ignore unrecognized fields. Producers may add new fields at any time without it constituting a breaking change.

**Implications:**
- Clients must not fail on unknown fields (use `model_config = ConfigDict(extra="ignore")` in Pydantic)
- Servers must not reject requests with extra fields unless security requires it
- New optional fields can be added to any response without versioning

---

## Contract Type Taxonomy

Every payload belongs to one of six contract types. Identifying the type determines which structure rules apply.

### 1. State Representation

The canonical form of a resource as the server knows it. Returned by GET endpoints.

```json
{
  "id": "a1b2c3d4-...",
  "name": "My Project",
  "status": "ACTIVE",
  "created_at_time": "2024-01-15T09:30:00Z",
  "updated_at_time": "2024-01-16T14:20:00Z"
}
```

**Rules:** Contains all fields the consumer is authorized to see. Uses `response_model_exclude_none=True` to omit null fields (absent-over-null). Cross-reference: `governing-field-lifecycle` for absent-over-null semantics.

### 2. Desired State (Create)

The fields needed to create a new resource. Sent as the body of POST (create) or PUT (full replace).

```json
{
  "name": "New Project",
  "description": "A project for testing"
}
```

**Rules:** Omits server-managed fields (`id`, `created_at_time`, `status`). Only includes fields the client controls. Uses a separate Pydantic model (e.g., `ProjectCreate`). Cross-reference: `governing-schema-implementation` for model naming conventions.

### 3. Partial Mutation (Patch)

A subset of mutable fields to update. Sent as the body of PATCH using JSON Merge Patch (RFC 7396).

```json
{
  "name": "Renamed Project"
}
```

**Rules:** Only includes fields being changed. Absent fields are untouched. `null` means "clear this field." Content-Type must be `application/merge-patch+json`. Cross-reference: `governing-mutation-semantics` for PATCH conventions.

### 4. Command / Action

Parameters for a custom operation. Does not map to a resource's field structure.

```json
{
  "target_organization_id": "o3p4q5r6-...",
  "transfer_members": true
}
```

**Rules:** Field names describe the operation's parameters, not resource attributes. Always a JSON object (even if empty `{}`). Validated by a dedicated Pydantic model (e.g., `TransferProjectCommand`).

### 5. Collection Result

A list of resources with pagination metadata. Returned by list endpoints.

```json
{
  "items": [...],
  "next_page_token": "...",
  "total_count": 42
}
```

**Rules:** Uses `items` array wrapper. Includes pagination metadata as sibling keys. Cross-reference: `governing-collection-endpoints` for full collection response conventions.

### 6. Problem Document

An error response following RFC 9457 Problem Details.

```json
{
  "type": "https://api.example.com/problems/not-found",
  "title": "Not Found",
  "status": 404,
  "detail": "Project a1b2c3d4 does not exist.",
  "instance": "/projects/a1b2c3d4"
}
```

**Rules:** Content-Type is `application/problem+json`. Cross-reference: `governing-error-responses` for full Problem Details conventions.

---

## Semantic Field Ontology

Fields within a payload fall into six semantic categories. Consistent placement and naming across these categories makes payloads self-describing.

### 1. Identity Fields

Uniquely identify the resource.

| Field | Type | Notes |
|-------|------|-------|
| `id` | string (UUID) | Server-generated, immutable |
| `slug` | string | Human-readable, unique, immutable after creation |

### 2. Attribute Fields

Describe the resource's state.

| Pattern | Example | Notes |
|---------|---------|-------|
| Domain nouns | `name`, `description`, `status` | Core business data |
| Qualified nouns | `display_name`, `email_address` | Disambiguated attributes |
| No prepositions | `start_time`, not `time_of_start` | Keep names concise |

### 3. Relationship Fields

Reference other resources by ID.

| Pattern | Example | Notes |
|---------|---------|-------|
| `{resource}_id` | `project_id`, `owner_id` | Foreign key reference |
| `{resource}_ids` | `member_ids` | Plural for array of references |

### 4. Metadata Fields

Server-managed lifecycle data.

| Field | Type | Notes |
|-------|------|-------|
| `created_at_time` | string (RFC 3339) | Immutable after creation |
| `updated_at_time` | string (RFC 3339) | Updated on every mutation |
| `created_by_id` | string (UUID) | Actor who created the resource |

### 5. Control Fields

Affect processing behavior without being part of the resource's state.

| Pattern | Example | Notes |
|---------|---------|-------|
| `dry_run` | boolean | Validate without persisting |
| `idempotency_key` | string | Ensure exactly-once processing |
| `expand` | array of strings | Include related resources inline |

### 6. Link Fields

Provide navigational URLs for related resources or actions.

| Pattern | Example | Notes |
|---------|---------|-------|
| `self` | URL to this resource | HATEOAS self-link |
| `{relation}` | URL to related resource | Named by relationship |

---

## Review Dimensions

When reviewing or auditing payload structure, evaluate these dimensions:

### 1. Top-Level Shape
- Is every response body a JSON object (never a bare array)?
- Do single resources return the object directly (no unnecessary wrapping)?
- Do collections use the `items` array wrapper?
- Do commands use a JSON object body (even if empty)?

### 2. I-JSON Compliance
- Is the encoding UTF-8 without BOM?
- Are all object keys unique within their parent?
- Are large numbers handled safely (or represented as strings)?
- Do consumers use must-ignore for unrecognized fields?

### 3. Contract Type Identification
- Can each payload be classified into exactly one contract type?
- Are separate Pydantic models used for create, update, patch, and response?
- Are command payloads distinct from resource representations?

### 4. Field Ontology
- Do identity fields use `id` and `slug` conventions?
- Do relationship fields use the `{resource}_id` pattern?
- Do metadata fields use `_time` suffixes for timestamps?
- Are control fields clearly separated from resource attributes?

---

## Anti-Patterns

### 1. Top-Level Arrays
Returning `[{"id": "..."}]` instead of `{"items": [{"id": "..."}]}`. Impossible to extend with pagination metadata without a breaking change.

### 2. Inconsistent Collection Keys
Using `data` in some endpoints, `results` in others, and `items` elsewhere. Pick `items` and enforce it everywhere.

### 3. Mixed Contract Types
Using the same Pydantic model for creation input and response output. This leaks server-managed fields into creation requests and creates mass-assignment vulnerabilities.

### 4. Nested Metadata Wrappers
Wrapping responses in `{"data": {...}, "meta": {...}}` for single resources. This adds unnecessary nesting. Reserve wrappers for collections.

### 5. Absent Empty Collections
Returning `null` or omitting the `items` key when a collection is empty. Always return `{"items": []}`.

---

## Examples

### Example 1: API Returning Bare Arrays

**Scenario:** `GET /projects` returns `[{"id": "..."}, ...]` as a bare JSON array.

**Findings:**
- **Top-Level Shape: FAIL** -- Response is a bare array, not a JSON object
- **Recommendation:** Wrap in `{"items": [...], "next_page_token": "...", "total_count": N}`

### Example 2: Inconsistent Envelope Structure

**Scenario:** `GET /projects` returns `{"data": [...]}`, `GET /users` returns `{"results": [...]}`, `GET /teams` returns `{"items": [...]}`.

**Findings:**
- **Top-Level Shape: FAIL** -- Three different collection wrapper keys across the API
- **Recommendation:** Standardize all collections to use `items` as the array key

### Example 3: Well-Structured Payloads

**Scenario:** All single resources return as top-level objects, collections use `{"items": [...]}`, commands use JSON objects, separate Pydantic models per contract type, `_id` suffix for relationships, `_time` suffix for timestamps.

**Findings:**
- All four dimensions: **PASS**
