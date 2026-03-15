---
name: auditing-spec-compliance
version: 1.0.0
user-invocable: false
description: Guide structured audit execution for OpenAPI specification compliance using a 9-category question bank covering Spec Metadata, Operation Identifiers, Tag Hierarchy, Component Reuse, Advanced Operations, Extension Governance, Spec Structure, Authoring Model, and Change Workflow, with Pass/Warn/Fail severity model mapped to Critical/Major/Minor/Cosmetic levels, compliance scorecard, Spectral/Redocly/Vacuum ruleset recommendations, and CI pipeline integration patterns. Use when conducting OpenAPI spec audits, reviewing specification compliance, auditing spec quality, prioritizing spec technical debt, or setting up CI spec governance. Triggered by: spec audit, OpenAPI audit, specification audit, spec compliance, spec quality, spec linting, Spectral rules, Redocly, Vacuum, spec governance, spec scorecard, spec quality gate, specification review.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Spec Compliance Auditing

Structured audit execution framework for OpenAPI specification compliance, providing a 9-category question bank, severity model, and CI tooling recommendations.

## Purpose

Ad-hoc spec reviews produce inconsistent results -- one reviewer checks operationIds, another checks schemas, a third skips both and focuses on tags. This skill provides a deterministic audit framework: a structured question bank that covers every governance dimension (including spec structure, authoring model, and change workflow), a severity model that prioritizes findings, and CI tool recommendations that automate enforcement. The result is reproducible, comprehensive audits that produce actionable, severity-prioritized reports.

Use this skill when conducting OpenAPI specification audits, reviewing compliance against governance standards, prioritizing spec technical debt, or setting up automated CI governance.

---

## Audit Question Bank

### Category 1: Spec Metadata

Questions about the Info object, version fields, servers array, extensions, and external documentation.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| M1 | Is `openapi` set to `3.1.0` (not defaulting to 3.0.x)? | governing-spec-metadata | Major |
| M2 | Does `info.version` follow semantic versioning and match the deployed API? | governing-spec-metadata | Major |
| M3 | Are all required Info fields populated (title, summary, description, version)? | governing-spec-metadata | Major |
| M4 | Does the description include authentication requirements? | governing-spec-metadata | Minor |
| M5 | Does the description include rate limiting policy? | governing-spec-metadata | Minor |
| M6 | Is the description LLM-readable (complete sentences, defined terms)? | governing-spec-metadata | Cosmetic |
| M7 | Is at least one server listed with production first? | governing-spec-metadata | Major |
| M8 | Do all servers have descriptions? | governing-spec-metadata | Minor |
| M9 | Is `x-api-id` set with a stable, unique identifier? | governing-spec-metadata | Minor |
| M10 | Is `x-audience` declared with a valid value? | governing-spec-metadata | Minor |
| M11 | Does `externalDocs` link to a live documentation page? | governing-spec-metadata | Minor |

### Category 2: Operation Identifiers

Questions about operationId naming, uniqueness, schema keys, parameter naming, and SDK generation readiness.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| O1 | Does every operation have an explicit `operationId`? | governing-operation-identifiers | Critical |
| O2 | Are all operationIds globally unique across the spec? | governing-operation-identifiers | Critical |
| O3 | Do operationIds follow camelCase verbNoun convention? | governing-operation-identifiers | Major |
| O4 | Are verb prefixes from the controlled vocabulary (list, get, create, update, delete)? | governing-operation-identifiers | Minor |
| O5 | Do all schema component keys follow UpperCamelCase (`^[A-Z][a-zA-Z0-9]+$`)? | governing-operation-identifiers | Major |
| O6 | Do schema names follow the `{Resource}{Suffix}` convention? | governing-operation-identifiers | Minor |
| O7 | Are query parameters snake_case? | governing-operation-identifiers | Major |
| O8 | Are header parameters Kebab-Title-Case? | governing-operation-identifiers | Minor |
| O9 | Are inline schemas extracted into named components? | governing-operation-identifiers | Major |
| O10 | Would generated SDK function names be self-documenting? | governing-operation-identifiers | Minor |

### Category 3: Tag Hierarchy

Questions about tag coverage, definitions, classification, and hierarchy structure.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| T1 | Does every operation have at least one tag? | governing-tag-hierarchy | Major |
| T2 | Is there exactly one navigation tag per operation? | governing-tag-hierarchy | Minor |
| T3 | Is every used tag defined in the top-level `tags` array? | governing-tag-hierarchy | Major |
| T4 | Does every tag definition have a description? | governing-tag-hierarchy | Minor |
| T5 | Are tag names PascalCase plural nouns? | governing-tag-hierarchy | Minor |
| T6 | Are tags classified by kind (navigation, badge, lifecycle, audience)? | governing-tag-hierarchy | Cosmetic |
| T7 | Does the hierarchy have at most 3 levels? | governing-tag-hierarchy | Minor |
| T8 | Is there a migration plan from `x-tagGroups` to 3.2.0 hierarchical tags? | governing-tag-hierarchy | Cosmetic |

### Category 4: Component Reuse

Questions about $ref usage, inline schemas, composition patterns, and error response reuse.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| R1 | Do all request body schemas use `$ref` (no inline definitions)? | governing-component-reuse | Major |
| R2 | Do all response schemas use `$ref` (no inline definitions)? | governing-component-reuse | Major |
| R3 | Are schemas used in 2+ places extracted to components? | governing-component-reuse | Major |
| R4 | Do all `$ref` values resolve to existing components? | governing-component-reuse | Critical |
| R5 | Is `ProblemDetail` (RFC 9457) defined as a reusable schema? | governing-component-reuse | Major |
| R6 | Are standard error responses (401, 403, 404, 422) reusable response components? | governing-component-reuse | Minor |
| R7 | Do all `oneOf` unions have a `discriminator`? | governing-component-reuse | Major |
| R8 | Is `allOf` depth limited to 2 levels? | governing-component-reuse | Minor |
| R9 | Is `anyOf` avoided in favor of `oneOf`? | governing-component-reuse | Cosmetic |
| R10 | Are circular `$ref` chains intentional (recursive data types)? | governing-component-reuse | Major |

### Category 5: Advanced Operations

Questions about QUERY method, webhooks, callbacks, streaming, and link objects.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| A1 | Are complex search operations using QUERY or documented POST workaround? | governing-advanced-operations | Minor |
| A2 | Are webhook event types documented in the top-level `webhooks` object? | governing-advanced-operations | Major |
| A3 | Do webhook payloads include event_id, event_type, and occurred_at? | governing-advanced-operations | Major |
| A4 | Is the webhook retry policy documented? | governing-advanced-operations | Minor |
| A5 | Do streaming endpoints use correct media types (text/event-stream, application/jsonl)? | governing-advanced-operations | Major |
| A6 | Is `itemSchema` or `x-itemSchema` provided for streaming responses? | governing-advanced-operations | Major |
| A7 | Do resource endpoints include link objects for related operations? | governing-advanced-operations | Cosmetic |
| A8 | Do all link operationIds reference existing operations? | governing-advanced-operations | Major |
| A9 | Are callbacks tied to the correct initiating operation? | governing-advanced-operations | Major |
| A10 | Is the Overlay specification used for audience-specific spec variants? | governing-advanced-operations | Cosmetic |

### Category 6: Extension Governance

Questions about x- prefix usage, extension registry, portability, and lifecycle.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| E1 | Do all extensions start with `x-` prefix? | governing-spec-extensions | Critical |
| E2 | Are extension names kebab-case? | governing-spec-extensions | Minor |
| E3 | Is every extension used in the spec registered in the extension registry? | governing-spec-extensions | Major |
| E4 | Are there deprecated extensions still in active use? | governing-spec-extensions | Major |
| E5 | Are there extensions for which native OpenAPI fields now exist? | governing-spec-extensions | Major |
| E6 | Is the spec valid without extensions (standard validators pass)? | governing-spec-extensions | Critical |
| E7 | Is fallback behavior documented for each extension? | governing-spec-extensions | Minor |
| E8 | Are org-specific extensions prefixed with the org name? | governing-spec-extensions | Cosmetic |

### Category 7: Spec Structure

Questions about multi-file layout, root file conventions, `$ref` strategies, and bundling.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| F1 | Is the spec split into multiple files when it exceeds 500 lines or 10 operations? | governing-spec-structure | Minor |
| F2 | Does the root file contain only metadata and `$ref` pointers (no inline definitions)? | governing-spec-structure | Major |
| F3 | Is one `$ref` strategy used consistently across all files? | governing-spec-structure | Major |
| F4 | Are all cross-file references using relative paths? | governing-spec-structure | Major |
| F5 | Is bundling configured to produce a single resolved file? | governing-spec-structure | Major |
| F6 | Is the bundle output in a `generated/` directory (separate from source)? | governing-spec-structure | Minor |
| F7 | Do downstream tools consume the bundle, not the source files? | governing-spec-structure | Major |
| F8 | Are path files organized one per resource domain? | governing-spec-structure | Minor |

### Category 8: Authoring Model

Questions about authoring model declaration, authority flow, source-of-truth policy, and maintenance tax.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| W1 | Is the authoring model (design-first, code-first, hybrid) explicitly declared? | governing-authoring-model | Major |
| W2 | Is the source-of-truth governance policy committed to the repository? | governing-authoring-model | Major |
| W3 | Is authority flow unidirectional or explicitly bounded with documented rules? | governing-authoring-model | Major |
| W4 | If dual-schema maintenance exists, is automated drift detection in place? | governing-authoring-model | Critical |
| W5 | Has the authoring model been reviewed since the last major team or consumer change? | governing-authoring-model | Minor |
| W6 | Is the authoring model enforceable by CI (not just a policy document)? | governing-authoring-model | Minor |

### Category 9: Change Workflow

Questions about Golden Path compliance, PR conventions, cross-team coordination, and backward compatibility.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| G1 | Do spec changes follow the Golden Path sequence (design → generate → implement → verify → deploy)? | governing-change-workflow | Major |
| G2 | Do spec-change PRs include a contract impact checklist? | governing-change-workflow | Minor |
| G3 | Do breaking changes require consumer acknowledgment before merge? | governing-change-workflow | Critical |
| G4 | Does deprecation precede removal by at least 2 minor versions? | governing-change-workflow | Major |
| G5 | Are cross-team coordination patterns used for changes affecting 3+ consumers? | governing-change-workflow | Minor |
| G6 | Are backward-compatible change techniques used by default? | governing-change-workflow | Major |
| G7 | Do verification failures loop back to design (not forward to deploy)? | governing-change-workflow | Major |
| G8 | Are spec-change PRs separate from implementation PRs? | governing-change-workflow | Minor |

---

## Severity Model

### Severity Levels

| Level | Definition | Timeframe | Examples |
|-------|------------|-----------|---------|
| **Critical** | Breaks SDK generation, causes spec validation failure, or creates security risk | Fix before next release | Missing operationId, broken $ref, extension without x- prefix |
| **Major** | Standards violation that degrades developer experience or causes integration issues | Fix within current sprint | Missing response schemas, inline body definitions, oneOf without discriminator |
| **Minor** | Convention deviation that reduces consistency but does not cause failures | Fix within current quarter | Missing tag descriptions, non-standard verb prefix, absent externalDocs |
| **Cosmetic** | Style preference or optimization with minimal functional impact | Add to backlog | LLM-readability, hierarchy migration plan, link objects |

### Pass/Warn/Fail Rating

Each audit question receives a rating:

| Rating | Criteria |
|--------|----------|
| **Pass** | Fully compliant with the governance standard |
| **Warn** | Partially compliant or compliant with caveats |
| **Fail** | Non-compliant with the governance standard |

### Overall Compliance Score

Calculate compliance percentage per category and overall:

```
Category Score = (Pass count + 0.5 * Warn count) / Total questions * 100
Overall Score = Average of all category scores
```

| Overall Score | Rating | Interpretation |
|--------------|--------|----------------|
| 90-100% | Excellent | Minor refinements only |
| 75-89% | Good | Some standards gaps to address |
| 50-74% | Needs Work | Significant compliance gaps |
| Below 50% | Critical | Fundamental governance failures |

---

## Audit Process

### Step 1: Discovery

Locate all OpenAPI specification files in the codebase:
- Glob for spec files (`**/openapi.yaml`, `**/openapi.json`, `**/openapi.yml`, `**/swagger.yaml`, `**/swagger.json`)
- Glob for spec fragments (`**/schemas/**/*.yaml`, `**/paths/**/*.yaml`)
- Glob for overlay files (`**/overlay*.yaml`)
- Glob for linting config (`**/.spectral.yaml`, `**/.redocly.yaml`, `**/vacuum.conf`)
- Identify spec version (3.0.x, 3.1.0, 3.2.0)
- Count total operations, schemas, tags, and extensions

### Step 2: Spec Metadata Audit

Walk through questions M1-M11. For each:
1. Examine the `info`, `openapi`, `servers`, and `externalDocs` objects
2. Check for organizational extensions
3. Evaluate description quality for LLM readability
4. Assign Pass/Warn/Fail rating

### Step 3: Operation Identifiers Audit

Walk through questions O1-O10. For each:
1. Collect all operationIds and check uniqueness
2. Verify naming convention compliance
3. Check schema component key casing
4. Evaluate SDK generation readiness
5. Assign Pass/Warn/Fail rating

### Step 4: Tag Hierarchy Audit

Walk through questions T1-T8. For each:
1. Map all tags used in operations to top-level definitions
2. Check classification metadata
3. Evaluate hierarchy structure
4. Assign Pass/Warn/Fail rating

### Step 5: Component Reuse Audit

Walk through questions R1-R10. For each:
1. Identify inline vs referenced schemas
2. Validate all $ref resolutions
3. Check composition patterns (allOf/oneOf/discriminator)
4. Verify error response reuse
5. Assign Pass/Warn/Fail rating

### Step 6: Advanced Operations Audit

Walk through questions A1-A10. For each:
1. Check for webhook definitions and payload structure
2. Evaluate streaming endpoint specifications
3. Review link objects and callback definitions
4. Assign Pass/Warn/Fail rating

Note: Questions A1-A10 are conditional -- only audit categories present in the spec. If no webhooks exist, mark webhook questions as N/A (do not count in scoring).

### Step 7: Extension Governance Audit

Walk through questions E1-E8. For each:
1. Collect all x- prefixed fields across the spec
2. Cross-reference with the extension registry
3. Check for superseded extensions
4. Verify spec validity without extensions
5. Assign Pass/Warn/Fail rating

### Step 8: Spec Structure Audit

Walk through questions F1-F8. For each:
1. Assess multi-file vs monolithic layout
2. Check root file conventions
3. Verify $ref strategy consistency
4. Check bundling configuration
5. Assign Pass/Warn/Fail rating

### Step 9: Authoring Model Audit

Walk through questions W1-W6. For each:
1. Check for authoring model declaration
2. Verify source-of-truth policy exists
3. Assess authority flow boundaries
4. Evaluate dual-schema maintenance tax
5. Assign Pass/Warn/Fail rating

### Step 10: Change Workflow Audit

Walk through questions G1-G8. For each:
1. Check Golden Path sequence compliance
2. Review PR conventions for spec changes
3. Evaluate cross-team coordination patterns
4. Assess backward compatibility practices
5. Assign Pass/Warn/Fail rating

### Step 11: Severity Prioritization

1. Collect all Fail and Warn findings
2. Assign severity level from the question bank
3. Sort findings by severity (Critical first)
4. Group by category for the final report

---

## CI Governance Tools

### Spectral (OpenAPI Linting)

| Aspect | Details |
|--------|---------|
| What it checks | Schema validity, naming conventions, description completeness |
| Integration | CLI, GitHub Actions, pre-commit hook |
| Custom rules | YAML-based ruleset files (`.spectral.yaml`) |
| Best for | M1-M8, O1-O9, T1-T5, R1-R3 |

**Recommended custom rules:**

```yaml
rules:
  # operationId must be camelCase
  operation-id-casing:
    description: operationId must be camelCase
    severity: error
    given: "$.paths.*.*.operationId"
    then:
      function: casing
      functionOptions:
        type: camel

  # Schema keys must be UpperCamelCase
  schema-key-casing:
    description: Schema component keys must be UpperCamelCase
    severity: error
    given: "$.components.schemas.*~"
    then:
      function: pattern
      functionOptions:
        match: "^[A-Z][a-zA-Z0-9]+$"

  # No inline request bodies
  no-inline-request-body:
    description: Request bodies must use $ref
    severity: warn
    given: "$.paths.*.*.requestBody.content.*.schema"
    then:
      field: "$ref"
      function: truthy

  # All tags must have descriptions
  tag-description:
    description: Every tag must have a description
    severity: warn
    given: "$.tags[*]"
    then:
      field: description
      function: truthy
```

### Redocly CLI

| Aspect | Details |
|--------|---------|
| What it checks | Spec validity, bundling, custom rules, preview |
| Integration | CLI, GitHub Actions, VS Code extension |
| Custom rules | Plugin-based rules in `.redocly.yaml` |
| Best for | Multi-file spec bundling, $ref validation, preview rendering |

### Vacuum

| Aspect | Details |
|--------|---------|
| What it checks | Full Spectral compatibility with better performance |
| Integration | CLI, Go library |
| Advantage | 10x faster than Spectral on large specs |
| Best for | CI pipelines with large specifications |

### FAIL/WARN Severity Rubric for CI

| Finding | Severity | CI Action |
|---------|----------|-----------|
| Missing operationId | FAIL | Block merge |
| Duplicate operationId | FAIL | Block merge |
| Broken $ref reference | FAIL | Block merge |
| Inline request/response body | WARN | Require acknowledgment |
| Missing tag description | WARN | Require acknowledgment |
| oneOf without discriminator | FAIL | Block merge |
| Non-camelCase operationId | WARN | Require acknowledgment |
| Missing Info fields | WARN | Require acknowledgment |

---

## Output Format

Structure every audit report as follows:

```
# OpenAPI Specification Compliance Audit

**Date**: YYYY-MM-DD
**Scope**: [spec files audited]
**Spec Version**: [3.0.x / 3.1.0 / 3.2.0]
**Operations**: [count]
**Schemas**: [count]
**Tags**: [count]

---

## Executive Summary

[2-3 sentences: overall compliance level, most critical finding, recommendation priority]

---

## Compliance Scorecard

| Category | Pass | Warn | Fail | N/A | Score |
|----------|------|------|------|-----|-------|
| Spec Metadata (M1-M11) | X | Y | Z | - | NN% |
| Operation Identifiers (O1-O10) | X | Y | Z | - | NN% |
| Tag Hierarchy (T1-T8) | X | Y | Z | - | NN% |
| Component Reuse (R1-R10) | X | Y | Z | - | NN% |
| Advanced Operations (A1-A10) | X | Y | Z | W | NN% |
| Extension Governance (E1-E8) | X | Y | Z | - | NN% |
| Spec Structure (F1-F8) | X | Y | Z | - | NN% |
| Authoring Model (W1-W6) | X | Y | Z | - | NN% |
| Change Workflow (G1-G8) | X | Y | Z | - | NN% |
| **Overall** | | | | | **NN%** |

---

## Findings by Severity

### Critical
| # | Question | Category | Location | Finding | Recommendation |
|---|----------|----------|----------|---------|----------------|

### Major
| # | Question | Category | Location | Finding | Recommendation |
|---|----------|----------|----------|---------|----------------|

### Minor
| # | Question | Category | Location | Finding | Recommendation |
|---|----------|----------|----------|---------|----------------|

### Cosmetic
| # | Question | Category | Location | Finding | Recommendation |
|---|----------|----------|----------|---------|----------------|

---

## Detailed Results by Category

### Spec Metadata
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Operation Identifiers
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Tag Hierarchy
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Component Reuse
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Advanced Operations
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Extension Governance
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Spec Structure
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Authoring Model
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Change Workflow
| # | Question | Rating | Notes |
|---|----------|--------|-------|

---

## CI Governance Recommendations

[Which tools to adopt, in what order, with configuration priorities]

---

## Improvement Roadmap

### Immediate (Before Next Release)
- [Critical findings]

### Short-Term (Current Sprint)
- [Major findings]

### Medium-Term (Current Quarter)
- [Minor findings]

### Backlog
- [Cosmetic findings]
```

---

## Guardrails

1. **Never modify files** -- you have Read, Grep, and Glob only
2. **Never produce inline code fixes** -- describe what should change, not the code
3. **Always walk through all questions across all 9 categories** -- report on every question even if it passes (mark N/A for inapplicable)
4. **Always classify severity** -- every finding gets a severity level from the question bank
5. **Always produce the full output format** -- partial reports are not acceptable
6. **Delegate fixes to spec_designer** -- end the report by recommending which findings to hand off
