---
name: auditing-contract-enforcement
version: 1.0.0
user-invocable: false
description: Guide structured audit execution for contract enforcement compliance using a 5-category question bank covering Contract Structure, Breaking Change Detection, Conformance Testing, SDK Generation, and Enforcement Pipeline, with Pass/Warn/Fail severity model mapped to Critical/Major/Minor/Cosmetic levels, compliance scorecard, enforcement maturity assessment, and improvement roadmap. Use when conducting contract enforcement audits, reviewing enforcement posture, auditing pipeline completeness, assessing enforcement maturity, or prioritizing enforcement technical debt. Triggered by: contract audit, enforcement audit, pipeline audit, contract compliance, enforcement maturity, enforcement posture, contract quality gate, enforcement scorecard, enforcement review.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Contract Enforcement Auditing

Structured audit execution framework for contract enforcement compliance, providing a question bank, severity model, maturity assessment, and improvement roadmap.

## Purpose

Ad-hoc enforcement reviews produce inconsistent results -- one reviewer checks the CI pipeline, another checks SDK generation, a third skips both and focuses on breaking changes. This skill provides a deterministic audit framework: a structured question bank that covers every enforcement dimension, a severity model that prioritizes findings, and a maturity assessment that benchmarks the organization's enforcement posture. The result is reproducible, comprehensive audits that produce actionable, severity-prioritized reports.

Use this skill when conducting contract enforcement audits, assessing enforcement maturity, or prioritizing enforcement technical debt.

---

## Audit Question Bank

### Category 1: Contract Structure

Questions about monorepo layout, artifact separation, workspace configuration, and dependency direction.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| S1 | Are editable spec files and generated artifacts in separate directories? | governing-contract-structure | Major |
| S2 | Is there a dedicated contract package (`packages/contract/` or equivalent)? | governing-contract-structure | Major |
| S3 | Does the contract package have zero dependencies on application packages? | governing-contract-structure | Critical |
| S4 | Are workspace build tasks declared with correct dependency ordering? | governing-contract-structure | Major |
| S5 | Are task inputs and outputs declared for caching? | governing-contract-structure | Minor |
| S6 | Does the frontend import from generated SDK, not from raw spec files? | governing-contract-structure | Major |
| S7 | Is there a `clean` script that removes only generated artifacts? | governing-contract-structure | Minor |
| S8 | Are generated files committed to version control? | governing-contract-structure | Minor |

### Category 2: Breaking Change Detection

Questions about automated diffing, backward compatibility, deprecation enforcement, and CI gates.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| B1 | Is automated breaking change detection (oasdiff or equivalent) configured? | detecting-breaking-changes | Critical |
| B2 | Does the detection run on every PR that modifies spec files? | detecting-breaking-changes | Critical |
| B3 | Is the base spec sourced from main branch or production? | detecting-breaking-changes | Major |
| B4 | Do ERR-level breaking changes block merge? | detecting-breaking-changes | Major |
| B5 | Do WARN-level findings require human review? | detecting-breaking-changes | Minor |
| B6 | Is there a deprecation-before-removal policy? | detecting-breaking-changes | Major |
| B7 | Are deprecation windows enforced via tooling (not just policy)? | detecting-breaking-changes | Minor |
| B8 | Is there an escalation path for intentional breaking changes? | detecting-breaking-changes | Minor |

### Category 3: Conformance Testing

Questions about spec-to-implementation testing, stateful coverage, database management, and validation completeness.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| C1 | Is automated conformance testing (Schemathesis or equivalent) configured? | testing-contract-conformance | Critical |
| C2 | Do conformance tests run against the published OpenAPI spec? | testing-contract-conformance | Major |
| C3 | Is `from_asgi()` used for FastAPI integration (no server lifecycle)? | testing-contract-conformance | Minor |
| C4 | Is stateful testing enabled when OpenAPI links are defined? | testing-contract-conformance | Major |
| C5 | Does the test database use the same engine as production? | testing-contract-conformance | Major |
| C6 | Is transaction rollback used for test isolation? | testing-contract-conformance | Minor |
| C7 | Are all documented response codes triggerable in tests? | testing-contract-conformance | Minor |
| C8 | Do conformance test failures block merge? | testing-contract-conformance | Major |

### Category 4: SDK Generation

Questions about code generation configuration, manual wrapper elimination, type conformance, and consumption hygiene.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| G1 | Is an SDK code generator (openapi-ts or equivalent) configured? | governing-sdk-generation | Critical |
| G2 | Does the generator consume the bundled spec (not raw multi-file)? | governing-sdk-generation | Major |
| G3 | Are there manual fetch/axios wrappers for spec-covered endpoints? | governing-sdk-generation | Major |
| G4 | Does `tsc --noEmit` pass after SDK regeneration? | governing-sdk-generation | Critical |
| G5 | Are there `@ts-ignore` or `as any` on SDK imports? | governing-sdk-generation | Major |
| G6 | Is the generator version pinned (not using `latest` or `^`)? | governing-sdk-generation | Minor |
| G7 | Is SDK regeneration triggered by spec changes in CI? | governing-sdk-generation | Major |
| G8 | Do consumers import directly from the generated SDK? | governing-sdk-generation | Major |

### Category 5: Enforcement Pipeline

Questions about pipeline stage completeness, dependency ordering, gate configuration, and observability.

| # | Question | Governance Source | Severity if Failed |
|---|----------|-------------------|-------------------|
| P1 | Are all 5 pipeline stages present (lint, bundle, diff, backend validate, frontend conform)? | governing-enforcement-pipeline | Critical |
| P2 | Does the lint stage run before bundle? | governing-enforcement-pipeline | Major |
| P3 | Does the bundle stage produce a shared artifact for downstream stages? | governing-enforcement-pipeline | Major |
| P4 | Do backend validate and frontend conform run in parallel? | governing-enforcement-pipeline | Minor |
| P5 | Is the diff stage comparing against the main branch spec? | governing-enforcement-pipeline | Major |
| P6 | Does the gate mode match the API's consumer profile? | governing-enforcement-pipeline | Minor |
| P7 | Are pipeline results posted to the PR? | governing-enforcement-pipeline | Minor |
| P8 | Does the pipeline trigger on spec, backend, and frontend changes? | governing-enforcement-pipeline | Major |

---

## Severity Model

### Severity Levels

| Level | Definition | Timeframe | Examples |
|-------|------------|-----------|---------|
| **Critical** | No enforcement exists for a key dimension, or enforcement is misconfigured to the point of being bypassed | Fix before next release | No breaking change detection, no conformance testing, circular contract dependencies |
| **Major** | Enforcement exists but has significant gaps that reduce its effectiveness | Fix within current sprint | Wrong base spec for diffing, manual wrappers alongside SDK, no merge blocking |
| **Minor** | Enforcement works but has optimization or completeness gaps | Fix within current quarter | Missing caching config, sequential stages that could be parallel, missing PR comments |
| **Cosmetic** | Style or process preference with minimal enforcement impact | Add to backlog | Generator version not pinned, missing clean script |

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
| 75-89% | Good | Some enforcement gaps to address |
| 50-74% | Needs Work | Significant enforcement gaps |
| Below 50% | Critical | Fundamental enforcement failures |

---

## Enforcement Maturity Assessment

### Maturity Levels

| Level | Name | Characteristics |
|-------|------|-----------------|
| **1** | Ad Hoc | No automated enforcement; contract compliance is manual and inconsistent |
| **2** | Spec Linting | Automated spec validation (lint stage) but no diffing, testing, or SDK enforcement |
| **3** | Breaking Change Detection | Spec linting + automated breaking change detection; no conformance testing |
| **4** | Full Verification | Linting + diffing + conformance testing + SDK generation; all stages present |
| **5** | Optimized | Full verification + observability metrics + quarterly gate mode reviews + documented escalation paths |

### Maturity Mapping

| Audit Score | Likely Maturity Level |
|------------|----------------------|
| Below 30% | Level 1: Ad Hoc |
| 30-50% | Level 2: Spec Linting |
| 50-70% | Level 3: Breaking Change Detection |
| 70-90% | Level 4: Full Verification |
| 90-100% | Level 5: Optimized |

---

## Audit Process

### Step 1: Discovery

Locate all contract enforcement artifacts in the codebase:
- Glob for contract package (`**/packages/contract/`, `**/contract/`)
- Glob for spec files (`**/openapi.yaml`, `**/openapi.json`)
- Glob for generated artifacts (`**/generated/`, `**/sdk/`)
- Glob for CI config (`**/.github/workflows/*.yml`, `**/Jenkinsfile`, `**/.gitlab-ci.yml`)
- Glob for tool config (`**/.spectral.yaml`, `**/redocly.yaml`, `**/openapi-ts.config.*`)
- Glob for conformance tests (`**/test_contract*`, `**/test_conformance*`)
- Identify workspace tool (Turborepo, Nx, npm workspaces)

### Step 2: Contract Structure Audit

Walk through questions S1-S8. For each:
1. Examine directory layout and artifact separation
2. Check workspace configuration and dependency ordering
3. Verify import paths in consuming packages
4. Assign Pass/Warn/Fail rating

### Step 3: Breaking Change Detection Audit

Walk through questions B1-B8. For each:
1. Check for oasdiff or equivalent configuration
2. Verify CI trigger conditions
3. Check base spec source and gate behavior
4. Review deprecation policy and escalation paths
5. Assign Pass/Warn/Fail rating

### Step 4: Conformance Testing Audit

Walk through questions C1-C8. For each:
1. Check for Schemathesis or equivalent configuration
2. Verify spec source and integration method
3. Check database configuration and isolation
4. Review test coverage and gate behavior
5. Assign Pass/Warn/Fail rating

### Step 5: SDK Generation Audit

Walk through questions G1-G8. For each:
1. Check for generator configuration
2. Search for manual fetch/axios wrappers
3. Verify type conformance practices
4. Check consumption patterns and import paths
5. Assign Pass/Warn/Fail rating

### Step 6: Enforcement Pipeline Audit

Walk through questions P1-P8. For each:
1. Check CI pipeline for all 5 stages
2. Verify stage ordering and dependencies
3. Check gate modes and PR feedback
4. Review trigger conditions
5. Assign Pass/Warn/Fail rating

### Step 7: Severity Prioritization

1. Collect all Fail and Warn findings
2. Assign severity level from the question bank
3. Sort findings by severity (Critical first)
4. Group by category for the final report
5. Assign maturity level based on overall score

---

## Output Format

Structure every audit report as follows:

```
# Contract Enforcement Compliance Audit

**Date**: YYYY-MM-DD
**Scope**: [contract package and CI pipeline audited]
**Workspace Tool**: [Turborepo / Nx / npm workspaces / other]
**Enforcement Maturity**: Level N - [Name]

---

## Executive Summary

[2-3 sentences: overall enforcement posture, most critical gap, recommendation priority]

---

## Compliance Scorecard

| Category | Pass | Warn | Fail | Score |
|----------|------|------|------|-------|
| Contract Structure (S1-S8) | X | Y | Z | NN% |
| Breaking Change Detection (B1-B8) | X | Y | Z | NN% |
| Conformance Testing (C1-C8) | X | Y | Z | NN% |
| SDK Generation (G1-G8) | X | Y | Z | NN% |
| Enforcement Pipeline (P1-P8) | X | Y | Z | NN% |
| **Overall** | | | | **NN%** |

---

## Findings by Severity

### Critical
| # | Question | Category | Finding | Recommendation |
|---|----------|----------|---------|----------------|

### Major
| # | Question | Category | Finding | Recommendation |
|---|----------|----------|---------|----------------|

### Minor
| # | Question | Category | Finding | Recommendation |
|---|----------|----------|---------|----------------|

### Cosmetic
| # | Question | Category | Finding | Recommendation |
|---|----------|----------|---------|----------------|

---

## Detailed Results by Category

### Contract Structure
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Breaking Change Detection
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Conformance Testing
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### SDK Generation
| # | Question | Rating | Notes |
|---|----------|--------|-------|

### Enforcement Pipeline
| # | Question | Rating | Notes |
|---|----------|--------|-------|

---

## Enforcement Maturity Assessment

**Current Level**: [1-5] - [Name]
**Target Level**: [1-5] - [Name]
**Gap Summary**: [What is needed to reach target level]

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
3. **Always walk through all questions** -- report on every question even if it passes
4. **Always classify severity** -- every finding gets a severity level from the question bank
5. **Always produce the full output format** -- partial reports are not acceptable
6. **Delegate fixes to contract_engineer** -- end the report by recommending which findings to hand off
