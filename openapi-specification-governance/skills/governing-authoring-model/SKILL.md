---
name: governing-authoring-model
version: 1.0.0
user-invocable: false
description: Guide evaluation, selection, and governance of API specification authoring models including design-first, code-first, and hybrid approaches, unidirectional authority flow from spec to implementation, dual-schema maintenance tax assessment, source-of-truth governance policy formalization, and criteria for choosing the right model for a given team and API. Use when evaluating authoring models, formalizing source-of-truth policy, auditing authority flow, or assessing dual-schema maintenance tax. Triggered by: authoring model, design-first, code-first, hybrid, source of truth, authority flow, spec-first, contract-first, dual schema, maintenance tax, governance model, API workflow, spec authority.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Authoring Model Governance

Comprehensive governance for selecting and enforcing the API specification authoring model -- design-first, code-first, or hybrid -- and the authority flow that determines which artifact is the source of truth.

## Purpose

Every API project must answer a foundational question: does the spec drive the code, or does the code drive the spec? The answer determines tool selection, CI pipeline design, team workflows, and the cost of keeping spec and implementation in sync. This skill provides evaluation criteria for each authoring model, formalizes the source-of-truth governance policy, and quantifies the maintenance tax of dual-schema systems.

**Boundary:** This skill governs the authoring posture and authority flow. For the specific workflow steps that implement the chosen model, see `governing-change-workflow`. For how the spec files are organized, see `governing-spec-structure`.

Use this skill when evaluating authoring models, formalizing source-of-truth policy, or auditing authority flow between spec and implementation.

---

## Authoring Models

### Design-First (Spec → Code)

The OpenAPI specification is written before any implementation code. The spec is the single source of truth.

```
Write spec  →  Generate server stubs  →  Implement logic  →  Validate conformance
```

| Aspect | Details |
|--------|---------|
| Source of truth | OpenAPI spec |
| Authority flow | Spec → Implementation |
| Spec author | API designers, architects |
| Implementation constraint | Must conform to spec |
| Tooling | Spec editors, server generators, conformance testers |

**Strengths:**
- Contract is reviewed before implementation effort begins
- Frontend and backend can develop in parallel against the same spec
- Breaking changes are caught at the spec level before code exists
- SDK generation produces types that are correct by construction

**Weaknesses:**
- Requires spec authoring expertise on the team
- Spec and implementation can drift if conformance testing is not automated
- Server stub generators may not match the team's framework preferences

### Code-First (Code → Spec)

Implementation code is written first. The spec is generated from code annotations, decorators, or runtime introspection.

```
Write code  →  Generate spec from code  →  Validate spec quality  →  Publish spec
```

| Aspect | Details |
|--------|---------|
| Source of truth | Implementation code |
| Authority flow | Implementation → Spec |
| Spec author | Framework (FastAPI, Spring, etc.) |
| Implementation constraint | None (code is authoritative) |
| Tooling | Framework auto-generation, spec validators |

**Strengths:**
- Developers work in their native language and framework
- Spec is always in sync with implementation (generated from running code)
- Lower barrier to entry (no spec authoring skills needed)
- FastAPI's Pydantic-to-OpenAPI generation is particularly strong

**Weaknesses:**
- Spec quality depends on code annotation quality
- Generated specs often have naming inconsistencies (framework defaults)
- Breaking changes are only detected after implementation, not before
- SDK generation consumes whatever the framework produces, including quirks

### Hybrid (Spec ↔ Code with Authority Rules)

Both spec and code are authored, with explicit rules governing which artifact has authority over which aspect.

```
Write spec (contract)  →  Implement code  →  Validate spec ↔ code conformance  →  Reconcile
```

| Aspect | Details |
|--------|---------|
| Source of truth | Spec for contract shape; code for implementation behavior |
| Authority flow | Bidirectional with governance rules |
| Spec author | API designers for contract; developers for implementation |
| Implementation constraint | Must conform to spec contract shape |
| Tooling | Spec editors, conformance testers, bidirectional validators |

**Strengths:**
- Combines design review rigor with developer ergonomics
- Allows spec-first for external APIs and code-first for internal utilities
- Supports gradual migration from code-first to design-first
- Practical for teams with mixed spec authoring expertise

**Weaknesses:**
- Requires clear authority rules to prevent confusion
- Dual maintenance if authority boundaries are unclear
- More complex CI pipeline (must validate in both directions)

---

## Authority Flow

### Unidirectional Authority (Recommended)

In a well-governed system, authority flows in one direction:

```
Spec (authoritative)  →  Implementation (conforming)  →  Generated artifacts (derived)
```

| Layer | Role | Can Be Changed By |
|-------|------|-------------------|
| Spec | Defines the contract | API designers via spec PR |
| Implementation | Implements the contract | Developers, constrained by spec |
| Generated artifacts | Derived from spec | Tooling only (never hand-edited) |

### Bidirectional Authority (Use With Caution)

When authority flows in both directions, explicit rules prevent conflict:

| Aspect | Authority | Example |
|--------|-----------|---------|
| Endpoint paths and methods | Spec | `/v1/projects` defined in spec |
| Request/response schemas | Spec | `ProjectCreate` schema defined in spec |
| Status codes and error shapes | Spec | `422 → ProblemDetail` defined in spec |
| Validation rules beyond schema | Code | `name` must not contain special characters |
| Business logic behavior | Code | Project creation triggers notification |
| Performance characteristics | Code | Pagination defaults, rate limits |

### Authority Flow Rules

| Rule | Rationale |
|------|-----------|
| Contract shape (paths, schemas, responses) is always spec-authoritative | Consumers depend on the contract, not the implementation |
| Behavioral details (validation rules, side effects) are always code-authoritative | These cannot be expressed in OpenAPI |
| When in doubt, the spec wins | Resolves ambiguity in favor of the consumer |
| Authority boundaries must be documented | Prevents teams from making ad-hoc decisions |

---

## Source-of-Truth Governance Policy

### Policy Template

Every API project should formalize its source-of-truth policy:

```markdown
## API Contract Governance Policy

### Authoring Model
[Design-First / Code-First / Hybrid]

### Source of Truth
- **Contract shape** (paths, schemas, responses): [Spec / Code]
- **Behavioral details** (validation, side effects): [Code]
- **Generated artifacts** (SDK, docs): [Derived from spec]

### Authority Flow
[Spec → Code / Code → Spec / Bidirectional with rules]

### Change Process
1. [Changes start in: spec / code]
2. [Conformance verified by: tool / manual review]
3. [Breaking changes require: deprecation window / approval / both]

### Enforcement
- CI gate: [tool and configuration]
- Conformance test: [tool and configuration]
- Breaking change detection: [tool and configuration]
```

### Policy Rules

| Rule | Rationale |
|------|-----------|
| Policy must be committed to the repository | Not in a wiki, not in Slack, not in someone's head |
| Policy must specify the authoring model explicitly | Prevents implicit assumptions |
| Policy must specify authority for each aspect | Resolves disputes before they happen |
| Policy must specify the change process | Prevents ad-hoc workflow variations |
| Policy must be referenced in the contract package README | Discoverable by anyone working on the API |

---

## Dual-Schema Maintenance Tax

### What Is the Tax?

When both the spec and the implementation define the same schema, every change must be made twice:

```
Spec: ProjectCreate schema  ←→  Code: ProjectCreate Pydantic model
```

### Tax Assessment

| Factor | Low Tax | High Tax |
|--------|---------|----------|
| Number of schemas | < 10 | > 50 |
| Change frequency | Monthly | Daily |
| Team size | 1-2 developers | 5+ developers |
| Schema complexity | Flat objects | Nested, composed, polymorphic |
| Drift detection | Automated (Schemathesis) | Manual review |

### Tax Reduction Strategies

| Strategy | Mechanism | Trade-off |
|----------|-----------|-----------|
| Generate code from spec | Server stubs from spec | Generated code may not match framework idioms |
| Generate spec from code | FastAPI auto-generation | Spec quality depends on annotation quality |
| Conformance testing | Schemathesis validates spec ↔ code | Tax remains but drift is caught |
| Schema sharing | Shared JSON Schema between spec and code | Requires tooling support in both ecosystems |
| Single authoring point | One artifact is authoritative, other is derived | Requires choosing which side to author |

### Tax Rules

| Rule | Rationale |
|------|-----------|
| Dual maintenance is acceptable only with automated drift detection | Manual sync is unsustainable |
| If tax exceeds team capacity, switch to single-source authoring | Drifted schemas are worse than no schema |
| Measure tax by counting dual-edit PRs per sprint | Quantifies the actual burden |
| Review authoring model quarterly | Team size, API complexity, and consumer count change over time |

---

## Model Selection Criteria

### Decision Matrix

| Factor | Favors Design-First | Favors Code-First | Favors Hybrid |
|--------|--------------------|--------------------|---------------|
| External consumers | Yes | - | - |
| Internal-only API | - | Yes | - |
| Strong spec expertise on team | Yes | - | - |
| No spec expertise | - | Yes | - |
| Frontend/backend parallel dev | Yes | - | Yes |
| Rapid prototyping phase | - | Yes | - |
| Contractual SLA on compatibility | Yes | - | - |
| FastAPI with mature Pydantic models | - | Yes | Yes |
| Multiple consumer platforms | Yes | - | - |
| Gradual migration from code-first | - | - | Yes |

### Selection Rules

| Rule | Rationale |
|------|-----------|
| Default to design-first for new APIs with external consumers | Consumer trust depends on contract stability |
| Default to code-first for internal prototypes | Speed matters more than contract rigor in early stages |
| Use hybrid when migrating from code-first | Allows gradual adoption of spec-first practices |
| Re-evaluate when the consumer count changes | An internal API that gains external consumers needs stricter governance |
| Document the choice and the rationale | Prevents re-litigation of decided questions |

---

## Review Dimensions

When reviewing or auditing the authoring model, evaluate these dimensions:

### 1. Model Clarity
- Is the authoring model explicitly declared?
- Do all team members know which artifact is the source of truth?
- Is the choice documented in the repository?

### 2. Authority Flow
- Is authority flow unidirectional or bidirectional?
- If bidirectional, are authority boundaries explicit?
- Are there conflicts where both spec and code claim authority?

### 3. Maintenance Tax
- How many dual-edit PRs occur per sprint?
- Is drift detection automated?
- Is the tax sustainable for the current team size?

### 4. Model Fit
- Does the model match the team's expertise?
- Does the model match the consumer profile?
- Has the model been reviewed since the last major team or consumer change?

---

## Anti-Patterns

### 1. Implicit Authoring Model
No declared model -- some developers treat the spec as authoritative, others treat the code as authoritative. Drift is inevitable.

### 2. Design-First Without Conformance Testing
Writing the spec first but never verifying that the implementation matches. The spec becomes aspirational documentation.

### 3. Code-First With Manual Spec Editing
Generating the spec from code but then hand-editing the generated output. Creates a merge conflict on every regeneration.

### 4. Hybrid Without Authority Rules
Using a hybrid model without specifying which aspects are spec-authoritative and which are code-authoritative. Resolving conflicts becomes political.

### 5. Never Re-evaluating
Choosing a model once and never revisiting, even as the team grows from 2 to 20 or the API gains external consumers.

---

## Guardrails

1. **Every API project must declare its authoring model** -- implicit models cause drift
2. **Source-of-truth policy must be committed to the repository** -- not in wikis or chat
3. **Authority flow must be unidirectional or explicitly bounded** -- ambiguous authority causes conflicts
4. **Dual-schema maintenance requires automated drift detection** -- manual sync is unsustainable
5. **Authoring model must be re-evaluated when team or consumer profile changes** -- review at least quarterly
6. **The chosen model must be enforceable by CI** -- a model that cannot be automated is a suggestion, not a policy
