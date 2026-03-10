---
name: shaping-writing-milestones
version: 1.0.0
user-invocable: false
description: Produce milestone map documents that define project phases with exit criteria, sequencing rationale, go/no-go decisions, and scope-at-risk items. Use when breaking shaped work into deliverable phases, defining exit criteria, establishing milestone sequencing, or identifying scope-at-risk. Triggered by: milestone map, project milestones, phases, exit criteria, sequencing, scope at risk, milestone planning, go/no-go, delivery phases.
allowed-tools:
  - Read
---

# Milestone Maps — Phases, Exit Criteria, Sequencing & Scope at Risk

Expert guidance for producing milestone map artifacts during the shaping process.

## Purpose

A milestone map breaks a shaped project into phases with verifiable exit criteria and explicit sequencing rationale. It answers "in what order should this work be delivered, and how do we know each phase is complete?" without prescribing daily task breakdown or sprint planning.

Use this skill after the Pitch is complete. The milestone map translates the Pitch's scope and appetite into delivery phases that each produce demonstrable progress. It is the final writing artifact before the project moves to reviews and then to the building team.

Milestones are not sprints. They define exit criteria and sequencing, not daily task assignments. The building team self-organizes within each milestone to meet the exit criteria however they see fit.

---

## Output Format

```markdown
# Milestone Map: <Project Name>

**Prepared by:** <Agent or Author>
**Date:** <YYYY-MM-DD>
**Status:** Draft | Under Review | Accepted
**Pitch:** <Reference to upstream Pitch>
**Appetite:** <Small Batch (~1-2 weeks) | Big Batch (~6 weeks)>

---

## 1. Milestone Overview

| # | Milestone Name | Duration | Key Deliverable | Depends On | Confidence |
|---|---------------|----------|----------------|-----------|------------|
| M1 | <Name> | <Duration estimate> | <One-sentence deliverable> | — | High / Medium / Low |
| M2 | <Name> | <Duration> | <Deliverable> | M1 | <Confidence> |
| M3 | <Name> | <Duration> | <Deliverable> | M2 | <Confidence> |
...

---

## 2. Milestone Details

### M1: <Milestone Name>

**Scope:**
- <What is included in this milestone>
- <Scope item>
- <Scope item>

**Exit Criteria:**
- [ ] <Specific, verifiable condition that must be true — "you can tell it's done when...">
- [ ] <Criterion>
- [ ] <Criterion>

**Go/No-Go Decision:** <What must be evaluated before proceeding to the next
milestone. This is a decision point, not just a status check. What information
would cause the team to stop, adjust scope, or change approach?>

**Risks:** <Specific risks from the Risk Register that affect this milestone>

---

### M2: <Milestone Name>

[Repeat the same structure for each milestone.]

---

## 3. Sequencing Rationale

<Paragraph explaining WHY the milestones are ordered this way — not just WHAT
the order is. What would break if you reordered? What does M1 produce that M2
depends on? Why is the most uncertain work early (or late)? Address the strategic
reasoning behind the sequencing.>

---

## 4. Scope at Risk

Items that might be cut if time pressure mounts, in priority order.

| # | Feature/Scope Item | Milestone | Why At Risk | Cut Strategy |
|---|-------------------|-----------|-------------|-------------|
| 1 | <Item> | <Which milestone> | <Why this might not fit> | <What specifically gets removed and what remains> |
| 2 | <Item> | <Milestone> | <Risk reason> | <Cut strategy> |
...

---

## 5. Not a Sprint Plan

This milestone map defines exit criteria and sequencing, NOT a daily task
breakdown. The building team self-organizes within each milestone.

- Milestones are NOT sprints — they may be shorter or longer than a sprint
- Exit criteria are NOT user stories — they describe the finished state, not
  individual tasks
- The team decides HOW to meet exit criteria — the map only defines WHAT "done"
  looks like
- Go/no-go decisions are checkpoints for the team and stakeholders, not
  gates that require external approval (unless specified)

---

## 6. Dependencies and Blockers

| Milestone | External Dependency | Owner | Status | Deadline |
|-----------|-------------------|-------|--------|----------|
| <Which milestone> | <Thing outside team's control> | <Person/team> | Confirmed / Pending / At Risk | <When it must be resolved> |
...
```

---

## Instructions

1. **Read the Pitch to understand scope, appetite, and solution shape.** The Pitch's appetite determines how many milestones are appropriate (Small Batch: 1-2, Big Batch: 3-5). The solution section and Existing vs New table reveal the natural seams where work can be divided. The rabbit holes and no-gos define what is in and out of scope for each milestone.

2. **Identify natural seams in the work.** Each milestone should deliver demonstrable progress — something a stakeholder could see and evaluate. Look for seams where: (a) a new capability becomes usable, (b) a risky component can be validated, or (c) feedback from one phase informs the next. The first milestone should include the highest-risk work when possible, so problems surface early.

3. **Write exit criteria as verifiable checkboxes.** Each exit criterion must pass the "stranger test" — could someone who did not build the feature verify whether it is met by observing the system? "The import feature works" fails. "A user can upload a 1,000-row CSV, see a validation preview within 10 seconds, correct errors inline, and confirm import — all rows appear in the catalog" passes. Criteria should be conditions, not tasks.

4. **Establish sequencing rationale.** Explain the dependencies between milestones — what does M1 produce that M2 needs? Why is risky work sequenced early? What would break if milestones were reordered? The rationale should convince a reader that this is the right order, not just describe the order.

5. **Identify scope-at-risk items and their cut strategies.** For each milestone, ask: "If we run out of time, what could be cut?" Scope-at-risk items should be ranked by cut priority (cut first = least essential). Each must have a concrete cut strategy — not "remove this feature" but "ship without sort/filter on the list view; users can find items using browser search."

---

## Quality Checklist

- [ ] The number of milestones is appropriate for the appetite (1-2 for Small Batch, 3-5 for Big Batch)
- [ ] Each milestone delivers demonstrable progress that a stakeholder can observe and evaluate
- [ ] Exit criteria are specific, verifiable conditions — not tasks, not vague "feature complete" statements
- [ ] Go/no-go decisions identify what information would cause the team to stop or change approach
- [ ] The sequencing rationale explains WHY the order is chosen, not just WHAT the order is
- [ ] Scope-at-risk items have concrete cut strategies, not just "remove feature X"
- [ ] The "Not a Sprint Plan" disclaimer is included and accurately describes the team's autonomy within milestones

---

## Anti-Patterns

### 1. Milestones as Task Lists
Writing milestones as ordered lists of tasks ("1. Set up database, 2. Build API, 3. Create UI"). Milestones define outcomes, not activities. The building team decides what tasks to perform. If a milestone reads like a sprint backlog, it has crossed from shaping into planning.

### 2. Exit Criteria That Cannot Be Verified
Writing "the feature is complete" or "tests pass" as exit criteria. These cannot be verified by someone who does not know what "complete" means. Good exit criteria describe observable system behavior: "A user who navigates to /settings/notifications sees a matrix of event types and channels, can toggle individual cells, and save preferences."

### 3. Linear Sequencing Without Rationale
Ordering milestones M1, M2, M3 without explaining why M2 comes after M1. If the order is arbitrary, the milestones are not well-defined seams — they are artificial divisions. If the order is meaningful, the rationale should explain the dependency or strategic reasoning.

### 4. No Scope at Risk
Claiming every feature is essential and nothing can be cut. Every project has scope that is lower priority than other scope. Identifying scope-at-risk is not pessimism — it is preparation that prevents the team from shipping nothing because they tried to ship everything.

### 5. Milestones Too Granular or Too Coarse
A Big Batch project with 8 milestones has milestones too granular — they become sprints. A Big Batch with 1 milestone has no seams — there is no intermediate progress to evaluate. 3-5 milestones for a 6-week project provides enough checkpoints without micromanaging.

---

## Examples

### Example 1: API Key Management — 3 Milestones

**Context:** A 6-week Big Batch project to add API key management: creation, authentication, scoped permissions, and usage analytics.

```markdown
# Milestone Map: API Key Management

## 1. Milestone Overview

| # | Milestone Name | Duration | Key Deliverable | Depends On | Confidence |
|---|---------------|----------|----------------|-----------|------------|
| M1 | Key CRUD + Basic Auth | 2 weeks | Users can create, view, and revoke API keys; keys authenticate API requests | — | High |
| M2 | Scoped Permissions | 2 weeks | Keys can be scoped to specific endpoints; unauthorized calls return 403 | M1 | Medium |
| M3 | Usage Analytics | 2 weeks | Key owners can see request counts, last-used timestamp, and error rates | M2 | Medium |

## 2. Milestone Details

### M1: Key CRUD + Basic Auth

**Scope:**
- API key creation with name and expiration
- Key hashing and secure storage
- Key reveal (show-once) flow
- API key authentication middleware
- Key revocation

**Exit Criteria:**
- [ ] A user can create a new API key, see the full key value once, and copy it
- [ ] The key list shows all keys with name, prefix, created date, and status
- [ ] An API request with a valid key header authenticates successfully
- [ ] An API request with a revoked key returns 401
- [ ] Key values are stored hashed — the plaintext is never persisted

**Go/No-Go Decision:** Does key authentication add acceptable latency to API
requests? If p95 auth overhead exceeds 10ms, the team should investigate caching
before proceeding to M2.

## 3. Sequencing Rationale

M1 comes first because scoped permissions (M2) and usage analytics (M3) both
require working keys. Without M1, there is nothing to scope and nothing to
measure. M2 precedes M3 because analytics must respect permission scopes — a
key's usage dashboard should only show data for endpoints the key is authorized
to access. Reordering M2 and M3 would require retroactively applying permission
filters to analytics data.

## 4. Scope at Risk

| # | Feature/Scope Item | Milestone | Why At Risk | Cut Strategy |
|---|-------------------|-----------|-------------|-------------|
| 1 | Error rate tracking | M3 | Requires correlating key auth with downstream errors — complex join | Ship usage analytics with request counts and last-used only; add error tracking next cycle |
| 2 | Key expiration enforcement | M1 | Expiration requires a background job or middleware check | Ship with manual revocation only; add expiration enforcement as a fast-follow |
```

### Example 2: Onboarding Redesign — 2 Milestones

**Context:** A 4-week project to streamline the workspace setup wizard from 7 steps to 3, deferring optional configuration to post-setup guidance.

```markdown
# Milestone Map: Onboarding Redesign

## 1. Milestone Overview

| # | Milestone Name | Duration | Key Deliverable | Depends On | Confidence |
|---|---------------|----------|----------------|-----------|------------|
| M1 | Streamlined Wizard | 2 weeks | New 3-step setup flow replacing the 7-step wizard | — | High |
| M2 | Contextual Guidance | 2 weeks | Post-setup prompts based on chosen use case | M1 | Medium |

## 3. Sequencing Rationale

M1 must come first because the streamlined wizard determines which configuration
is deferred — and the deferred items become the content of M2's contextual
guidance. Building M2 first would require guessing what the wizard defers, then
rebuilding when the wizard design changes. M1 also delivers immediate value
(faster setup) that can be measured while M2 is built.
```