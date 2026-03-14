---
name: governing-spec-extensions
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of OpenAPI specification extensions including x- prefix conventions, extension governance policies, portability impact assessment, standards-over-extensions philosophy, common vendor extensions (x-tagGroups, x-codeSamples, x-internal), deprecating extensions when standards catch up, and extension registry management. Use when designing custom extensions, reviewing x- prefixed fields, auditing extension portability, migrating extensions to standards, or managing an extension registry. Triggered by: x- extension, spec extension, vendor extension, x-tagGroups, x-codeSamples, x-internal, extension governance, extension registry, portability, custom extension, extension deprecation, standards migration.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Spec Extension Governance

Comprehensive governance for OpenAPI specification extensions -- the `x-` prefix convention, extension lifecycle, portability impact, and the philosophy of standards over vendor-specific workarounds.

## Purpose

OpenAPI extensions (`x-` prefixed fields) let organizations add custom metadata to specs. Used well, they enable API catalogs, governance dashboards, and custom tooling. Used poorly, they create vendor lock-in, break portability, and accumulate as dead metadata. This skill enforces a standards-first philosophy: prefer native OpenAPI fields, use extensions only when no standard exists, and deprecate extensions when standards catch up.

Use this skill when adding custom fields to OpenAPI specs, auditing existing extensions, or migrating vendor extensions to standard fields.

---

## Extension Fundamentals

### `x-` Prefix Convention

Any field in an OpenAPI document starting with `x-` is an extension:

```yaml
info:
  title: "Acme API"
  x-api-id: "acme-platform"
  x-audience: "external-public"

paths:
  /projects:
    get:
      x-internal: false
      x-rate-limit: 1000
```

### Where Extensions Are Allowed

Extensions can appear on any object in the OpenAPI document:

| Location | Common Extensions | Purpose |
|----------|------------------|---------|
| `info` | `x-api-id`, `x-audience`, `x-api-owner` | API identity and governance |
| `paths.{path}.{method}` | `x-internal`, `x-rate-limit`, `x-codeSamples` | Operation metadata |
| `tags` | `x-kind`, `x-parent` | Tag classification (pre-3.2.0) |
| `components.schemas` | `x-examples`, `x-deprecated-since` | Schema metadata |
| Root level | `x-tagGroups` | Documentation grouping |

### Extension Naming Rules

| Rule | Example | Rationale |
|------|---------|-----------|
| Always start with `x-` | `x-api-id` | Required by spec, signals non-standard |
| Use kebab-case | `x-rate-limit` not `x-rateLimit` | Consistent with YAML key conventions |
| Prefix with org name for org-specific extensions | `x-acme-billing-tier` | Prevents collision with vendor extensions |
| Keep names descriptive | `x-deprecation-date` not `x-dd` | Self-documenting metadata |

---

## Standards-Over-Extensions Philosophy

### The Decision Framework

Before adding an extension, ask:

```
1. Does OpenAPI 3.1.0 have a native field for this? → Use the native field
2. Does OpenAPI 3.2.0 add a native field for this? → Use an extension that maps cleanly to 3.2.0
3. Does a widely-adopted vendor extension exist? → Prefer the de facto standard
4. None of the above? → Create an organizational extension with governance
```

### Why Standards First

| Approach | Tooling Support | Portability | Maintenance |
|----------|----------------|-------------|-------------|
| Native field | Universal | Full | Zero |
| Widely-adopted extension | Most tools | High | Low |
| Org-specific extension | Custom tooling only | None | High |

### Extension-to-Standard Migration Examples

| Extension | Replaced By | Version |
|-----------|------------|---------|
| `x-nullable: true` | `type: ["string", "null"]` | 3.1.0 |
| `x-webhooks` | `webhooks` | 3.1.0 |
| `x-tagGroups` | `parent` field on tags | 3.2.0 |
| `x-summary` on tags | `summary` field on tags | 3.2.0 |

---

## Common Vendor Extensions

### Documentation Tools

| Extension | Tool | Purpose |
|-----------|------|---------|
| `x-tagGroups` | Redocly | Group tags in sidebar navigation |
| `x-codeSamples` | Redocly, ReadMe | Embed code samples in docs |
| `x-logo` | Redocly | Custom logo for documentation portal |
| `x-displayName` | Redocly | Override tag display name |

### Governance Extensions

| Extension | Purpose | Example Value |
|-----------|---------|---------------|
| `x-api-id` | Stable API identifier for catalogs | `"acme-platform"` |
| `x-audience` | Consumer audience classification | `"external-public"` |
| `x-api-owner` | Owning team identifier | `"platform-team"` |
| `x-api-lifecycle` | Current lifecycle stage | `"production"` |

### SDK Generation Extensions

| Extension | Generator | Purpose |
|-----------|-----------|---------|
| `x-enum-varnames` | OpenAPI Generator | Custom enum constant names |
| `x-enum-descriptions` | OpenAPI Generator | Descriptions for enum values |
| `x-group-parameters` | OpenAPI Generator | Group parameters into objects |

### Operational Extensions

| Extension | Purpose | Example Value |
|-----------|---------|---------------|
| `x-internal` | Mark endpoint as internal-only | `true` |
| `x-rate-limit` | Document rate limit | `1000` |
| `x-deprecated-since` | Track deprecation date | `"2024-06-01"` |
| `x-sunset-date` | Track sunset date | `"2025-06-01"` |

---

## Extension Governance Policy

### Extension Registry

Maintain a central registry of all approved extensions:

```yaml
# extension-registry.yaml
extensions:
  - name: x-api-id
    scope: info
    type: string
    required: true
    description: "Stable API identifier for the organization catalog"
    owner: "api-governance"
    added: "2024-01-15"

  - name: x-audience
    scope: info
    type: string
    required: true
    enum: [external-public, external-partner, internal, team-internal]
    description: "Consumer audience classification"
    owner: "api-governance"
    added: "2024-01-15"

  - name: x-internal
    scope: operation
    type: boolean
    required: false
    default: false
    description: "Marks operation as internal-only (excluded from public docs)"
    owner: "api-governance"
    added: "2024-03-01"
```

### Extension Lifecycle

| Stage | Actions |
|-------|---------|
| **Proposed** | Extension is documented in a proposal with rationale |
| **Approved** | Added to the registry, tooling updated to consume it |
| **Active** | Used across API specs, enforced by CI |
| **Deprecated** | Standard alternative exists, migration plan published |
| **Retired** | Removed from specs, CI warns on presence |

### Approval Criteria

Before approving a new extension, verify:

1. **No native field exists** in current or upcoming OpenAPI versions
2. **No widely-adopted vendor extension** covers the same need
3. **Clear consumer benefit** -- who reads this extension and what do they do with it?
4. **Tooling commitment** -- at least one tool will consume it
5. **Naming follows conventions** -- `x-` prefix, kebab-case, descriptive

---

## Portability Impact Assessment

### Impact Levels

| Level | Description | Example |
|-------|-------------|---------|
| **None** | Extension is purely informational, ignored by standard tools | `x-api-owner` |
| **Low** | Extension enhances display but standard behavior is acceptable without it | `x-codeSamples` |
| **Medium** | Extension changes behavior in specific tools, workarounds exist | `x-tagGroups` |
| **High** | Extension is required for correct SDK generation or tooling behavior | `x-enum-varnames` |

### Portability Rules

| Rule | Rationale |
|------|-----------|
| Never make spec validity depend on extensions | Standard validators must pass without extensions |
| Document fallback behavior | What happens when a tool ignores the extension? |
| Avoid semantic extensions | Extensions that change the meaning of standard fields create confusion |
| Test with vanilla tools | Validate the spec with tools that don't support the extension |

---

## Deprecating Extensions

### When to Deprecate

An extension should be deprecated when:
- A native OpenAPI field now covers the same functionality
- A more widely-adopted vendor extension replaces it
- The tooling that consumed it has been retired
- The organizational need no longer exists

### Deprecation Process

1. **Mark as deprecated** in the extension registry with the migration path
2. **Update CI rules** to warn (not fail) on deprecated extension usage
3. **Publish migration guide** showing before/after examples
4. **Set retirement date** (typically one major API version cycle)
5. **Remove from specs** after the retirement date
6. **Update CI rules** to fail on presence of retired extensions

### Example: `x-nullable` to Native Nullability

```yaml
# Before (3.0.x with extension):
properties:
  middle_name:
    type: string
    x-nullable: true

# After (3.1.0 with native field):
properties:
  middle_name:
    type: ["string", "null"]
```

---

## Review Dimensions

When reviewing or auditing spec extensions, evaluate these dimensions:

### 1. Extension Necessity
- Does a native OpenAPI field exist for this purpose?
- Is there a widely-adopted vendor extension?
- Is the extension in the organization's approved registry?

### 2. Naming Compliance
- Does the extension start with `x-`?
- Is it kebab-case?
- Does it use an org prefix for org-specific extensions?
- Is the name descriptive and unambiguous?

### 3. Portability Impact
- What happens when tools ignore this extension?
- Does the spec remain valid without it?
- Is the fallback behavior documented?

### 4. Lifecycle Status
- Is the extension active, deprecated, or retired?
- Are deprecated extensions tagged with migration paths?
- Are there retired extensions still present in the spec?

### 5. Registry Compliance
- Is every extension used in the spec registered?
- Are unregistered extensions flagged for approval or removal?
- Is the registry up to date with current usage?

---

## Anti-Patterns

### 1. Extension Sprawl
Adding `x-` fields ad hoc without registry governance. Extensions accumulate as dead metadata that no tool reads.

### 2. Semantic Overloading
Using `x-` fields to change the meaning of standard fields (e.g., `x-required: true` instead of listing in `required` array). Breaks standard tooling expectations.

### 3. Vendor Lock-In
Depending on vendor-specific extensions for core functionality without documenting fallback behavior. Switching documentation tools becomes a spec rewrite.

### 4. Zombie Extensions
Extensions from deprecated tools or abandoned initiatives that persist in specs indefinitely. Regular audits should clean these.

### 5. Undocumented Extensions
Extensions without registry entries, descriptions, or consumer documentation. No one knows what they do or whether they can be removed.

### 6. Premature Extension
Creating an org-specific extension when a widely-adopted vendor extension already exists for the same purpose. Prefer de facto standards.

---

## Guardrails

1. **Standards first** -- never add an extension when a native field exists
2. **Every extension must be in the registry** -- unapproved extensions are technical debt
3. **Document fallback behavior** -- the spec must be valid without extensions
4. **Deprecate proactively** -- when a standard catches up, start the migration immediately
5. **Audit quarterly** -- find and remove zombie extensions
6. **Prefer widely-adopted over custom** -- `x-tagGroups` over `x-acme-tag-groups`
