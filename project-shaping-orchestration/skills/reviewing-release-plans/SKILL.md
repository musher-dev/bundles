---
name: shaping-reviewing-release-plans
version: 1.0.0
user-invocable: false
description: Produce a release plan review evaluating rollout strategy, migration safety, observability, rollback procedures, feature flags, and communication plans for a shaped project. Use when a Milestone Map and Architecture Sketch need release readiness assessment. Triggered by: release plan review, rollout strategy, migration plan, observability, rollback plan, deployment, feature flags, release readiness.
allowed-tools:
  - Read
---

# Release Plan Review

Expert guidance for producing release plan review artifacts during the shaping process.

## Purpose

A release plan review evaluates whether the shaped project can be released safely and incrementally. It catches deployment risks, missing observability, inadequate rollback procedures, and communication gaps before the building team ships anything.

Use this skill after the Milestone Map and Architecture Sketch are complete. The review ensures each milestone can be deployed independently, that the team will know when something goes wrong, and that they can recover when it does.

---

## Output Format

```markdown
# Release Plan Review

**Artifact Reviewed:** <Milestone Map and Architecture Sketch names>
**Date:** <YYYY-MM-DD>
**Reviewer Role:** Release/Ops Planner

---

## Dimension Ratings

| # | Dimension | Rating | Notes |
|---|-----------|--------|-------|
| 1 | Rollout Strategy | Pass / Partial / Fail | <Brief justification> |
| 2 | Migration Safety | Pass / Partial / Fail | <Brief justification> |
| 3 | Observability | Pass / Partial / Fail | <Brief justification> |
| 4 | Rollback Plan | Pass / Partial / Fail | <Brief justification> |
| 5 | Feature Flags | Pass / Partial / Fail | <Brief justification> |
| 6 | Communication Plan | Pass / Partial / Fail | <Brief justification> |

---

## Release Sequence

| Step | Action | Owner | Duration | Rollback Trigger |
|------|--------|-------|----------|-----------------|
| 1 | <Deployment action> | <Team/person> | <Time estimate> | <What metric or signal triggers rollback> |
...

---

## Observability Checklist

- [ ] Key metrics defined (latency, error rate, throughput)
- [ ] Dashboard created or updated for new functionality
- [ ] Alerts configured for anomalous behavior
- [ ] Log queries prepared for debugging common issues
- [ ] Health check endpoints updated to reflect new components
- [ ] Canary/smoke tests defined for post-deploy verification

---

## Rollback Procedure

### Trigger Criteria
<Specific metrics or signals that indicate a rollback is needed.>

### Steps
1. <Concrete rollback action>
2. <Action>
3. <Action>

### Post-Rollback Verification
<How to verify the rollback was successful.>

### Data Considerations
<What happens to data created during the failed release. Is it compatible
with the rolled-back version?>

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

- [ ] Each milestone can be deployed independently without breaking existing functionality
- [ ] Data/schema migrations can run without downtime (or downtime is scheduled and communicated)
- [ ] Observability covers key metrics, alerts, and debugging tools
- [ ] Rollback procedure is documented with trigger criteria and verification steps
- [ ] Feature flags are used for gradual exposure where appropriate
- [ ] Stakeholders and affected teams are identified for release communication
- [ ] No Critical findings remain unresolved

---

## Verdict

**<Approved / Approved with Conditions / Rejected>**

<Summary rationale — 2-3 sentences.>
```

---

## Instructions

1. **Map the release sequence.** Read the Milestone Map and determine how each milestone will be deployed. Identify the deployment steps, owners, and timing. For each step, define what metric or signal would trigger a rollback. If milestones must be deployed in sequence, document the dependencies.

2. **Evaluate each dimension.**
   - **Rollout Strategy:** Check for phased rollout (canary, percentage-based, full). Assess whether the team can deploy to a subset of users first.
   - **Migration Safety:** Cross-reference with the Architecture Sketch data shapes. Can schema changes run online? Are expand-contract patterns used? What about data backfills?
   - **Observability:** Verify metrics, dashboards, alerts, and log queries are planned. The building team should know what "healthy" looks like before shipping.
   - **Rollback Plan:** Verify there is a documented procedure. Check that rollback handles data created during the failed release.
   - **Feature Flags:** Assess whether new functionality uses feature flags for gradual exposure and kill switches.
   - **Communication Plan:** Identify who needs to know about the release (support, documentation, customers) and when.

3. **Build the release sequence.** Create a step-by-step table of deployment actions with owners and rollback triggers. This is the operational playbook for release day.

4. **Evaluate the observability checklist.** Mark each item as covered or uncovered. Uncovered items become findings.

5. **Render the verdict.** All conditions met = "Approved." Manageable gaps = "Approved with Conditions." Missing rollback plan or unsafe migrations = "Rejected."

---

## Quality Checklist

- [ ] The release sequence covers every deployment step from code merge to production verification
- [ ] Each milestone's deployment is assessed independently
- [ ] Migration safety is evaluated per the Architecture Sketch's data shape changes
- [ ] The rollback procedure includes trigger criteria, steps, verification, and data considerations
- [ ] The observability checklist is fully evaluated with uncovered items flagged
- [ ] Communication plan identifies specific stakeholders and timing
- [ ] The verdict is consistent with the Conditions for Approval checklist

---

## Anti-Patterns

### 1. Big-Bang Releases
Proposing to deploy all milestones at once. Each milestone should be independently deployable. If milestones cannot be deployed separately, the milestone boundaries are wrong.

### 2. Rollback as an Afterthought
Writing "rollback: redeploy previous version" without considering data. If the release includes a migration that adds data, rolling back the code does not roll back the data. The procedure must address data compatibility.

### 3. Observability Gaps Dismissed as "We'll Add It Later"
Shipping without metrics, alerts, or dashboards with a plan to "add observability after launch." If you cannot tell whether the release is healthy, you cannot tell when to rollback.

### 4. Feature Flags for Everything
Proposing feature flags for every change, including backend logic and data schema. Feature flags add complexity. Use them for user-facing behavior changes that benefit from gradual rollout, not for internal implementation details.

### 5. Missing Communication Plan
Deploying changes that affect user behavior without notifying support, documentation, or customer-facing teams. These teams need advance notice to update their materials and prepare for questions.

---

## Examples

### Example 1: API Key Management Release

The Milestone Map has 3 milestones: M1 (Key CRUD + Auth), M2 (Scoped Permissions), M3 (Usage Analytics).

**Rollout Strategy** is rated **Pass** because each milestone is independently deployable: M1 adds new endpoints without affecting existing ones, M2 adds permission checking that defaults to "allow all" until configured, M3 adds read-only analytics with no impact on existing functionality.

**Rollback Plan** is rated **Partial** because M1 includes a database migration adding the `api_keys` table. The rollback procedure says "redeploy previous version" but does not address what happens to API keys created between deploy and rollback. Recommendation: document that rollback of M1 should disable the new key endpoints via feature flag while leaving the table in place, then clean up in a follow-up migration.

### Example 2: Onboarding Redesign Release

The Milestone Map has 2 milestones: M1 (Streamlined Wizard), M2 (Contextual Guidance).

**Feature Flags** is rated **Pass** because M1 uses a feature flag to gradually roll out the new wizard: 5% of new signups see it first, then 25%, then 100%. The old wizard remains available as a fallback. M2 builds on M1 and also uses a flag.

**Communication Plan** is rated **Fail** because the onboarding change affects every new user, but no plan exists to notify the customer success team, update the help documentation, or inform existing customers about the new experience. Recommendation: add communication steps to the release sequence for documentation updates, CS team briefing, and changelog entry.