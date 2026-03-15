---
name: governing-change-workflow
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of API specification change workflows including the Golden Path sequence (design, generate, implement, verify, deploy), cross-team coordination patterns for spec changes, spec-change PR conventions and review requirements, backward-compatible change techniques, and versioning strategies for spec evolution. Use when designing spec change workflows, auditing change processes, reviewing PR conventions, or implementing backward-compatible changes. Triggered by: change workflow, Golden Path, spec change, PR convention, backward compatible, spec evolution, change process, cross-team coordination, spec PR, API change, change management, spec versioning, workflow governance.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Change Workflow Governance

Comprehensive governance for how API specification changes flow through the team -- from design through deployment, with coordination patterns and backward compatibility techniques.

## Purpose

Spec changes are not code changes. A spec change is a contract commitment that affects every consumer, every SDK, and every integration test. Treating spec changes like ordinary code -- push, review, merge -- creates breaks that only surface when consumers update. This skill defines the Golden Path workflow for spec changes, coordination patterns for multi-team APIs, PR conventions that make spec changes reviewable, and backward-compatible change techniques that preserve consumer trust.

**Boundary:** This skill governs the procedural workflow for spec changes. For the authoring model that determines who has authority, see `governing-authoring-model`. For breaking change detection tooling, see `contract-enforcement-governance/detecting-breaking-changes`.

Use this skill when designing spec change workflows, auditing change processes, or reviewing PR conventions for spec modifications.

---

## The Golden Path

### 5-Step Sequence

Every spec change follows this sequence, regardless of authoring model:

```
1. DESIGN     →  2. GENERATE    →  3. IMPLEMENT   →  4. VERIFY      →  5. DEPLOY
Draft spec       Bundle spec       Write/update      Run enforcement   Publish spec
change in PR     + generate SDK    implementation    pipeline          + deploy impl
```

### Step Details

| Step | Actor | Artifact | Gate |
|------|-------|----------|------|
| 1. Design | API designer / developer | Spec diff in PR | Human review of contract change |
| 2. Generate | CI pipeline | Bundled spec + SDK types | Bundle succeeds, no unresolved refs |
| 3. Implement | Backend + frontend developers | Route handlers + UI code | Unit tests pass |
| 4. Verify | CI pipeline | Enforcement report | All 5 pipeline stages pass |
| 5. Deploy | Release process | Production API + published spec | Smoke tests against production |

### Step Dependencies

```
DESIGN ──→ GENERATE ──→ IMPLEMENT ──→ VERIFY ──→ DEPLOY
  ↑                                      │
  └──── Fix findings and return ─────────┘
```

### Workflow Rules

| Rule | Rationale |
|------|-----------|
| Spec change must be designed before implementation begins | Prevents building against a moving contract |
| Generation happens automatically after spec change | Ensures SDK types reflect the latest spec |
| Implementation starts only after generation succeeds | Developers work against accurate types |
| Verification runs the full enforcement pipeline | Catches breaks, conformance failures, type errors |
| Verification failures loop back to design, not forward to deploy | Never deploy a failing contract |

---

## Spec-Change PR Conventions

### PR Structure

Spec changes should be structured for reviewability:

```markdown
## Spec Change: [Brief description]

### Contract Impact
- [ ] New endpoint(s): [list]
- [ ] Modified endpoint(s): [list]
- [ ] New schema(s): [list]
- [ ] Modified schema(s): [list]
- [ ] Breaking change: [yes/no, with details]
- [ ] Deprecation: [yes/no, with sunset date]

### Motivation
[Why this contract change is needed]

### Consumer Impact
[How this affects SDK users, frontend, integrations]

### Backward Compatibility
[How backward compatibility is preserved, or why a break is necessary]
```

### PR Rules

| Rule | Rationale |
|------|-----------|
| Spec changes must be in their own PR or clearly separated from implementation | Spec review requires different expertise than code review |
| PR description must include contract impact checklist | Reviewers need to know what changed at a glance |
| Breaking changes must be called out explicitly | Prevents accidental approval of breaks |
| Consumer impact must be described | Forces the author to think about downstream effects |
| Spec-only PRs are valid (implementation follows in a separate PR) | Allows spec review before implementation effort |

### Review Requirements

| Change Type | Minimum Reviewers | Required Expertise |
|-------------|-------------------|-------------------|
| New endpoint | 1 | API design |
| Schema modification | 1 | API design |
| Breaking change | 2 | API design + consumer representative |
| Deprecation | 1 | API design |
| New API version | 2 | API design + tech lead |

---

## Cross-Team Coordination

### When Coordination Is Needed

| Trigger | Coordination Required |
|---------|----------------------|
| Any breaking change | Consumer teams must acknowledge and plan migration |
| New endpoint that changes domain boundaries | Adjacent API teams must review for overlap |
| Schema shared across APIs | All consuming API teams must review |
| Deprecation with sunset date | Consumer teams must schedule migration before sunset |
| New API version | All consumers must be notified of migration timeline |

### Coordination Patterns

#### Pattern 1: Spec-Change RFC

For significant changes, publish a lightweight RFC before writing the spec diff:

```markdown
## API Change RFC: [Title]

**Author**: [name]
**Date**: [date]
**Status**: [Draft / Review / Approved / Rejected]

### Problem
[What consumer need drives this change]

### Proposed Change
[High-level description of the contract change]

### Alternatives Considered
[Other approaches and why they were rejected]

### Consumer Impact
[Which teams/services are affected and how]

### Migration Path
[How consumers will adopt the change]

### Timeline
[When the change will be available and when old behavior will be removed]
```

#### Pattern 2: Spec-Change Notification

For routine changes, notify affected consumers via standardized channels:

| Change Type | Notification Channel | Timing |
|-------------|---------------------|--------|
| New endpoint | Changelog + SDK release notes | On merge |
| Non-breaking schema change | Changelog | On merge |
| Deprecation | Direct notification to consumers | Before merge |
| Breaking change | Direct notification + migration guide | 2+ sprints before merge |
| New API version | All-hands announcement | Quarter before release |

#### Pattern 3: Consumer Opt-In

For changes that add new capabilities without breaking existing behavior:

1. Merge the spec change
2. Publish updated SDK
3. Consumers adopt the new capability at their own pace
4. No coordination overhead

### Coordination Rules

| Rule | Rationale |
|------|-----------|
| Breaking changes require consumer acknowledgment before merge | Prevents surprise breaks |
| Deprecations require direct notification, not just changelog | Consumers must actively plan migration |
| RFCs are required for changes affecting 3+ consumer teams | Coordination overhead scales with consumer count |
| Routine non-breaking changes do not require pre-merge coordination | Low-risk changes should not be bottlenecked |

---

## Backward-Compatible Change Techniques

### Adding New Capabilities

| Technique | Example | Consumer Impact |
|-----------|---------|-----------------|
| Add optional field to request | New `description` field on `ProjectCreate` | None -- old clients omit it |
| Add field to response | New `created_by` on `ProjectResponse` | None -- clients ignore unknown fields |
| Add new endpoint | `GET /v1/analytics` | None -- clients do not call it yet |
| Add enum value | New `archived` status | Minimal -- clients should handle unknown enums |
| Add optional query parameter | `?include=members` | None -- old queries still work |

### Evolving Existing Capabilities

| Technique | Example | Consumer Impact |
|-----------|---------|-----------------|
| Loosen validation | `maxLength: 100` → `maxLength: 255` | None -- previously valid input still valid |
| Make required field optional | `region` becomes optional with default | None -- clients still send it |
| Widen response type | `string` → `string \| null` | Minimal -- clients should handle nullability |
| Add response header | New `X-Request-Id` header | None -- clients ignore unknown headers |

### Deprecation Workflow

```
Sprint N:     Add deprecated: true to field/endpoint
Sprint N:     Add x-sunset date
Sprint N:     Add Sunset response header
Sprint N+1:   Notify consumers directly
Sprint N+M:   Remove (M ≥ 2 minor versions or sunset date reached)
```

### Techniques to Avoid

| Technique | Problem |
|-----------|---------|
| Removing a field without deprecation | Breaks consumers immediately |
| Renaming a field | Equivalent to remove + add; breaks consumers |
| Changing a field type | Breaks client deserialization |
| Making an optional field required | Breaks clients that do not send it |
| Narrowing an enum | Breaks clients that send removed values |

---

## Versioning Strategies

### When to Version

| Change | Requires New Version? | Approach |
|--------|----------------------|----------|
| New endpoint | No | Add to current version |
| New optional field | No | Add to current version |
| Non-breaking schema change | No | Increment `info.version` patch |
| Deprecation | No | Increment `info.version` minor |
| Breaking change (cannot be avoided) | Yes | New major version |

### Version Coexistence

When a breaking change requires a new major version:

```
/v1/projects  →  Maintained (bug fixes only, sunset date set)
/v2/projects  →  Active development
```

| Rule | Rationale |
|------|-----------|
| Maximum 2 concurrent major versions | More than 2 creates unsustainable maintenance burden |
| Old version gets bug fixes only, no features | Incentivizes migration |
| Sunset date set on old version at new version launch | Clear migration timeline |
| Old version removed after sunset date | Reduces maintenance scope |

---

## Review Dimensions

When reviewing or auditing change workflows, evaluate these dimensions:

### 1. Workflow Completeness
- Does every spec change follow the Golden Path?
- Are there changes that skip directly from design to deploy?
- Is verification (step 4) consistently executed?

### 2. PR Quality
- Do spec-change PRs include the contract impact checklist?
- Are breaking changes called out explicitly?
- Is consumer impact described?

### 3. Coordination Effectiveness
- Are breaking changes coordinated with consumer teams before merge?
- Are deprecations directly notified to affected consumers?
- Are RFCs used for significant changes?

### 4. Compatibility Practices
- Are backward-compatible techniques used by default?
- Is the deprecation workflow followed before removal?
- Are breaking changes a last resort, not a convenience?

---

## Anti-Patterns

### 1. Spec Change as Afterthought
Implementation is complete before the spec is updated. The spec becomes documentation of what was built, not a design artifact.

### 2. Breaking Change Without Coordination
Merging a breaking spec change without notifying consumer teams. Consumers discover the break when their CI fails or production breaks.

### 3. Perpetual Deprecation
Marking fields as deprecated but never removing them. Deprecated fields accumulate, confusing consumers about what is actually supported.

### 4. Skip-to-Deploy
Merging a spec change and deploying without running the enforcement pipeline. Verification exists to catch problems before production.

### 5. Monolith PR
Combining spec changes, implementation, frontend updates, and infrastructure changes in a single PR. Reviewers cannot isolate the contract impact.

---

## Guardrails

1. **Every spec change must follow the Golden Path sequence** -- design, generate, implement, verify, deploy
2. **Spec-change PRs must include the contract impact checklist** -- reviewers need structured context
3. **Breaking changes require consumer acknowledgment before merge** -- not just reviewer approval
4. **Deprecation must precede removal by at least 2 minor versions** -- consumers need migration time
5. **Verification failures must loop back to design, not forward to deploy** -- never deploy a failing contract
6. **Cross-team coordination scales with consumer count** -- 1 consumer = notification, 3+ consumers = RFC
