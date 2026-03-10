---
name: shaping-writing-risk-registers
version: 1.0.0
user-invocable: false
description: Produce risk register documents that catalog rabbit holes, external dependencies, unknowables, scope creep risks, and fallback plans for a shaped project. Use when assessing project risks, identifying rabbit holes, cataloging external dependencies, or planning risk mitigation before build begins. Triggered by: risk register, rabbit holes, external dependencies, unknowables, scope creep, fallback plans, risk assessment, mitigation, confidence level.
allowed-tools:
  - Read
---

# Risk Registers — Rabbit Holes, Dependencies, Unknowables & Fallbacks

Expert guidance for producing risk register artifacts during the shaping process.

## Purpose

A risk register catalogs everything that could prevent a shaped project from shipping within its appetite. It transforms vague concerns ("this might be hard") into structured risks with severity ratings, mitigation strategies, and concrete fallback plans.

The risk register exists because optimism is the default state during shaping. By systematically asking "what could go wrong?" for every component, integration, and assumption, the register surfaces risks early enough to address them — either by adjusting the solution, spiking on unknowns, or pre-approving fallback plans that the building team can execute without delay.

Write a risk register after the Architecture Sketch is complete, when the technical shape of the solution is clear enough to reason about specific risks.

---

## Output Format

```markdown
# Risk Register: <Project Name>

**Prepared by:** <Agent or Author>
**Date:** <YYYY-MM-DD>
**Status:** Draft | Under Review | Accepted
**Architecture Sketch:** <Reference to upstream Architecture Sketch>
**Problem Frame:** <Reference to upstream Problem Frame>

---

## 1. Rabbit Holes

Areas where the team could spend unbounded time without clear progress indicators.

| # | Risk | Description | Severity | Mitigation | Time Cap |
|---|------|------------|----------|------------|----------|
| RH1 | <Short name> | <What could consume unbounded time and why> | High / Medium / Low | <Specific action to prevent falling in> | <Maximum time to spend before triggering fallback> |
| RH2 | ... | ... | ... | ... | ... |
...

---

## 2. External Dependencies

Things outside the team's control that could block or delay the project.

| # | Dependency | Owner | Status | Fallback | Deadline |
|---|-----------|-------|--------|----------|----------|
| ED1 | <What is depended on> | <Person or team responsible> | Confirmed / Pending / At Risk | <What to do if this dependency is not met> | <When this must be resolved by> |
| ED2 | ... | ... | ... | ... | ... |
...

---

## 3. Unknowables

Things we do not know and cannot determine without investigation.

### U1: <What We Don't Know>

- **Why It Matters:** <What decision or approach depends on this answer>
- **Spike Plan:** <Concrete steps to reduce uncertainty — who does what, for how long>
- **Time Budget:** <Maximum time to spend on this spike before making a decision with incomplete information>

### U2: <What We Don't Know>

[Repeat for each unknowable.]

---

## 4. Scope Creep Risks

Features or requests likely to be added during build that would expand scope beyond appetite.

| # | Feature/Request | Why It's Tempting | Why It's Out | Deferral Target |
|---|----------------|-------------------|-------------|-----------------|
| SC1 | <Feature that stakeholders might request> | <Why it seems related or valuable> | <Why it breaks the appetite or is a separate project> | <When it could be addressed: next cycle, separate project, never> |
...

---

## 5. Fallback Plans

For every risk with severity High, define a concrete fallback.

### Fallback for RH<N>: <Risk Name>

- **If:** <Specific trigger condition — how do we know the risk has materialized>
- **Then:** <Concrete fallback action the team takes>
- **Accepting:** <Trade-off — what is lost or reduced by taking the fallback>

[Repeat for each high-severity risk.]

---

## 6. Risk Summary

**Overall Confidence:** <High / Medium / Low>

**Justification:** <2-3 sentences explaining the confidence level. Reference the
number and severity of risks, the status of external dependencies, and the
completeness of spike plans.>

**Risks That Could Block Shipping:**
- <Risk ID and name>
- <Risk ID and name>

**Recommended Actions Before Build Starts:**
1. <Action that should be completed before the building team begins>
2. <Action>
...
```

---

## Instructions

1. **Read the Architecture Sketch and Problem Frame to understand scope and technical approach.** The Architecture Sketch's hard parts section is the primary input for rabbit holes. The component inventory identifies where complexity lives. The integration points identify external dependencies. The Problem Frame's appetite and non-goals define what "too much time" means.

2. **Identify rabbit holes — areas of unbounded complexity.** For each hard part in the Architecture Sketch, ask: "Could a team spend the entire appetite on just this piece?" If yes, it is a rabbit hole. Define a time cap — the maximum time to spend before escalating or falling back. Common rabbit holes: performance optimization without clear targets, custom implementations of solved problems, edge cases that affect <1% of users.

3. **Catalog external dependencies and verify their status.** For each integration point and external system in the Architecture Sketch, determine: who owns it, is it ready, and what happens if it is not ready? Dependencies with "Pending" or "At Risk" status need fallback plans. Dependencies with "Confirmed" status should include the evidence of confirmation (not just the assumption).

4. **List unknowables and design concrete spike plans.** Unknowables are different from risks — they are things we literally do not know and cannot reason about until we investigate. For each unknowable, design a time-boxed spike: what specific experiment, prototype, or research will reduce the uncertainty? What question will the spike answer? What happens if the spike does not produce a clear answer?

5. **Write fallback plans for every high-severity risk.** A fallback plan has three parts: the trigger (how we know the risk has materialized), the action (what the team does), and the trade-off (what is lost). Fallback plans must be pre-approved during shaping so the building team can execute them without waiting for permission.

---

## Quality Checklist

- [ ] Every hard part from the Architecture Sketch is represented as a rabbit hole with a severity rating and time cap
- [ ] Every external dependency has an owner, a status, and a fallback plan
- [ ] Every unknowable has a concrete spike plan with a time budget and specific question to answer
- [ ] Scope creep risks identify features that are genuinely tempting (not straw men) with honest reasons why they are out
- [ ] Every high-severity risk has a fallback plan with trigger, action, and trade-off
- [ ] The risk summary confidence level is consistent with the severity and count of identified risks
- [ ] Recommended actions before build are specific and actionable, not vague "reduce risk"

---

## Anti-Patterns

### 1. Vague Risks Without Mitigation
Writing "performance might be an issue" without specifying what performance, what threshold constitutes failure, or what to do about it. Every risk must have a concrete mitigation strategy or fallback plan. A risk without a response is just a worry.

### 2. Time Caps Without Enforcement Mechanisms
Setting a time cap of "2 days on performance optimization" without defining what happens at day 2. The time cap is meaningless without a fallback: "After 2 days, if p95 latency is still above 500ms, ship with a loading indicator and optimize in the next cycle."

### 3. Ignoring Scope Creep as a Risk Category
Treating scope creep as a management problem rather than a risk to be registered. During build, stakeholders will request adjacent features. The risk register should pre-identify the most likely scope creep requests and document why they are out of scope, so the building team can point to the register instead of re-litigating boundaries.

### 4. Unknowables Treated as Risks
Conflating "we don't know if the API supports batch operations" with "the API might be slow." The first is an unknowable (resolved by reading the docs or running a spike); the second is a risk (mitigated by setting performance targets and fallback plans). Unknowables require investigation; risks require mitigation.

### 5. Over-Confidence in Risk Summary
Rating overall confidence as "High" when multiple external dependencies are "Pending" and unknowables have no spike plans. The confidence level must honestly reflect the state of the register. A register with unresolved high-severity risks and pending dependencies is "Low" confidence regardless of how good the solution design is.

---

## Examples

### Example 1: Payment Integration Risk Register

**Context:** The Architecture Sketch proposes integrating a new payment provider for subscription billing. The hard parts include webhook reliability, currency conversion, and prorated billing calculations.

**How the artifact looks in practice (abbreviated):**

```markdown
# Risk Register: Payment Provider Integration

## 1. Rabbit Holes

| # | Risk | Description | Severity | Mitigation | Time Cap |
|---|------|------------|----------|------------|----------|
| RH1 | Proration calculation | Computing prorated charges for mid-cycle plan changes involves edge cases around billing cycles, timezone differences, and partial months. Team could spend weeks on edge cases. | High | Start with the simple case (prorate by day count in current period). Defer timezone-aware proration to a future cycle. | 3 days max; if not resolved, ship with round-to-nearest-day |
| RH2 | Webhook idempotency | Payment webhooks can arrive out of order or be retried. Building a fully idempotent handler that handles every ordering permutation could consume the entire appetite. | Medium | Use the payment provider's event ID as an idempotency key. Process events in creation order, skip duplicates. Do not attempt to reorder. | 2 days |

## 3. Unknowables

### U1: Webhook delivery reliability in our infrastructure

- **Why It Matters:** If webhooks are frequently delayed or lost, we need a polling fallback. This changes the architecture significantly.
- **Spike Plan:** Set up a test endpoint, subscribe to sandbox webhooks, and measure delivery latency and success rate over 48 hours.
- **Time Budget:** 2 days. If delivery rate is below 99%, add a polling reconciliation job.

## 6. Risk Summary

**Overall Confidence:** Medium

**Justification:** The payment provider's API is well-documented and the integration
pattern is standard. However, two unknowables (webhook reliability, sandbox-to-production
parity) could significantly change the architecture if results are unfavorable.

**Recommended Actions Before Build Starts:**
1. Complete the webhook reliability spike (U1) — results determine whether we need polling fallback
2. Confirm sandbox-to-production API parity with payment provider support (ED2)
```

### Example 2: Search Feature Risk Register

**Context:** The Architecture Sketch proposes adding full-text search across multiple entity types. The hard parts include index latency, relevance tuning, and handling entities with different schemas.

**How the artifact looks in practice (abbreviated):**

```markdown
# Risk Register: Full-Text Search

## 1. Rabbit Holes

| # | Risk | Description | Severity | Mitigation | Time Cap |
|---|------|------------|----------|------------|----------|
| RH1 | Relevance tuning | Search relevance is subjective and can be tuned indefinitely. Without clear success criteria, the team could spend weeks adjusting weights, boosting, and field scoring. | High | Define 10 specific search queries with expected top-3 results. Tune until those 10 pass. Do not optimize beyond that. | 2 days of tuning after initial implementation |

## 4. Scope Creep Risks

| # | Feature/Request | Why It's Tempting | Why It's Out | Deferral Target |
|---|----------------|-------------------|-------------|-----------------|
| SC1 | Faceted filtering | Users will want to filter search results by type, date, status | Facets require significant index redesign and UI work beyond the appetite | Next cycle if search adoption is high |
| SC2 | Search analytics | Stakeholders will want to know what users search for | Valuable but does not affect the core search experience | Separate project after search ships |

## 5. Fallback Plans

### Fallback for RH1: Relevance Tuning

- **If:** After 2 days of tuning, fewer than 7 of 10 test queries return expected results in top 3
- **Then:** Ship with current relevance and add a "Sort by: Newest / Relevance" toggle so users can fall back to chronological sorting
- **Accepting:** Search relevance will be "good enough" rather than "excellent" at launch
```