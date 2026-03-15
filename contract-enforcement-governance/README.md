# Contract Enforcement Governance

Audits, designs, and enforces contract verification across the development pipeline — breaking change detection, conformance testing, SDK generation, CI gates, and monorepo artifact structure.

## Metadata

- **Domain:** contract-enforcement
- **Risk Level:** medium
- **Technologies:** None (vendor-agnostic patterns; skills reference specific tools as recommended implementations)
- **Aliases:** contract-gov, enforcement-governance, contract-pipeline

## Boundary

This bundle governs the enforcement layer between specification authoring and route implementation — detecting breaks, testing conformance, generating SDKs, configuring CI gates, and structuring monorepo artifacts. For the OpenAPI specification document itself (metadata, operations, tags, components, extensions), see `openapi-specification-governance`. For framework-specific route implementation (naming, HTTP semantics, payloads, schemas), see `api-route-governance`.

## Components

### Skills

| Name | Scope |
|------|-------|
| `governing-contract-structure` | Monorepo workspace config (Turborepo/Nx), `packages/contract/` layout, artifact separation (editable vs generated vs implementation source) |
| `detecting-breaking-changes` | oasdiff integration, breaking change classification (ERR/WARN), backward compatibility rules, deprecation window enforcement, CI gate config |
| `testing-contract-conformance` | Schemathesis integration, `from_asgi()` for FastAPI, stateful testing with OpenAPI links, test DB management, state machine config |
| `governing-sdk-generation` | @hey-api/openapi-ts config, prohibiting manual fetch wrappers, `tsc --noEmit` conformance, generated SDK consumption rules, regeneration triggers |
| `governing-enforcement-pipeline` | 5-stage pipeline (lint, bundle, diff, backend validate, frontend conform), gate config, blocking vs warning modes |
| `auditing-contract-enforcement` | Audit question bank across all 5 enforcement dimensions, compliance scoring, enforcement maturity assessment |

### Agents

| Name | Purpose | Model | Tools |
|------|---------|-------|-------|
| `contract_auditor` | Read-only audits of contract enforcement posture across break detection, test coverage, SDK generation, CI gates, repo structure | opus | Read, Grep, Glob |
| `contract_engineer` | Implements enforcement infrastructure: oasdiff config, Schemathesis scaffolding, SDK generation config, CI pipeline files, monorepo layout | sonnet | Read, Grep, Glob, Edit, Write |

### Tools
- None
