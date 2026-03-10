---
name: shaping-writing-problem-frames
version: 1.0.0
user-invocable: false
description: Write problem frame documents that define the trigger, impact, success criteria, appetite, non-goals, and constraints for a shaped project. Use when framing a problem for shaping, defining success criteria, setting appetite, establishing project boundaries, or articulating the trigger moment. Triggered by: problem frame, problem definition, trigger, impact, success criteria, appetite, non-goals, constraints, problem statement.
allowed-tools:
  - Read
---

# Problem Frames — Trigger, Impact, Success Criteria, Appetite & Non-Goals

Expert guidance for producing problem frame artifacts during the shaping process.

## Purpose

A problem frame defines what problem the shaping effort is solving and how much time is worth spending on it. It converts the raw facts from the Context Brief into a structured problem statement with measurable success criteria, an explicit time budget (appetite), and clear boundaries (non-goals).

The problem frame exists because solutions shaped without a clear problem statement tend to grow unbounded. By defining the trigger moment, the affected personas, the desired outcome, and the appetite before any solution work begins, the frame prevents the team from solving the wrong problem or spending too long on the right one.

Write a problem frame after the Context Brief is complete and before domain scenarios, UX breadboards, or architecture sketches begin.

---

## Output Format

```markdown
# Problem Frame: <Problem Title>

**Prepared by:** <Agent or Author>
**Date:** <YYYY-MM-DD>
**Status:** Draft | Under Review | Accepted
**Context Brief:** <Reference to the upstream Context Brief>

---

## 1. Trigger Scene

[A narrative paragraph — not a bulleted list — describing the specific moment the
problem occurs. Include who is involved, what they are trying to do, where they are
in the workflow, and what goes wrong. Paint a vivid picture so that anyone reading
this can immediately understand the pain point. This is not an abstract description
of a category of problems; it is a concrete scene.]

---

## 2. Who Feels It

| Persona | How They Experience It | Frequency | Severity |
|---------|----------------------|-----------|----------|
| <Name/Role> | <Specific description of what this persona encounters> | <Daily/Weekly/Monthly/Per event> | <Critical/High/Medium/Low> |
| <Name/Role> | <Description> | <Frequency> | <Severity> |
...

---

## 3. Current Behavior

[Description of what happens today — the status quo. Walk through the current
workflow step by step, highlighting where the pain points are. Use concrete
details, not abstractions. If there are workarounds that people use, describe
them here. The goal is to make the gap between current and desired behavior
visible.]

---

## 4. Desired Outcome

[Description of what should happen instead. This is the target state — what the
world looks like when the problem is solved. Be concrete about the observable
change: what does the user experience differently? What behavior changes? What
metric moves? This is NOT a solution description — it describes the outcome
without prescribing how to achieve it.]

---

## 5. Success Criteria

Measurable outcomes that prove the problem is solved. Each criterion must be
observable and verifiable.

1. <Criterion with specific metric or observable behavior>
2. <Criterion>
3. <Criterion>
...

---

## 6. Appetite

**Size:** <Small Batch (~1-2 weeks) | Big Batch (~6 weeks)>

**Justification:** [Why this problem deserves this amount of time. What is the
cost of the problem that makes it worth this investment? If the appetite were
smaller, what would be lost? If larger, what additional value would not justify
the extra time?]

**What "done" means at this appetite:** [The minimum viable outcome that counts
as solving the problem within this time budget. Not everything about the problem
needs to be solved — just enough to justify the investment.]

---

## 7. Non-Goals

Things explicitly out of scope for this effort, even if they are related to the
problem.

1. **<Non-goal>** — <Brief reason why this is excluded>
2. **<Non-goal>** — <Reason>
3. **<Non-goal>** — <Reason>
...

---

## 8. Constraints

Known technical, business, or regulatory constraints that bound the solution
space. These are not non-goals (things we choose not to do) but limitations
we must work within.

1. **<Constraint>** — <Why this constraint exists and how it limits the solution>
2. **<Constraint>** — <Explanation>
...
```

---

## Instructions

1. **Read the Context Brief to understand the raw situation.** Absorb the verified facts, assumptions, unknowns, and glossary. The problem frame must be grounded in evidence from the Context Brief, not in the framer's opinions. Note any high-risk assumptions that could invalidate the problem if proven wrong — these should inform the appetite decision.

2. **Identify the trigger moment.** Find the specific event or situation that causes pain. This is not "users are frustrated with X" but "when a team lead opens the weekly report on Monday morning, the data from the weekend is missing because the sync job only runs on weekdays." The trigger is a concrete scene with a who, what, when, and where. Write it as a narrative paragraph in the Trigger Scene section.

3. **Map who is affected and how severely.** For each persona who experiences the problem, describe their specific encounter with it. Frequency and severity determine priority: a problem that affects all users daily at critical severity demands different appetite than one that affects 5% of users monthly at low severity. Be honest about frequency — inflate it and the team will build for the wrong scale.

4. **Define success criteria that are observable and measurable.** Each criterion must pass the "stranger test" — could someone unfamiliar with the project verify whether this criterion is met by observing the system? "Users are happier" fails the test. "The workspace setup step has a completion rate above 60%" passes it. Prefer quantitative criteria where metrics exist; use qualitative criteria only when the outcome is genuinely not measurable.

5. **Set appetite and explicitly list non-goals to bound the effort.** The appetite is a time budget, not an estimate. It answers: "How much time is this problem worth?" Consider the cost of the problem (from the Trigger Scene and Who Feels It) against the cost of the investment. Then list non-goals — related things you are deliberately choosing not to solve. Non-goals protect the team from scope creep by making boundaries explicit before solution work begins.

---

## Quality Checklist

- [ ] The Trigger Scene is a narrative paragraph describing a specific person in a specific situation, not an abstract problem category
- [ ] The Who Feels It table includes frequency and severity for every persona, grounded in Context Brief facts
- [ ] Current Behavior describes the status quo with enough detail that someone unfamiliar could follow the workflow
- [ ] Desired Outcome describes the target state without prescribing a specific solution
- [ ] Every success criterion is observable and verifiable by someone who did not build the solution
- [ ] The appetite includes a justification that references the cost of the problem, not just the cost of the solution
- [ ] Non-goals are specific, numbered, and each includes a brief reason for exclusion

---

## Anti-Patterns

### 1. Solution Masquerading as Problem
Writing "we need a dashboard" instead of "team leads cannot see weekend data on Monday morning." The problem frame describes what is wrong, not what to build. If the frame contains solution language (add, build, implement, create), it has crossed into solution territory.

### 2. Abstract Problem Statements
Writing "users experience friction in the onboarding flow" instead of describing the specific moment of friction. Abstract statements feel true but are not actionable. The trigger scene must be concrete enough to verify by observing a real user.

### 3. Success Criteria Without Baselines
Writing "reduce drop-off rate" without stating the current rate or the target. A success criterion that does not include a measurable threshold cannot be evaluated. If no baseline exists, the criterion should state what will be measured and that a baseline will be established, not just the direction of improvement.

### 4. Appetite Based on Solution Complexity
Setting the appetite to "6 weeks because the solution is complex" instead of "6 weeks because the problem costs us $X per month in support tickets." Appetite is driven by the value of solving the problem, not the perceived difficulty of the solution. If the problem is not worth 6 weeks, the team should find a smaller solution, not accept a larger appetite.

### 5. Missing Non-Goals
Omitting the non-goals section or writing only one vague exclusion. Every real problem has adjacent territory that stakeholders will want to include. Without explicit non-goals, scope creep happens invisibly during solution work.

---

## Examples

### Example 1: Weekend Data Sync Gap

**Context:** A Context Brief documented that team leads report missing data in Monday morning reports, analytics confirmed a gap in weekend processing, and the sync job schedule was verified as weekday-only.

**How the artifact looks in practice (abbreviated):**

```markdown
# Problem Frame: Weekend Data Sync Gap

**Prepared by:** Problem Framer
**Date:** 2026-02-12
**Status:** Draft
**Context Brief:** Context Brief: Reporting Data Gaps, 2026-02-08

---

## 1. Trigger Scene

Every Monday morning at 9 AM, Sarah, a team lead at a mid-size customer, opens
her weekly performance report to prepare for the 10 AM standup. The report shows
data through Friday evening but nothing from Saturday or Sunday. She knows her
team worked over the weekend to hit a deadline, but their contributions are
invisible. She manually checks three different screens to piece together weekend
activity, delaying her standup prep by 20 minutes. By the time the sync catches
up on Monday afternoon, the standup has already happened and decisions were made
with incomplete data.

## 5. Success Criteria

1. Monday morning reports include data from Saturday and Sunday with no more
   than 2 hours of delay from the last event.
2. Team leads no longer need to manually check alternative screens to verify
   weekend data (measured by page-view analytics on workaround screens).
3. The sync job processes weekend data without manual intervention.

## 6. Appetite

**Size:** Small Batch (~1-2 weeks)

**Justification:** This affects every customer with weekend activity (approximately
40% of active accounts). The workaround costs an estimated 20 minutes per team
lead per Monday. The fix is likely a scheduling change plus verification, not a
system redesign.

**What "done" means at this appetite:** Weekend data appears in Monday reports
automatically. We are NOT optimizing sync latency for real-time — a 2-hour delay
is acceptable.

## 7. Non-Goals

1. **Real-time data sync** — The current 1-hour weekday sync cadence is acceptable; we are only fixing the weekend gap.
2. **Historical backfill** — Past Mondays with missing data will not be retroactively corrected.
3. **Custom sync schedules per customer** — All customers get the same schedule.
```

### Example 2: Bulk Resource Import

**Context:** A Context Brief documented that administrators add catalog items one at a time, with organizations of 200+ items reporting multi-day manual entry efforts and frequent data entry errors.

**How the artifact looks in practice (abbreviated):**

```markdown
# Problem Frame: Manual Catalog Entry Bottleneck

**Prepared by:** Problem Framer
**Date:** 2026-02-14
**Status:** Under Review
**Context Brief:** Context Brief: Catalog Management Pain Points, 2026-02-10

---

## 1. Trigger Scene

James is an IT administrator onboarding a 200-person organization. He has a
spreadsheet with 340 resources that need to be added to the catalog. Using the
current single-item form, he enters the first 50 items over two hours. On the
third day, he discovers that 15 items from day one have incorrect category codes
because he misread the dropdown labels. There is no way to bulk-correct them, so
he deletes and re-creates each one manually. By day four, he tells his manager
the platform "wasn't built for organizations this size."

## 6. Appetite

**Size:** Small Batch (~1-2 weeks)

**Justification:** Every new enterprise customer goes through this onboarding
pain. Sales reports it as the #2 objection in enterprise deals. The cost is
measured in churned trials and delayed onboarding (average 5 days for 100+ items
versus our target of 1 day).

**What "done" means at this appetite:** An administrator can upload a CSV, see
validation errors before committing, and import all valid rows in one action.

## 7. Non-Goals

1. **Excel file support** — CSV only; Excel parsing adds complexity without enough value for this appetite.
2. **Field mapping UI** — Columns must match our template; we are not building a general-purpose data mapper.
3. **Export and re-import** — This is an import tool, not a sync tool. Round-tripping is a separate project.
4. **Scheduled or recurring imports** — One-time upload only.
```