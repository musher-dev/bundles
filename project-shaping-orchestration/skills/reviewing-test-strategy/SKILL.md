---
name: shaping-reviewing-test-strategy
version: 1.0.0
user-invocable: false
description: Produce a test strategy review evaluating scenario coverage, test type distribution, edge case coverage, testability architecture, data requirements, and regression safety for a shaped project. Use when a Pitch and Domain Scenarios need testing assessment before implementation. Triggered by: test strategy review, testability, coverage gaps, test types, testing approach, quality assurance, test plan review, testing pyramid.
allowed-tools:
  - Read
---

# Test Strategy Review

Expert guidance for producing test strategy review artifacts during the shaping process.

## Purpose

A test strategy review evaluates whether the shaped solution can be tested effectively and whether the testing approach covers the scenarios, edge cases, and regression risks identified during shaping. It catches testability problems and coverage gaps before the implementation team starts building.

Use this skill after the Pitch and Domain Scenarios are complete. The review cross-references domain scenarios against test types to identify coverage gaps, assesses whether the architecture supports efficient testing, and recommends test boundaries.

---

## Output Format

```markdown
# Test Strategy Review

**Artifact Reviewed:** <Pitch and Domain Scenarios names>
**Date:** <YYYY-MM-DD>
**Reviewer Role:** QA/Test Strategist

---

## Dimension Ratings

| # | Dimension | Rating | Notes |
|---|-----------|--------|-------|
| 1 | Scenario Coverage | Pass / Partial / Fail | <Brief justification> |
| 2 | Test Type Distribution | Pass / Partial / Fail | <Brief justification> |
| 3 | Edge Case Coverage | Pass / Partial / Fail | <Brief justification> |
| 4 | Testability Architecture | Pass / Partial / Fail | <Brief justification> |
| 5 | Data Requirements | Pass / Partial / Fail | <Brief justification> |
| 6 | Regression Safety | Pass / Partial / Fail | <Brief justification> |

---

## Coverage Matrix

| Scenario | Unit | Integration | E2E | Manual | Gap? |
|----------|------|------------|-----|--------|------|
| <Core scenario from Domain Scenarios> | <Covered/Not needed> | <Covered/Not needed> | <Covered/Not needed> | <Covered/Not needed> | <Yes/No> |
...

---

## Testability Risks

Aspects of the architecture that make testing difficult.

| # | Risk | Component/Area | Impact | Mitigation |
|---|------|---------------|--------|------------|
| 1 | <What makes testing hard> | <Which component> | <Consequence for test quality> | <How to make it testable> |
...

---

## Recommended Test Boundaries

Where to draw the line between test types.

- **Unit tests:** <What to unit test and what to skip>
- **Integration tests:** <What integration boundaries to test>
- **End-to-end tests:** <Which critical paths need E2E coverage>
- **Manual testing:** <What requires human judgment>

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

## Recommendations

1. <Priority-ordered recommendation>
2. ...

---

## Conditions for Approval

- [ ] Every core scenario from Domain Scenarios is covered by at least one test type
- [ ] "Must Handle" edge cases have explicit test coverage assigned
- [ ] The architecture supports testing without excessive mocking or complex setup
- [ ] Test data requirements are identified (fixtures, factories, seed data)
- [ ] Regression risks from modified existing components are addressed
- [ ] No Critical findings remain unresolved

---

## Verdict

**<Approved / Approved with Conditions / Rejected>**

<Summary rationale — 2-3 sentences.>
```

---

## Instructions

1. **Build the coverage matrix.** List every core scenario and edge case from the Domain Scenarios. For each, determine which test types provide coverage: unit tests (isolated logic), integration tests (component boundaries), E2E tests (full user flows), or manual testing (subjective quality). Mark gaps where no test type covers a scenario.

2. **Assess testability architecture.** Examine the Architecture Sketch for components that are difficult to test: tight coupling, external dependencies without abstractions, stateful operations without reset mechanisms, time-dependent behavior, and non-deterministic operations. Record each as a testability risk with mitigation.

3. **Define test boundaries.** Recommend where each test type should focus. Unit tests cover business logic and validation rules. Integration tests cover API boundaries and data access. E2E tests cover critical happy paths. Manual testing covers subjective UX quality. The goal is efficient allocation, not maximum coverage.

4. **Evaluate data requirements and regression safety.** Identify what test data the building team needs: fixtures for unit tests, factories for integration tests, seed data for E2E. Assess whether changes to existing components risk regressions and whether existing tests cover those risks.

5. **Render the verdict.** All core scenarios covered, no testability blockers = "Approved." Some gaps but manageable = "Approved with Conditions." Critical scenarios untestable = "Rejected."

---

## Quality Checklist

- [ ] The coverage matrix includes every core scenario and "Must Handle" edge case from Domain Scenarios
- [ ] Each scenario has at least one test type assigned, or the gap is explicitly noted
- [ ] Testability risks identify specific components and concrete mitigations
- [ ] Test boundaries are defined for each test type with clear scope
- [ ] Data requirements specify what kinds of test data the team needs to prepare
- [ ] Regression risks from existing component modifications are identified
- [ ] The verdict is consistent with the Conditions for Approval checklist

---

## Anti-Patterns

### 1. Testing Everything at E2E Level
Proposing E2E tests for every scenario. E2E tests are slow, flaky, and expensive. Most scenarios should be covered at the unit or integration level, with E2E reserved for critical paths.

### 2. Ignoring Edge Cases in Test Planning
Building the coverage matrix from core scenarios only while ignoring the edge case table. "Must Handle" edge cases often reveal the most important test cases.

### 3. Testability as an Afterthought
Noting that a component is "hard to test" without recommending architectural changes. If testing requires mocking 12 dependencies, the architecture has a testability problem that should be addressed during design.

### 4. No Test Data Strategy
Assuming test data will be "figured out during implementation." Test data requirements (realistic volumes, edge case data, multi-tenant isolation) should be identified during shaping.

### 5. Regression Blindness
Focusing only on new code without assessing regression risk in modified existing components. The Pitch's "Existing vs New" table identifies what changes — those changes need regression coverage.

---

## Examples

### Example 1: Bulk Import Feature

The Domain Scenarios describe CSV upload, validation preview, error correction, and import confirmation. The Architecture Sketch shows a new CSV parser, a validation engine, and modifications to the existing item creation endpoint.

**Testability Architecture** is rated **Partial** because the CSV parser and validation engine are independent and easily unit-testable, but the import confirmation flow depends on the file storage service. Without an abstraction layer, integration tests would need actual file storage. Recommendation: add a file storage port/interface.

**Coverage Matrix excerpt:**

| Scenario | Unit | Integration | E2E | Manual | Gap? |
|----------|------|------------|-----|--------|------|
| Upload valid CSV | Not needed | Covered | Covered | No | No |
| Validation finds errors | Covered | Covered | Not needed | No | No |
| Fix errors inline | Not needed | Not needed | Covered | Yes (UX feel) | No |
| Import 10,000 rows | Not needed | Covered (performance) | Not needed | No | No |

### Example 2: Notification Preferences

The Domain Scenarios show preference matrix configuration, per-event toggles, and notification routing based on preferences. The Architecture Sketch modifies the existing notification dispatcher.

**Regression Safety** is rated **Partial** because the notification dispatcher is modified to check user preferences before sending. Existing tests cover the dispatch path but do not test the new preference-checking logic. If preference checking introduces a bug, all notifications could be silently suppressed. Recommendation: add integration tests that verify notifications still send when preferences allow, not just when they block.