---
name: shaping-writing-pitches
version: 1.0.0
user-invocable: false
description: Synthesize upstream shaping artifacts into a persuasive Pitch document that communicates a shaped solution to decision-makers and implementation teams. Use when all upstream artifacts (Context Brief, Problem Frame, Domain Scenarios, UX Breadboard, Architecture Sketch, Risk Register) are complete and the solution needs to be packaged for the betting table. Triggered by: pitch, shaped pitch, problem statement, appetite, solution summary, rabbit holes, no-gos, betting table, project pitch.
allowed-tools:
  - Read
---

# Writing Shaped Pitches

Expert guidance for producing the synthesis pitch artifact during the shaping process.

## Purpose

A Pitch is the synthesis artifact of the shaping process. It combines all upstream work — Context Brief, Problem Frame, Domain Scenarios, UX Breadboard, Architecture Sketch, and Risk Register — into a single persuasive document that communicates what to build, why to build it, and how long to spend on it.

Use this skill when the shaping work is complete and the solution needs to be presented to decision-makers at a betting table, or handed off to an implementation team. The Pitch is not a specification; it is a communication artifact that transfers understanding while leaving room for builders to exercise judgment during implementation.

A well-written Pitch eliminates the need for the implementation team to re-derive reasoning behind decisions. It pre-closes rabbit holes, sets explicit boundaries, and makes the appetite clear so the team knows when to cut scope versus when to push through difficulty.

---

## Output Format

```markdown
# Pitch: <Descriptive Title>

**Status:** <Draft | Review | Accepted | Rejected>
**Appetite:** <Small Batch (~1-2 weeks) | Big Batch (~6 weeks)>
**Author:** <Name/Role>
**Date:** <YYYY-MM-DD>
**Upstream Artifacts:** <List of completed upstream artifact names>

---

## Problem

<A narrative paragraph — not a bulleted list — that paints the trigger scene.
Describe a specific person in a specific situation encountering this problem.
Make the reader feel the friction, confusion, or failure that motivates this work.
Draw from the Problem Frame to ground the narrative in observed reality.
End with the consequence: what happens if we do nothing.>

---

## Appetite

**Size:** <Small Batch (~1-2 weeks) | Big Batch (~6 weeks)>

**Justification:** <Why this size? What makes this a small/big batch? Reference the
scope of the solution and the complexity discovered during shaping. If the solution
were larger, explain what was descoped to fit.>

**Definition of Done:** <What does "done" look like at this appetite? Describe the
end state in concrete terms — what a user can do, what the system guarantees, what
observable behavior changes. This is not a task list; it is a description of the
finished state.>

---

## Solution

<Walk through the shaped solution using breadboard notation from the UX Breadboard.
Describe the key places, affordances, and flows. Use prose paragraphs that reference
breadboard elements, not wireframes or pixel-level details.>

### Flow: <Primary Flow Name>

**Starting Place:** <PLACE_NAME>

<Describe what the user sees conceptually — what affordances are available, what
information is displayed. Walk through the interaction step by step.>

**Transition:** <Action that moves the user to the next place>

**Next Place:** <PLACE_NAME>

<Continue describing the flow through each place and transition until the flow
reaches its terminal state.>

### Flow: <Secondary Flow Name>

<Repeat for each significant flow. Most pitches have 2-4 flows.>

### Key Design Decisions

- **<Decision Area>:** <What was decided and why. Reference the Architecture Sketch
  or Domain Scenarios where relevant.>
- **<Decision Area>:** <Repeat for each significant design decision.>

---

## Existing vs New

| Aspect | Existing | New | Modified |
|--------|----------|-----|----------|
| <Component/Capability> | <What already exists> | <What must be built from scratch> | <What existing thing changes> |
...

<Add rows for every component, capability, data model, or integration point that
the solution touches. This table makes scope tangible.>

---

## Rabbit Holes

### RH1: <Short Name>

We considered <option A> vs <option B>. Decision: <chosen approach> because
<rationale>. <Note what the rejected option would have cost.>

### RH2: <Short Name>

We considered <option A> vs <option B>. Decision: <chosen approach> because
<rationale>.

<Include 3-7 rabbit holes. Each must be a decision, not merely a risk observation.
The implementation team should not need to re-evaluate these choices.>

---

## No-Gos

1. **<Feature/Scope Item>** — <Brief reason why this is explicitly excluded, even
   though it may seem related.>
2. **<Feature/Scope Item>** — <Brief reason.>
3. **<Feature/Scope Item>** — <Brief reason.>

<Include 3-8 no-gos. These protect the team from scope creep.>

---

## Open Questions for Implementation Team

1. <Decision that requires implementation-level knowledge. Frame as a question.>
2. <Another implementation decision left to the builders.>
3. <Another implementation decision left to the builders.>

<Include 2-5 open questions. These are decisions the shapers intentionally did NOT
make because they require hands-on technical context.>

---

## Source Artifacts

| Artifact | Status | Key Findings Summary |
|----------|--------|---------------------|
| Context Brief | <Complete/Draft/Pending> | <1-2 sentence summary of what this artifact contributed> |
| Problem Frame | <Status> | <Summary> |
| Domain Scenarios | <Status> | <Summary> |
| UX Breadboard | <Status> | <Summary> |
| Architecture Sketch | <Status> | <Summary> |
| Risk Register | <Status> | <Summary> |
```

---

## Instructions

1. **Read all upstream artifacts.** Open and thoroughly read the Context Brief, Problem Frame, Domain Scenarios, UX Breadboard, Architecture Sketch, and Risk Register. Identify the key findings, decisions, and constraints from each. If any artifact is missing or incomplete, note it in the Source Artifacts table with "Pending" or "Draft" status. Do not fabricate information that upstream artifacts should have provided.

2. **Write the Problem as a narrative scene.** Using the Problem Frame as your primary source, craft a paragraph that makes the reader feel the pain. Choose a specific trigger scenario — a real person in a real situation — and narrate what happens. Do not list bullet points. End with the cost of inaction. The Problem section is the emotional engine of the Pitch; it must create urgency without exaggeration.

3. **Define the Solution using breadboard notation.** Translate the UX Breadboard flows into prose that walks the reader through the shaped solution. Name each flow. For each flow, describe the starting place, the affordances available, the transitions, and the end state. Reference Architecture Sketch decisions where they affect the user-facing solution. Do not include wireframes, pixel measurements, or visual design details.

4. **Convert risks into decisions.** Read the Risk Register and transform each significant risk into a Rabbit Hole entry. For each, articulate the options considered and document the decision made. If a risk cannot be pre-decided (requires implementation-level knowledge), move it to Open Questions instead. Every Rabbit Hole must close a decision; every Open Question must explain why the decision was deferred.

5. **Set clear boundaries.** Write the No-Gos list by identifying related features and natural extensions explicitly excluded from this project. For each, provide a brief reason. Then write the Appetite section, justifying the chosen time budget by referencing the scope. Define "done" in concrete, observable terms — not as a task checklist but as a description of the finished state.

---

## Quality Checklist

- [ ] The Problem section is a narrative paragraph (not a bulleted list) that makes the reader feel the pain of a specific person in a specific situation
- [ ] The Appetite includes an explicit time budget, a justification for the chosen size, and a concrete definition of done
- [ ] The Solution uses breadboard notation (places, affordances, flows) and does not include wireframes or pixel-level specifications
- [ ] The Existing vs New table covers every component or integration point the solution touches, making scope tangible
- [ ] Every Rabbit Hole entry documents a decision (not just a risk) with considered options, the chosen approach, and the rationale
- [ ] No-Gos are numbered, specific, and each includes a brief reason for exclusion
- [ ] The Source Artifacts table lists all six upstream artifacts with their status and a key findings summary

---

## Anti-Patterns

### 1. The Specification Pitch
Writing the Pitch as a detailed technical specification with database schemas and API contracts. A Pitch communicates the shaped solution at a conceptual level. It tells builders what to build and why, not how to build it. Implementation details belong to the implementation team.

### 2. The Orphan Pitch
Writing a Pitch without completing upstream artifacts first. The Pitch is a synthesis document — it combines findings from six upstream artifacts. Skipping upstream work produces a Pitch built on assumptions rather than shaped understanding.

### 3. The Risk List Instead of Decisions
Listing risks in the Rabbit Holes section without making decisions. "We might have performance issues with large datasets" is a risk observation. A proper rabbit hole is: "We considered lazy loading vs pagination. Decision: pagination because it gives predictable response times and simpler client logic."

### 4. The Infinite Appetite
Failing to constrain the appetite or writing a vague definition of done. "Done means the feature works well" is not a definition. "Done means a user can create, edit, and delete items from the list view without navigating to a separate page" is.

### 5. The Missing No-Gos
Omitting No-Gos or writing only one vague exclusion. Every shaped project has adjacent scope that stakeholders or builders will be tempted to include. No-Gos protect the project from scope creep by making exclusions explicit before work begins.

---

## Examples

### Example 1: Notification Preferences Overhaul

**Context:** Users receive too many notifications and have no granular control. Upstream shaping identified per-event-type control with sensible defaults as the solution.

The **Problem** section narrates a project manager returning from a two-day conference to 847 unread notification emails, mostly status updates on tasks she already completed. She spends 40 minutes scanning for the three that matter. Cost of inaction: users disable notifications entirely, missing critical alerts.

The **Appetite** is Big Batch (~6 weeks) because the solution touches notification infrastructure, user preferences storage, and the settings UI. Done means: users can configure preferences per event type per channel, and new event types inherit sensible defaults.

The **Rabbit Holes** include: "We considered per-workspace preferences vs global preferences. Decision: global with per-workspace overrides, because most users want consistent settings but team leads need workspace-specific control."

The **No-Gos** include: (1) notification scheduling/digest mode — separate project; (2) custom notification sounds — low value, high complexity; (3) notification templates — out of appetite.

### Example 2: Bulk Import for Resource Catalog

**Context:** Administrators add catalog items one at a time. Organizations with 200+ items report multi-day manual entry. CSV is the dominant format; validation feedback is the primary pain point.

The **Problem** section narrates an IT administrator onboarding a 200-person organization with 340 resources. Using the single-item form, she estimates 3 days of data entry. On day two, she discovers 15 items have incorrect category codes with no way to bulk-correct them.

The **Appetite** is Small Batch (~1-2 weeks) — constrained to CSV upload with validation preview. Done means: upload a CSV, see validation preview with errors highlighted row by row, correct errors inline, confirm import, all items appear immediately.

The **Existing vs New** table shows: item creation endpoint exists but handles single items only (modified for batch); CSV parser is new; validation preview UI is new; catalog list view exists unchanged.

The **No-Gos** include: (1) Excel support — CSV only; (2) export/re-import round-tripping — separate project; (3) scheduled imports — out of appetite; (4) field mapping UI — columns must match template.