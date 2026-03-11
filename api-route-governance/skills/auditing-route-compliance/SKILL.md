---
name: auditing-route-compliance
version: 1.0.0
user-invocable: false
description: Guide structured audit execution for API route compliance using a 4-category question bank covering Structural Integrity, Protocol Semantics, Contract Definition, and Documentation, with Pass/Warn/Fail severity model mapped to Critical/Major/Minor/Cosmetic levels, and CI tool recommendations including Spectral, Optic, and Schemathesis. Use when conducting API audits, reviewing route compliance, prioritizing API technical debt, or setting up CI governance tooling. Triggered by: route audit, API audit, compliance audit, route compliance, audit rubric, Spectral, Optic, Schemathesis, CI governance, API linting, spec linting, audit question bank, severity model, API quality gate.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Route Compliance Auditing

Structured audit execution framework for API route compliance, providing a question bank, severity model, and CI tooling recommendations.

## Purpose

Ad-hoc API reviews produce inconsistent results -- one reviewer checks naming, another checks status codes, a third skips both and focuses on auth. This skill provides a deterministic audit framework: a structured question bank that covers every governance dimension, a severity model that prioritizes findings, and CI tool recommendations that automate enforcement. The result is reproducible, comprehensive audits that produce actionable, severity-prioritized reports.

Use this skill when conducting API route audits, reviewing compliance against governance standards, prioritizing API technical debt, or setting up automated CI governance.

---

## Audit Question Bank

### Category 1: Structural Integrity

Questions about URI design, resource modeling, and route architecture.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| S1 | Are all collection resources named with plural nouns? | governing-route-naming | Minor |
| S2 | Are all path segments kebab-case? | governing-route-naming | Major |
| S3 | Does any URI exceed 3 levels of resource nesting? | governing-route-naming | Major |
| S4 | Do custom operations use the `:verb` syntax (AIP-136)? | governing-route-naming | Minor |
| S5 | Are sequential integer IDs exposed in any URI? | governing-route-naming | Critical |
| S6 | Is the `/me` alias available for user-scoped resources? | governing-route-naming | Minor |
| S7 | Do tenant-scoped endpoints include `{org_slug}` in the path? | governing-tenant-routes | Critical |
| S8 | Are global and tenant-scoped endpoints clearly separated? | governing-tenant-routes | Major |
| S9 | Is nesting depth consistent across similar resource types? | governing-route-naming | Cosmetic |
| S10 | Do slug-capable resources accept both slug and UUID? | governing-route-naming | Minor |

### Category 2: Protocol Semantics

Questions about HTTP method usage, status codes, and RFC compliance.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| P1 | Do any GET or HEAD endpoints produce side effects? | governing-http-semantics | Critical |
| P2 | Is PUT used for full replacement (not partial updates)? | governing-http-semantics | Major |
| P3 | Does DELETE handle already-deleted resources gracefully? | governing-http-semantics | Minor |
| P4 | Are 201 and 202 distinguished correctly? | governing-http-semantics | Major |
| P5 | Are 401 and 403 used for correct scenarios? | governing-http-semantics | Major |
| P6 | Are 400 and 422 distinguished (syntax vs semantic errors)? | governing-http-semantics | Minor |
| P7 | Do 429 responses include `Retry-After` header? | governing-http-semantics | Minor |
| P8 | Do endpoints validate request `Content-Type`? | governing-http-semantics | Major |
| P9 | Is URI-based versioning used with `/api/v{N}/` prefix? | governing-api-versioning | Major |
| P10 | Do deprecated endpoints include `Deprecation` and `Sunset` headers? | governing-api-versioning | Major |

### Category 3: Contract Definition

Questions about response shapes, error formats, pagination, and OpenAPI compliance.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| C1 | Do error responses use `application/problem+json`? | governing-error-responses | Major |
| C2 | Do all error responses include `type`, `title`, `status`, `detail`, `instance`? | governing-error-responses | Major |
| C3 | Do 422 responses include an `errors` array with field-level details? | governing-error-responses | Major |
| C4 | Do collection responses use the `data`/`meta`/`links` envelope? | governing-collection-endpoints | Major |
| C5 | Is cursor-based pagination used (not offset)? | governing-collection-endpoints | Major |
| C6 | Is `page_size` bounded with min/max/default? | governing-collection-endpoints | Minor |
| C7 | Are filter parameters restricted to indexed columns? | governing-collection-endpoints | Major |
| C8 | Does every endpoint have a declared `response_model`? | governing-openapi-contracts | Major |
| C9 | Are error responses declared in the `responses` dict? | governing-openapi-contracts | Minor |
| C10 | Do operation IDs follow the `verb_resource` convention? | governing-openapi-contracts | Minor |

### Category 4: Documentation

Questions about spec completeness, metadata, and discoverability.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| D1 | Is the OpenAPI version set to 3.1.0? | governing-openapi-contracts | Minor |
| D2 | Are all required metadata fields populated (title, summary, version)? | governing-openapi-contracts | Minor |
| D3 | Do endpoints have meaningful summaries and descriptions? | governing-openapi-contracts | Minor |
| D4 | Is `\f` used to exclude internal notes from the spec? | governing-openapi-contracts | Cosmetic |
| D5 | Are tags applied consistently with metadata defined? | governing-openapi-contracts | Cosmetic |
| D6 | Are query parameters (filters, sort, pagination) snake_case? | governing-route-naming | Minor |
| D7 | Are JSON response properties snake_case? | governing-route-naming | Major |
| D8 | Is the gateway dual-validation middleware documented? | governing-tenant-routes | Minor |
| D9 | Are breaking changes classified and tracked? | governing-api-versioning | Major |
| D10 | Is a deprecation timeline published for deprecated endpoints? | governing-api-versioning | Minor |

---

## Severity Model

### Severity Levels

| Level | Definition | Timeframe | Examples |
|-------|------------|-----------|---------|
| **Critical** | Security vulnerability, data exposure risk, or fundamental architectural flaw that cannot be fixed without breaking changes | Fix before next release | Sequential IDs exposed, GET with side effects, tenant data accessible without org_slug |
| **Major** | Standards violation that degrades developer experience, causes integration failures, or harms API reliability | Fix within current sprint | Wrong status codes, missing error envelope, offset pagination, inconsistent casing |
| **Minor** | Convention deviation that reduces consistency but does not cause failures | Fix within current quarter | Missing `/me` alias, undeclared error responses, absent `Retry-After` header |
| **Cosmetic** | Style preference or optimization opportunity with minimal functional impact | Add to backlog | Tag metadata ordering, nesting depth inconsistency, `\f` truncation missing |

### Pass/Warn/Fail Rating

Each audit question receives a rating:

| Rating | Criteria |
|--------|----------|
| **Pass** | Fully compliant with the governance standard |
| **Warn** | Partially compliant or compliant with caveats |
| **Fail** | Non-compliant with the governance standard |

### Overall Compliance Score

Calculate the compliance percentage per category and overall:

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

Locate all API-defining files in the codebase:
- Glob for route files (`**/routes/**`, `**/routers/**`, `**/endpoints/**`, `**/api/**`)
- Glob for OpenAPI specs (`**/openapi.json`, `**/openapi.yaml`, `**/swagger.*`)
- Glob for error handling (`**/exceptions/**`, `**/errors/**`, `**/handlers/**`)
- Glob for middleware (`**/middleware/**`, `**/deps/**`, `**/dependencies/**`)
- Catalog every endpoint: method, path, handler function, response model

### Step 2: Structural Integrity Audit

Walk through questions S1-S10. For each:
1. Search the codebase for evidence of compliance or violation
2. Record the finding with file path and line number
3. Assign Pass/Warn/Fail rating

### Step 3: Protocol Semantics Audit

Walk through questions P1-P10. For each:
1. Examine route decorators for method selection and status codes
2. Check error handlers for correct status code usage
3. Verify header handling (Content-Type, Deprecation, Sunset)
4. Assign Pass/Warn/Fail rating

### Step 4: Contract Definition Audit

Walk through questions C1-C10. For each:
1. Examine response models and error response structures
2. Check pagination implementation (cursor vs offset)
3. Verify OpenAPI configuration and response declarations
4. Assign Pass/Warn/Fail rating

### Step 5: Documentation Audit

Walk through questions D1-D10. For each:
1. Check OpenAPI metadata configuration
2. Review endpoint docstrings and summaries
3. Verify naming consistency across query params and response properties
4. Assign Pass/Warn/Fail rating

### Step 6: Severity Prioritization

1. Collect all Fail and Warn findings
2. Assign severity level (Critical/Major/Minor/Cosmetic) from the question bank
3. Sort findings by severity (Critical first)
4. Group by category for the final report

---

## CI Governance Tools

### Spectral (OpenAPI Linting)

**Purpose:** Lint OpenAPI specs against configurable rulesets.

| Aspect | Details |
|--------|---------|
| What it checks | Schema validity, naming conventions, description completeness, response codes |
| Integration | CLI, GitHub Actions, pre-commit hook |
| Custom rules | YAML-based ruleset files (`.spectral.yaml`) |
| Best for | Questions D1-D5, C8-C10, S1-S2 |

**Recommended rules:**
- `operation-operationId`: Every operation has an operationId
- `operation-description`: Every operation has a description
- `oas3-valid-media-example`: Examples match their schemas
- Custom: snake_case property names, kebab-case paths

### Optic (API Diff and Change Detection)

**Purpose:** Detect breaking changes between API spec versions.

| Aspect | Details |
|--------|---------|
| What it checks | Field removals, type changes, required field additions, endpoint removals |
| Integration | CLI, CI pipeline, GitHub PR comments |
| Custom rules | Policy-as-code for allowed/forbidden changes |
| Best for | Questions P9-P10, D9-D10 |

**Recommended configuration:**
- Block: response field removal, type change, required field addition
- Warn: new required query parameter, enum value removal
- Allow: new optional field, new endpoint, response field addition

### Schemathesis (Property-Based API Testing)

**Purpose:** Generate test cases from OpenAPI specs to find runtime compliance issues.

| Aspect | Details |
|--------|---------|
| What it checks | Status code correctness, response schema compliance, error format compliance |
| Integration | CLI, pytest plugin, CI pipeline |
| Test generation | Automatic from OpenAPI spec -- no hand-written tests needed |
| Best for | Questions P1-P8, C1-C3 |

**Recommended usage:**
- Run against staging environment after deployment
- Validates that actual responses match declared schemas
- Catches status code mismatches and undeclared error formats

### Tool Selection Matrix

| Audit Category | Spectral | Optic | Schemathesis |
|---------------|----------|-------|-------------|
| Structural Integrity | Partial (naming) | No | No |
| Protocol Semantics | No | Partial (changes) | Yes (runtime) |
| Contract Definition | Partial (schema) | No | Yes (runtime) |
| Documentation | Yes (completeness) | No | No |

---

## Output Format

Structure every audit report as follows:

```
# API Route Compliance Audit

**Date**: YYYY-MM-DD
**Scope**: [files/directories audited]
**Framework**: [detected framework]
**Endpoints Audited**: [count]

---

## Executive Summary

[2-3 sentences: overall compliance level, most critical finding, recommendation priority]

---

## Compliance Scorecard

| Category | Pass | Warn | Fail | Score |
|----------|------|------|------|-------|
| Structural Integrity (S1-S10) | X | Y | Z | NN% |
| Protocol Semantics (P1-P10) | X | Y | Z | NN% |
| Contract Definition (C1-C10) | X | Y | Z | NN% |
| Documentation (D1-D10) | X | Y | Z | NN% |
| **Overall** | | | | **NN%** |

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

### Structural Integrity
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Protocol Semantics
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Contract Definition
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Documentation
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
3. **Always walk through all 40 questions** -- report on every question even if it passes
4. **Always classify severity** -- every finding gets a severity level from the question bank
5. **Always produce the full output format** -- partial reports are not acceptable
6. **Delegate fixes to route_designer** -- end the report by recommending which findings to hand off
