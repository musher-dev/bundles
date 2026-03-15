---
name: governing-enforcement-pipeline
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of contract enforcement CI pipelines covering the 5-stage pipeline (lint, bundle, diff, backend validate, frontend conform), gate configuration with blocking and warning modes, stage dependency ordering, parallel vs sequential execution, artifact passing between stages, and pipeline observability. Use when designing contract enforcement CI, auditing pipeline completeness, reviewing gate modes, or configuring enforcement stages. Triggered by: enforcement pipeline, CI pipeline, contract CI, lint stage, bundle stage, diff stage, validate stage, conform stage, gate config, blocking mode, warning mode, pipeline stages, CI gates, contract enforcement.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Enforcement Pipeline Governance

Comprehensive governance for the CI/CD pipeline that enforces contract compliance across the full development lifecycle.

## Purpose

Individual enforcement tools (linting, diffing, testing) provide value in isolation, but their real power emerges when orchestrated into a pipeline with defined stage ordering, artifact flow, and gate behavior. A linter that catches spec errors is useless if it runs after SDK generation has already produced invalid types. A breaking change detector is meaningless if it compares against the wrong base. This skill defines the canonical 5-stage enforcement pipeline, its stage dependencies, gate configuration, and observability requirements.

**Boundary:** This skill orchestrates the enforcement stages defined by sibling skills. For the content of each stage, see `detecting-breaking-changes`, `testing-contract-conformance`, `governing-sdk-generation`, and `governing-contract-structure`.

Use this skill when designing or auditing the CI pipeline that enforces contract compliance.

---

## The 5-Stage Pipeline

### Stage Overview

```
Stage 1       Stage 2       Stage 3        Stage 4            Stage 5
 LINT    →    BUNDLE    →    DIFF     →   BACKEND         →  FRONTEND
                                          VALIDATE            CONFORM
                                         (parallel)          (parallel)
```

| Stage | Input | Output | Tool | Purpose |
|-------|-------|--------|------|---------|
| 1. Lint | `openapi/**/*.yaml` | Pass/Fail | Spectral, Redocly, Vacuum | Validate spec syntax, naming, structure |
| 2. Bundle | `openapi/**/*.yaml` | `openapi.bundled.json` | Redocly CLI | Resolve `$ref`, produce single file |
| 3. Diff | `openapi.bundled.json` + base spec | Breaking change report | oasdiff | Detect breaking changes vs main branch |
| 4. Backend Validate | `openapi.bundled.json` + backend source | Conformance report | Schemathesis | Verify implementation matches spec |
| 5. Frontend Conform | `openapi.bundled.json` + frontend source | Type check report | openapi-ts + tsc | Regenerate SDK, verify type compatibility |

### Stage Dependencies

```
lint ──→ bundle ──→ diff ──┐
                           ├──→ (done)
         bundle ──→ backend-validate ──┘
         bundle ──→ frontend-conform ──┘
```

| Dependency Rule | Rationale |
|-----------------|-----------|
| Bundle depends on Lint passing | No point bundling an invalid spec |
| Diff depends on Bundle output | Diffing requires a resolved single-file spec |
| Backend Validate depends on Bundle output | Schemathesis needs the bundled spec |
| Frontend Conform depends on Bundle output | SDK generation needs the bundled spec |
| Backend Validate and Frontend Conform are independent | Can run in parallel after Bundle |
| Diff and Backend/Frontend stages are independent | Can run in parallel after Bundle |

---

## Stage 1: Lint

### Purpose
Validate that the OpenAPI specification is syntactically valid, follows naming conventions, and meets structural requirements before any downstream processing.

### Configuration

```yaml
# .spectral.yaml
extends:
  - spectral:oas

rules:
  operation-operationId: error
  operation-operationId-unique: error
  oas3-schema: error
  no-$ref-siblings: warn
  operation-tag-defined: error

  # Custom rules from governing-spec-metadata, governing-operation-identifiers
  operation-id-casing:
    severity: error
    given: "$.paths.*.*.operationId"
    then:
      function: casing
      functionOptions:
        type: camel
```

### Gate Behavior

| Finding Severity | Blocking Mode | Warning Mode |
|-----------------|---------------|--------------|
| Error | Block pipeline | Block pipeline |
| Warning | Block pipeline | Log and continue |
| Info | Log and continue | Log and continue |

---

## Stage 2: Bundle

### Purpose
Resolve all `$ref` pointers across multi-file spec sources into a single `openapi.bundled.json` file that downstream stages consume.

### Configuration

```yaml
# redocly.yaml
apis:
  main:
    root: openapi/openapi.yaml
    output: generated/openapi.bundled.json

rules:
  no-unresolved-refs: error
  no-unused-components: warn
```

### Execution

```bash
redocly bundle openapi/openapi.yaml -o generated/openapi.bundled.json
```

### Bundle Rules

| Rule | Rationale |
|------|-----------|
| Bundle must resolve all `$ref` pointers | Unresolved refs break downstream tools |
| Output must be deterministic | Same input produces same output for caching |
| Unused components should be flagged | Dead schemas add noise |
| Bundle output is an artifact, not a source file | Never hand-edit the bundled output |

---

## Stage 3: Diff

### Purpose
Compare the bundled spec against the base branch (main) to detect breaking changes before merge.

### Configuration

```bash
# Get base spec from main branch
git show origin/main:packages/contract/generated/openapi.bundled.json > /tmp/base-spec.json

# Run diff
oasdiff breaking /tmp/base-spec.json generated/openapi.bundled.json --fail-on ERR
```

### Gate Behavior

| Change Level | Strict Mode | Standard Mode | Advisory Mode |
|-------------|-------------|---------------|---------------|
| ERR (breaking) | Block merge | Block merge | Comment on PR |
| WARN (potentially breaking) | Block merge | Comment on PR | Log only |

### Diff Rules

| Rule | Rationale |
|------|-----------|
| Base spec must be from main branch or production | Comparing against wrong base produces false results |
| ERR findings block merge in all non-advisory modes | Breaking changes must be intentional |
| WARN findings require human review | Potentially breaking changes need judgment |
| Diff report must be posted as PR comment | Visibility for reviewers |

---

## Stage 4: Backend Validate

### Purpose
Verify that the backend implementation conforms to the bundled spec by running Schemathesis against the application.

### Configuration

```python
# tests/test_contract.py
import schemathesis
from myapp.main import app

schema = schemathesis.from_asgi("/openapi.json", app=app)

@schema.parametrize()
def test_contract_conformance(case):
    response = case.call_asgi()
    case.validate_response(response)
```

### CI Execution

```yaml
- name: Run conformance tests
  run: pytest tests/test_contract.py -v --tb=short
```

### Backend Validate Rules

| Rule | Rationale |
|------|-----------|
| Tests run against the bundled spec, not a copy | Single source of truth |
| `from_asgi()` for FastAPI (no server required) | Simplifies CI execution |
| Test database uses same engine as production | Prevents engine-specific divergence |
| Failures block merge | Non-conforming implementation is a contract violation |

---

## Stage 5: Frontend Conform

### Purpose
Regenerate the SDK from the bundled spec and verify that the frontend codebase type-checks against the new types.

### Execution

```bash
# Regenerate SDK
npx openapi-ts

# Type check
npx tsc --noEmit

# Run frontend tests
npm run test
```

### Frontend Conform Rules

| Rule | Rationale |
|------|-----------|
| SDK must be regenerated from the bundled spec | Ensures types match the current contract |
| `tsc --noEmit` must pass | Type errors indicate contract drift |
| No `@ts-ignore` on SDK imports | Suppressed errors hide real contract issues |
| Frontend tests must pass | Behavioral regressions from type changes |

---

## Pipeline Configuration

### GitHub Actions Example

```yaml
name: Contract Enforcement

on:
  pull_request:
    paths:
      - "packages/contract/**"
      - "backend/src/**"
      - "frontend/src/**"

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx @stoplight/spectral-cli lint packages/contract/openapi/openapi.yaml

  bundle:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npx @redocly/cli bundle packages/contract/openapi/openapi.yaml -o packages/contract/generated/openapi.bundled.json
      - uses: actions/upload-artifact@v4
        with:
          name: bundled-spec
          path: packages/contract/generated/openapi.bundled.json

  diff:
    needs: bundle
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/download-artifact@v4
        with:
          name: bundled-spec
      - run: git show origin/main:packages/contract/generated/openapi.bundled.json > /tmp/base.json
      - uses: tufin/oasdiff-action/breaking@main
        with:
          base: /tmp/base.json
          revision: openapi.bundled.json
          fail-on: ERR

  backend-validate:
    needs: bundle
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with:
          name: bundled-spec
      - run: pip install -r requirements.txt
      - run: pytest tests/test_contract.py

  frontend-conform:
    needs: bundle
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/download-artifact@v4
        with:
          name: bundled-spec
      - run: npm ci
      - run: npx openapi-ts
      - run: npx tsc --noEmit
```

### Gate Modes

| Mode | When to Use | ERR | WARN |
|------|------------|-----|------|
| **Strict** | Public APIs, external consumers | Block all stages | Block all stages |
| **Standard** | Internal APIs, coordinated teams | Block merge | Require review |
| **Advisory** | Pre-launch, prototyping phase | PR comment | Log only |

### Choosing a Gate Mode

| Signal | Points Toward |
|--------|---------------|
| External consumers who cannot coordinate upgrades | Strict |
| Internal consumers with shared deploy cadence | Standard |
| No consumers yet (pre-launch) | Advisory |
| Contractual SLA on backward compatibility | Strict |
| Rapid iteration phase with breaking changes expected | Advisory |

---

## Pipeline Observability

### Required Metrics

| Metric | Purpose | Alert Threshold |
|--------|---------|-----------------|
| Pipeline pass rate | Overall enforcement health | < 80% over 7 days |
| Stage failure distribution | Identify weakest enforcement area | Any stage > 50% failure rate |
| Breaking change frequency | Contract stability | > 2 ERR findings per week |
| Pipeline duration | Developer experience | > 10 minutes total |
| False positive rate | Gate credibility | > 10% of blocked PRs overridden |

### PR Comment Format

```markdown
## Contract Enforcement Report

| Stage | Status | Duration | Findings |
|-------|--------|----------|----------|
| Lint | Pass | 12s | 0 errors, 2 warnings |
| Bundle | Pass | 3s | - |
| Diff | Pass | 5s | 0 ERR, 1 WARN |
| Backend Validate | Pass | 45s | 23 tests passed |
| Frontend Conform | Pass | 18s | 0 type errors |

**Overall: PASS**
```

---

## Review Dimensions

When reviewing or auditing the enforcement pipeline, evaluate these dimensions:

### 1. Stage Completeness
- Are all 5 stages present?
- Are stage dependencies correct (lint before bundle, bundle before everything else)?
- Are stages 4 and 5 running in parallel?

### 2. Gate Configuration
- Does the gate mode match the API's consumer profile?
- Are ERR findings blocking merge?
- Is there an escalation path for intentional breaks?

### 3. Artifact Flow
- Does the bundled spec flow from stage 2 to stages 3, 4, and 5?
- Is the base spec for diffing sourced from the correct reference?
- Are artifacts preserved for debugging?

### 4. Pipeline Triggers
- Does the pipeline trigger on spec file changes?
- Does the pipeline trigger on backend source changes?
- Does the pipeline trigger on frontend source changes?

---

## Anti-Patterns

### 1. Lint-Only Pipeline
Running only spec linting without breaking change detection, conformance testing, or SDK conformance. Catches syntax errors but misses contract drift.

### 2. Sequential Backend/Frontend Stages
Running backend validate and frontend conform sequentially when they have no dependency on each other. Doubles pipeline duration unnecessarily.

### 3. Missing Artifact Passing
Each stage fetching and bundling the spec independently instead of sharing the bundle artifact. Wastes time and risks inconsistency.

### 4. Wrong Base for Diff
Comparing the current spec against a hardcoded version instead of the main branch. Produces stale diff results.

### 5. No PR Feedback
Pipeline runs but posts no results to the PR. Developers must navigate to CI logs to understand failures.

---

## Guardrails

1. **All 5 stages must be present in the pipeline** -- partial pipelines leave enforcement gaps
2. **Stage dependency ordering must be enforced** -- lint before bundle, bundle before all downstream stages
3. **Backend validate and frontend conform must run in parallel** -- sequential execution wastes CI time
4. **Bundled spec must be shared as an artifact between stages** -- prevents redundant bundling
5. **Pipeline results must be posted to the PR** -- visibility for reviewers without navigating CI logs
6. **Gate mode must be documented and reviewed quarterly** -- ensure mode matches current consumer profile
