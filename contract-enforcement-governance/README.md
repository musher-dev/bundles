# Contract Enforcement Governance

Catch breaking changes, enforce conformance tests, and gate CI before bad contracts ship. This bundle governs the enforcement layer between your OpenAPI spec and your implementation — breaking change detection with oasdiff, conformance testing with Schemathesis, SDK generation with @hey-api/openapi-ts, and multi-stage CI pipelines.

## Quick Start

Install via the [Mush CLI](https://github.com/musher-dev/mush):

```sh
mush bundle install musher-dev/contract-enforcement-governance
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Audit our contract enforcement posture — check for breaking change detection, conformance tests, SDK generation, and CI gates
```

## What's Inside

The bundle ships two agents and six skills that cover the full contract enforcement pipeline.

The **contract_auditor** agent performs a read-only assessment of your enforcement posture across five dimensions: contract structure, breaking change detection, conformance testing, SDK generation, and CI pipeline completeness. It produces a severity-prioritized findings report with a maturity score and improvement roadmap.

The **contract_engineer** agent implements the infrastructure that the auditor recommends — oasdiff configuration for breaking change classification, Schemathesis test scaffolding with `from_asgi()` for FastAPI, `@hey-api/openapi-ts` setup for SDK generation, CI pipeline files with blocking and warning gates, and monorepo contract package layout.

The enforcement skills are organized around a 5-stage pipeline model: lint, bundle, diff, backend validate, frontend conform. Each stage can run as a blocking gate or advisory warning, with artifact passing between stages and observability hooks.

## Usage Examples

**Assess enforcement maturity**
```
Audit our contract enforcement setup — how mature is our breaking change detection, conformance testing, and CI pipeline?
```
The contract_auditor evaluates your posture and produces a maturity score with prioritized next steps.

**Set up breaking change detection**
```
Configure oasdiff to detect breaking changes in our OpenAPI spec and block PRs that introduce ERR-level breaks
```
The contract_engineer sets up oasdiff with severity classification (ERR vs WARN), deprecation window enforcement, and CI gate configuration.

**Scaffold conformance tests**
```
Set up Schemathesis conformance tests against our FastAPI app with stateful testing and test database management
```
Creates property-based contract tests that validate your implementation matches the spec, with stateful testing via OpenAPI links.

## Boundaries

This bundle governs the **enforcement pipeline** — the tooling and CI configuration that verifies your implementation matches your spec.

For **OpenAPI specification document** standards (metadata, operationIds, tags, component reuse, multi-file structure), see [openapi-specification-governance](../openapi-specification-governance/).

For **route and payload implementation** concerns (URI naming, HTTP semantics, error responses, Pydantic models), see [api-route-governance](../api-route-governance/).

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `governing-contract-structure` | Monorepo layout, artifact separation (editable spec vs generated outputs), workspace config |
| `detecting-breaking-changes` | oasdiff integration, ERR/WARN classification, deprecation windows, sunset headers, CI gates |
| `testing-contract-conformance` | Schemathesis setup, `from_asgi()` integration, stateful testing, test DB management |
| `governing-sdk-generation` | `@hey-api/openapi-ts` config, `tsc --noEmit` validation, consumption rules, regeneration triggers |
| `governing-enforcement-pipeline` | 5-stage pipeline (lint → bundle → diff → validate → conform), gate modes, artifact passing |
| `auditing-contract-enforcement` | 5-category question bank, severity model, compliance scorecard, maturity assessment |

### Agents

| Agent | Role |
|-------|------|
| `contract_auditor` | Read-only enforcement posture audits with maturity scoring and improvement roadmap |
| `contract_engineer` | Implements enforcement infrastructure — oasdiff, Schemathesis, SDK generation, CI pipelines |

</details>

---

**Domain:** contract-enforcement · **Technologies:** oasdiff, Schemathesis, @hey-api/openapi-ts (harness-agnostic) · **Aliases:** contract-gov, enforcement-governance, contract-pipeline
