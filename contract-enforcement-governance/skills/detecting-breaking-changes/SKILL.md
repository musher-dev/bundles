---
name: detecting-breaking-changes
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of breaking change detection for API contracts using oasdiff integration, breaking change classification with ERR and WARN severity levels, backward compatibility rules for request and response schemas, deprecation window enforcement with sunset headers, and CI gate configuration for merge blocking. Use when configuring breaking change detection, auditing backward compatibility, reviewing deprecation policies, or setting up oasdiff CI gates. Triggered by: breaking change, oasdiff, backward compatibility, deprecation, sunset header, API versioning, schema evolution, breaking diff, contract diff, API diff, compatibility check, semver.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Breaking Change Detection

Comprehensive governance for detecting, classifying, and preventing breaking changes in API contracts using automated diffing and CI enforcement.

## Purpose

Breaking changes are the most expensive defect in API-driven systems. A removed field, a tightened validation, or a changed response structure silently breaks every consumer. Manual review cannot reliably catch breaks -- a 200-line spec diff may hide a single property removal that breaks production clients. This skill enforces automated breaking change detection using oasdiff, classification of changes by severity, and CI gates that block merges until breaks are resolved or acknowledged.

**Boundary:** This skill governs detection and classification of contract changes. For the CI pipeline that orchestrates detection alongside other stages, see `governing-enforcement-pipeline`. For deprecation mechanics within the spec itself, see `openapi-specification-governance/governing-spec-metadata`.

Use this skill when configuring breaking change detection, auditing backward compatibility, or reviewing spec evolution practices.

---

## Breaking Change Taxonomy

### What Constitutes a Breaking Change

| Change Type | Breaking? | Severity | Example |
|------------|-----------|----------|---------|
| Remove endpoint | Yes | ERR | `DELETE /v1/projects/{id}` path removed |
| Remove response field | Yes | ERR | `name` removed from `ProjectResponse` |
| Add required request field | Yes | ERR | New required `region` on `ProjectCreate` |
| Narrow enum values | Yes | ERR | Remove `draft` from status enum |
| Tighten validation | Yes | ERR | Change `maxLength: 255` to `maxLength: 100` |
| Change field type | Yes | ERR | `id` from `string` to `integer` |
| Remove response status code | Yes | ERR | `204` response removed from DELETE |
| Add optional request field | No | - | New optional `description` field |
| Add response field | No | - | New `created_by` in response |
| Widen enum values | No | - | Add `archived` to status enum |
| Loosen validation | No | - | Change `minLength: 3` to `minLength: 1` |
| Add new endpoint | No | - | New `GET /v1/analytics` path |
| Deprecate (without removal) | No | WARN | `deprecated: true` on field |

### The Compatibility Rule

**Request schemas follow contravariance**: inputs can be loosened (accept more) but not tightened (reject previously valid input).

**Response schemas follow covariance**: outputs can be extended (add fields) but not narrowed (remove fields consumers may depend on).

```
Request:  New schema must accept everything old schema accepted
Response: New schema must include everything old schema included
```

---

## oasdiff Integration

### Installation and Basic Usage

```bash
# Install
go install github.com/tufin/oasdiff@latest

# Basic breaking change check
oasdiff breaking base.yaml revision.yaml

# Check against a URL (e.g., production spec)
oasdiff breaking https://api.example.com/openapi.json revision.yaml

# Output formats
oasdiff breaking base.yaml revision.yaml --format json
oasdiff breaking base.yaml revision.yaml --format yaml
oasdiff breaking base.yaml revision.yaml --format text
```

### Change Classification

oasdiff classifies changes into two severity levels:

| Level | Meaning | CI Action | Example |
|-------|---------|-----------|---------|
| **ERR** | Confirmed breaking change | Block merge | Removed field, tightened validation |
| **WARN** | Potentially breaking change | Require review | Deprecation added, description changed |

### CI Exit Codes

| Exit Code | Meaning |
|-----------|---------|
| 0 | No breaking changes |
| 1 | Breaking changes detected (ERR) |
| Other | Tool error |

### Common oasdiff Checks

```bash
# Breaking changes only (ERR level)
oasdiff breaking base.yaml revision.yaml --fail-on ERR

# Breaking changes and warnings
oasdiff breaking base.yaml revision.yaml --fail-on WARN

# Full changelog (all changes, not just breaking)
oasdiff changelog base.yaml revision.yaml

# Diff summary
oasdiff diff base.yaml revision.yaml --format json
```

---

## Deprecation Window Enforcement

### Deprecation-Before-Removal Policy

No field, endpoint, or enum value may be removed without passing through a deprecation window:

```
1. Mark deprecated    →  2. Wait N releases  →  3. Remove
   (deprecated: true)     (sunset window)        (breaking change, announced)
```

### Sunset Header Convention

```yaml
# In the OpenAPI spec
paths:
  /v1/legacy-endpoint:
    get:
      deprecated: true
      x-sunset: "2025-06-01"
      responses:
        "200":
          headers:
            Sunset:
              schema:
                type: string
                format: date
              description: "Date after which this endpoint will be removed"
```

### Deprecation Rules

| Rule | Rationale |
|------|-----------|
| Deprecation must be announced at least 2 minor versions before removal | Gives consumers time to migrate |
| `deprecated: true` must be set on the spec element | Machine-readable deprecation signal |
| `x-sunset` extension should specify the removal date | Consumers can automate migration planning |
| Sunset header must be returned in API responses | Runtime signal for consumers still calling deprecated endpoints |
| Removal without prior deprecation is always ERR severity | Breaking consumer trust is more expensive than the migration cost |

### oasdiff Deprecation Checks

```bash
# Check for removals without prior deprecation
oasdiff breaking base.yaml revision.yaml --deprecation-days-beta 31 --deprecation-days-stable 180
```

| Flag | Meaning |
|------|---------|
| `--deprecation-days-beta` | Minimum days between deprecation and removal for beta APIs |
| `--deprecation-days-stable` | Minimum days between deprecation and removal for stable APIs |

---

## Backward Compatibility Rules

### Request Schema Evolution

| Change | Compatible? | Rule |
|--------|------------|------|
| Add optional property | Yes | New property has default or is optional |
| Add required property | No | Existing clients do not send it |
| Remove property | Yes | Server ignores unknown properties (if configured) |
| Widen string maxLength | Yes | Accepts more input |
| Narrow string maxLength | No | Rejects previously valid input |
| Widen enum | Yes | Accepts more values |
| Narrow enum | No | Rejects previously valid values |
| Change type | No | Existing payloads will not match |

### Response Schema Evolution

| Change | Compatible? | Rule |
|--------|------------|------|
| Add property | Yes | Clients should ignore unknown properties |
| Remove property | No | Clients may depend on it |
| Make nullable property non-nullable | Yes | Reduces null-handling burden |
| Make non-nullable property nullable | No | Clients may not handle null |
| Widen enum | No | Clients may not handle new values |
| Narrow enum | Yes | Reduces cases clients must handle |
| Change type | No | Client deserialization will fail |

### Header and Parameter Evolution

| Change | Compatible? | Rule |
|--------|------------|------|
| Add optional query parameter | Yes | Existing requests still valid |
| Add required query parameter | No | Existing requests will fail |
| Remove query parameter | Yes | Server ignores it |
| Add response header | Yes | Clients ignore unknown headers |
| Remove response header | No | Clients may depend on it |

---

## CI Gate Configuration

### GitHub Actions

```yaml
name: Contract Breaking Change Check

on:
  pull_request:
    paths:
      - "packages/contract/openapi/**"
      - "packages/contract/generated/openapi.bundled.json"

jobs:
  breaking-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Get base spec
        run: git show origin/main:packages/contract/generated/openapi.bundled.json > /tmp/base-spec.json

      - name: Bundle current spec
        run: npx @redocly/cli bundle packages/contract/openapi/openapi.yaml -o /tmp/current-spec.json

      - name: Check breaking changes
        uses: tufin/oasdiff-action/breaking@main
        with:
          base: /tmp/base-spec.json
          revision: /tmp/current-spec.json
          fail-on: ERR
```

### Gate Behavior

| Mode | ERR Behavior | WARN Behavior | Use Case |
|------|-------------|---------------|----------|
| **Strict** | Block merge | Block merge | Production APIs with external consumers |
| **Standard** | Block merge | Require review comment | Internal APIs with coordinated consumers |
| **Advisory** | Require review comment | Log only | Early development, pre-launch APIs |

---

## Review Dimensions

When reviewing or auditing breaking change detection, evaluate these dimensions:

### 1. Detection Coverage
- Is oasdiff (or equivalent) integrated into CI?
- Does the check run on every PR that modifies spec files?
- Is the base spec sourced from the correct reference (main branch or production)?

### 2. Classification Accuracy
- Are ERR findings genuinely breaking?
- Are WARN findings appropriately flagged for review?
- Are there false positives that should be suppressed?

### 3. Deprecation Compliance
- Do removed elements have prior deprecation records?
- Are deprecation windows enforced (minimum days between deprecation and removal)?
- Are sunset headers configured for deprecated endpoints?

### 4. Gate Effectiveness
- Does the gate mode match the API's consumer profile?
- Are blocked PRs getting resolved (not just bypassed)?
- Is there an escalation path for intentional breaking changes?

---

## Anti-Patterns

### 1. Manual Breaking Change Review
Relying on PR reviewers to spot breaking changes in spec diffs. Humans miss subtle breaks (enum narrowing, type changes nested deep in schemas).

### 2. Remove-Without-Deprecate
Removing endpoints or fields without a preceding deprecation period. Breaks consumer trust and forces emergency migrations.

### 3. Base Spec From Wrong Source
Comparing against a stale or incorrect base spec. The base must be the currently deployed/released spec, not a random earlier version.

### 4. Blanket WARN Suppression
Suppressing all WARN-level findings to reduce noise. Warnings exist to catch potentially breaking changes that need human judgment.

### 5. No Escalation Path
No process for intentional breaking changes. Sometimes breaking changes are necessary -- the system needs a documented path for acknowledging and communicating them.

---

## Guardrails

1. **Every spec change PR must pass automated breaking change detection** -- manual review alone is insufficient
2. **ERR-level breaking changes must block merge by default** -- intentional breaks require explicit acknowledgment
3. **Removal requires prior deprecation** -- enforce minimum deprecation windows via oasdiff flags
4. **Base spec must be sourced from the deployment reference** -- main branch or production, not an arbitrary version
5. **WARN-level findings require human review, not suppression** -- warnings exist to catch edge cases
6. **Intentional breaking changes must be documented** -- changelog entry, migration guide, consumer notification
