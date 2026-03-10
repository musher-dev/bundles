---
name: shaping-reviewing-codebase-fit
version: 1.0.0
user-invocable: false
description: Produce a codebase fit review that compares an Architecture Sketch's assumptions against the actual codebase, verifying component existence, naming conventions, pattern consistency, complexity, dependency impact, and integration points. Use when an Architecture Sketch needs reality-checking against the real codebase before implementation commits. Triggered by: codebase fit review, codebase reality check, architecture assumptions, naming conventions, complexity assessment, existing patterns, codebase alignment.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Bash
---

# Codebase Fit Review

Expert guidance for producing codebase fit review artifacts during the shaping process.

## Purpose

A codebase fit review verifies whether an Architecture Sketch's assumptions about the existing codebase are accurate. Sketches are written during shaping when the author may be working from memory, documentation, or incomplete knowledge. This review compares assumptions against reality by searching the actual code.

Use this skill after an Architecture Sketch has been written. The reviewing agent actively searches the codebase using Grep, Glob, and Bash to verify that assumed components exist, naming conventions match, patterns are consistent, and integration points work as described. This is the only reviewing skill that reads the codebase directly.

---

## Output Format

```markdown
# Codebase Fit Review

**Artifact Reviewed:** <Name of the Architecture Sketch>
**Date:** <YYYY-MM-DD>
**Reviewer Role:** Codebase Reality Checker

---

## Dimension Ratings

| # | Dimension | Rating | Notes |
|---|-----------|--------|-------|
| 1 | Component Existence | Pass / Partial / Fail | <Brief justification> |
| 2 | Naming Alignment | Pass / Partial / Fail | <Brief justification> |
| 3 | Pattern Consistency | Pass / Partial / Fail | <Brief justification> |
| 4 | Complexity Assessment | Pass / Partial / Fail | <Brief justification> |
| 5 | Dependency Impact | Pass / Partial / Fail | <Brief justification> |
| 6 | Integration Points | Pass / Partial / Fail | <Brief justification> |

---

## Assumption Verification

| # | Assumption from Sketch | Verified? | Actual State | Impact if Wrong |
|---|----------------------|-----------|-------------|-----------------|
| A1 | <What the sketch assumes> | Yes / No / Partial | <What actually exists> | <Consequence of the mismatch> |
...

---

## Naming Comparison

| Proposed Name | Codebase Convention | Suggested Name | Rationale |
|--------------|-------------------|---------------|-----------|
| <Name from sketch> | <Pattern found in codebase> | <Recommended name> | <Why the change is needed> |
...

---

## Affected Files Estimate

| Area | Files Affected | Change Type | Complexity |
|------|---------------|-------------|------------|
| <Module/directory> | <Count or range> | New / Modified / Deleted | Low / Medium / High |
...

---

## Dimension-Specific Analysis

### Component Existence
<Summary of which assumed components exist, are missing, or have moved.>

### Pattern Consistency
<Summary of whether proposed approach follows existing patterns or introduces new ones.>

### Dependency Impact
<Summary of shared modules affected and their consumers.>

---

## Findings

### Critical

| # | Finding | Dimension | Impact | Recommendation |
|---|---------|-----------|--------|----------------|
| 1 | ... | ... | ... | ... |

### Major

| # | Finding | Dimension | Impact | Recommendation |
|---|---------|-----------|--------|----------------|
| 1 | ... | ... | ... | ... |

### Minor

| # | Finding | Dimension | Impact | Recommendation |
|---|---------|-----------|--------|----------------|
| 1 | ... | ... | ... | ... |

---

## Conditions for Approval

- [ ] All "existing" components referenced in the sketch have been located in the codebase
- [ ] Proposed names align with codebase naming conventions (or deviations are justified)
- [ ] The proposed approach follows established patterns (or new patterns are acknowledged)
- [ ] The scope of affected files is realistic for the defined appetite
- [ ] Integration points exist and behave as the sketch assumes
- [ ] No assumption marked "Verified: No" remains unaddressed

---

## Verdict

**<Fit / Fit with Gaps / Misaligned>**

<1-3 sentence justification.>
```

---

## Instructions

1. **Extract every assumption from the Architecture Sketch.** List each component referenced as "existing," each naming choice, each integration point, and each claim about how the codebase works. Include implicit assumptions (e.g., "we will add a handler to the existing router" assumes the router exists and accepts new handlers).

2. **Search the codebase to verify each assumption.** Use Glob to check file/directory existence. Use Grep to find naming patterns, function signatures, class definitions, and imports. Use Bash for structural queries (directory listings, line counts, dependency checks). Record whether each assumption is verified (Yes), refuted (No), or partially correct (Partial).

3. **Compare naming conventions.** Collect proposed names and compare against codebase patterns. Check file naming (kebab-case, snake_case, PascalCase), module structure (flat vs nested), variable naming, and domain terminology. Populate the Naming Comparison table for mismatches.

4. **Assess complexity and scope.** Estimate files to change, create, or delete per area. Categorize by complexity (Low: straightforward additions; Medium: modifications to existing logic; High: refactoring shared infrastructure). Compare total scope against the appetite.

5. **Compile findings and render the verdict.** Use fit-specific labels: **Fit** (all assumptions verified, naming aligns, patterns consistent), **Fit with Gaps** (some assumptions wrong but adjustments are minor), **Misaligned** (fundamental assumptions incorrect, requiring significant sketch revision).

---

## Quality Checklist

- [ ] Every component referenced as "existing" has been searched for in the codebase
- [ ] Assumption Verification includes both explicit and implicit assumptions
- [ ] Naming Comparison shows actual codebase conventions with evidence (file paths, existing names)
- [ ] Affected Files Estimate is based on codebase search results, not guesswork
- [ ] Pattern Consistency analysis references specific existing patterns found in code
- [ ] Dependency Impact identifies concrete consumers of shared modules affected
- [ ] Verdict uses correct labels (Fit / Fit with Gaps / Misaligned)

---

## Anti-Patterns

### 1. Armchair Verification
Rating assumptions as "Verified: Yes" based on memory without actually searching the codebase. This skill exists because assumptions drift from reality. Every assumption must be verified by searching the code.

### 2. Surface-Level Name Checking
Verifying a file exists by name without checking its contents. A file named `UserService` might be deprecated, empty, or have a different interface than assumed. Verify both existence and shape.

### 3. Ignoring Implicit Assumptions
Only verifying explicitly named components while ignoring implicit assumptions like "the database supports this query pattern" or "the build system includes this directory."

### 4. Scope Underestimation
Reporting only directly mentioned files as "affected" without considering ripple effects: test files, type definitions, barrel exports, configuration files, and downstream consumers of changed interfaces.

### 5. Pattern Conflict Dismissal
Noting a proposed approach differs from existing patterns but dismissing it as "fine because the new way is better." Pattern inconsistency has a real cost in cognitive load and maintenance. If a new pattern is introduced, acknowledge the migration cost.

---

## Examples

### Example 1: Adding a Notification System

An Architecture Sketch proposes a `NotificationService`, `NotificationRepository`, and `notifications` table. It assumes an existing `EventBus` with subscribe capability and an `EmailAdapter`.

**Assumption Verification:**
- A1: "EventBus exists for subscribing to domain events" — Verified: Partial. `EventBus` exists but uses a publish-only interface with a handler registry at startup, not runtime subscription.
- A2: "EmailAdapter exists" — Verified: No. Email is handled by a `MailgunClient` in infrastructure, not an adapter conforming to a port.

**Naming Comparison:**
- Proposed `NotificationService` — codebase uses `Handler` for orchestration. Suggested: `SendNotificationHandler`.
- Proposed `notifications/` directory — codebase uses `src/contexts/{domain}/`. Suggested: `src/contexts/notification/`.

**Verdict:** Fit with Gaps — core approach is sound but two assumptions need correction and naming should align.

### Example 2: API Endpoint Restructuring

An Architecture Sketch proposes flattening nested endpoints from `/resources/{id}/sub-resources` to `/sub-resources?resource_id={id}`. It assumes only 3 endpoints need restructuring.

**Assumption Verification:**
- A1: "Frontend uses a centralized API client" — Verified: Yes.
- A2: "Only 3 endpoints need restructuring" — Verified: No. Grep reveals 7 endpoints using the nested pattern.

**Affected Files Estimate:**
- Backend routers: 7 files (Modified, Medium)
- Frontend call sites: 12 files (Modified, Low)
- Integration tests: 14 files (Modified, Low)
- Total: ~41 files vs sketch estimate of ~15.

**Verdict:** Fit with Gaps — approach is technically sound but scope is significantly larger than estimated. Sketch should be revised to narrow scope or confirm appetite accommodates full scope.