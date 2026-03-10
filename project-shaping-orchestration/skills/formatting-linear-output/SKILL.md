---
name: shaping-formatting-linear-output
version: 1.0.0
user-invocable: false
description: Transform completed shaping artifacts (Pitch, Milestone Map, and upstream documents) into a structured Linear Output Package ready for import into Linear, including project configuration, milestones, document references, and starter issues for the first milestone. Use when shaping is complete and the team is ready to transition from shaping to building. Triggered by: linear output, linear formatting, project import, milestone structure, starter issues, linear project, output formatting, linear export.
allowed-tools:
  - Read
---

# Linear Output Formatting

Expert guidance for transforming shaped artifacts into a Linear-ready output package.

## Purpose

Transform completed shaping artifacts into a structured package that can be imported into Linear (or any project management tool with similar concepts). This is a formatting skill, not a writing or reviewing skill — it does not create new content or evaluate quality. It reorganizes existing decisions from the Pitch, Milestone Map, and other artifacts into the fields and structures that Linear expects.

Use this skill when shaping is complete and the team is ready to begin building. The output package provides everything needed to set up the Linear project in one pass: project metadata, milestones with exit criteria, document references, and coarse-grained starter issues for the first milestone only.

---

## Output Format

```markdown
# Linear Output Package

## 1. Linear Project Config

| Field | Value |
|-------|-------|
| **Project Name** | <From Pitch title> |
| **Identifier** | <SHORT-CODE, 3-5 uppercase letters derived from project name> |
| **Status** | Backlog |
| **Priority** | <Urgent / High / Medium / Low — derived from Pitch appetite> |
| **Target Date** | <From Milestone Map final milestone target> |
| **Lead** | <From Pitch or "Assign on kickoff"> |
| **Teams** | <Team names involved, from Pitch> |
| **Labels** | <Relevant labels: "shaped", area labels from Architecture Sketch> |

---

## 2. Project Description

> Paste the following as the Linear project description.

### Problem

<2-3 sentences condensed from the Pitch problem statement. Focus on who is
affected and what the cost of inaction is.>

### Solution

<2-3 sentences condensed from the Pitch solution summary. Focus on the approach,
not implementation details.>

### Appetite

<1 sentence stating the time/effort budget from the Pitch.>

### Key Decisions

- <Decision 1 from Pitch Rabbit Holes — the most important pre-made decision>
- <Decision 2>
- <Decision 3>

### Shaped Artifacts

This project was shaped with the following artifacts (attached as documents):
Context Brief, Problem Frame, Domain Scenarios, UX Breadboard, Architecture
Sketch, Risk Register, Pitch, Milestone Map.

---

## 3. Milestones

### Milestone 1: <Name from Milestone Map>

**Description:** <2-3 sentences summarizing what this milestone delivers.>

**Target Date:** <From Milestone Map>

**Exit Criteria:**
- [ ] <Criterion 1 — a verifiable condition, not a task>
- [ ] <Criterion 2>
- [ ] <Criterion 3>

---

### Milestone 2: <Name from Milestone Map>

**Description:** <2-3 sentences.>

**Target Date:** <From Milestone Map>

**Exit Criteria:**
- [ ] <Criterion 1>
- [ ] <Criterion 2>
- [ ] <Criterion 3>

---

<Repeat for each milestone in the Milestone Map.>

---

## 4. Documents

| Document | Source Artifact | Content Summary | Attachment? |
|----------|---------------|-----------------|-------------|
| Context Brief | Context Brief | <1-sentence summary> | Yes |
| Problem Frame | Problem Frame | <1-sentence summary> | Yes |
| Domain Scenarios | Domain Scenarios | <1-sentence summary> | Yes |
| UX Breadboard | UX Breadboard | <1-sentence summary> | Yes |
| Architecture Sketch | Architecture Sketch | <1-sentence summary> | Yes |
| Risk Register | Risk Register | <1-sentence summary> | Yes |
| Pitch | Pitch | <1-sentence summary> | Yes |
| Milestone Map | Milestone Map | <1-sentence summary> | Yes |

---

## 5. Starter Issues for M1

> Only create starter issues for the FIRST milestone. Keep them coarse-grained.
> The building team will decompose further during development.

| Issue Title | Description | Labels | Priority | Estimate |
|-------------|-------------|--------|----------|----------|
| <Verb-noun format, e.g., "Set up database schema"> | <1-2 sentences: what and why, referencing the relevant shaping artifact> | <Area labels> | <Priority> | <S / M / L> |
| <Issue title> | <Description> | <Labels> | <Priority> | <Estimate> |
| <Issue title> | <Description> | <Labels> | <Priority> | <Estimate> |

**Starter Issue Guidelines:**
- 3-7 issues maximum for M1
- Each issue should represent 1-3 days of work, not hours
- Issues are starting points, not a complete breakdown
- Reference shaping artifacts in descriptions so builders have context
- Use verb-noun format for titles: "Implement X," "Set up Y," "Add Z"

---

## 6. Import Checklist

- [ ] Create Linear project with fields from Section 1
- [ ] Paste project description from Section 2
- [ ] Create milestones from Section 3 with target dates
- [ ] Attach shaping documents listed in Section 4
- [ ] Create starter issues from Section 5 and assign to M1
- [ ] Assign project lead
- [ ] Set project target date to final milestone date
- [ ] Add project to relevant team roadmap
- [ ] Notify team members that the project is ready for kickoff

---

## What NOT to Import

**Do not create Linear issues for:**

- **Individual domain scenarios.** Scenarios are reference material, not work items.
  Creating an issue per scenario produces dozens of micro-tasks that constrain
  how the team works.
- **Risk register items.** Risks are monitoring concerns, not tasks. The team
  reviews risks during development, not closes them as issues.
- **UX breadboard screens.** Breadboard elements are design guidance, not
  implementation tickets. The team decides how to implement screens during build.
- **Architecture Sketch components.** Components are structural guidance, not tasks.
  Creating an issue per component leads to artificial sequencing.
- **Milestones beyond M1.** Only create starter issues for the first milestone.
  The team plans subsequent milestones as they complete earlier ones.

**Why:** Shaping produces direction, not a task list. Over-importing removes
builder autonomy. The building team references shaping documents and creates
their own issues as they work.
```

---

## Instructions

1. **Read the Pitch and Milestone Map as primary inputs.** The Pitch provides project metadata (title, problem, solution, appetite, key decisions). The Milestone Map provides milestone structure, sequencing, target dates, and exit criteria. These two artifacts contain most of what is needed.

2. **Extract project metadata for Linear project configuration.** Derive the project name from the Pitch title. Generate a short identifier (3-5 uppercase letters). Map appetite to priority (tight appetite = High/Urgent, standard = Medium, exploratory = Low). Pull the target date from the final milestone.

3. **Structure milestones with exit criteria from the Milestone Map.** Transfer each milestone name, description, and target date. Convert exit criteria into checkbox format. Ensure criteria are verifiable conditions, not vague statements. If Milestone Map criteria are vague, flag this but do not invent criteria.

4. **Create starter issues for M1 only.** Read the first milestone's scope. Create 3-7 coarse-grained issues that give the building team a starting point. Each issue should reference the relevant shaping artifact. Keep issues at the 1-3 day level.

5. **Generate the import checklist and verify completeness.** Walk through all six sections to confirm every field is populated. Check that the document table lists all 8 artifacts. Verify starter issues reference shaping artifacts. Confirm the "What NOT to Import" section is included.

---

## Quality Checklist

- [ ] Project configuration has all fields populated with values derived from actual shaping artifacts
- [ ] Project description is a condensation of the Pitch, not a copy-paste of the entire Pitch
- [ ] Every milestone from the Milestone Map appears in Section 3 with exit criteria in checkbox format
- [ ] Document table lists all 8 standard shaping artifacts with accurate content summaries
- [ ] Starter issues exist only for M1, number between 3-7, and are coarse-grained (1-3 day scope)
- [ ] "What NOT to Import" section is present and warns against over-decomposition
- [ ] Import checklist is complete and actionable

---

## Anti-Patterns

### 1. Over-Decomposition
Creating a Linear issue for every scenario, component, or risk. This removes builder autonomy and turns shaping output into a prescriptive task list. Starter issues should be coarse-grained starting points, not a complete work breakdown.

### 2. Copy-Paste Project Description
Pasting the entire Pitch as the Linear project description. The description should be a condensed summary (problem, solution, appetite, key decisions) that orients someone opening the project for the first time. The full Pitch is attached as a document.

### 3. Premature Issue Creation for Later Milestones
Creating issues for M2, M3, and beyond before M1 is complete. Later milestones will change as the team learns from building M1. Only M1 gets starter issues.

### 4. Vague Exit Criteria
Transferring criteria like "feature is complete" without making them verifiable. Each criterion should describe a specific observable condition checkable by someone who did not build the feature.

### 5. Missing Artifact References
Creating issues without referencing shaping artifacts. Builders who pick up an issue need to know where to find the domain scenarios, architecture sketch, or UX breadboard. Every issue description should include at least one artifact reference.

---

## Examples

### Example 1: API Key Management Feature

**Context:** Shaping complete for API Key Management. Pitch defines 6-week appetite. Milestone Map has 3 milestones: M1 (Key CRUD + Basic Auth), M2 (Scoped Permissions), M3 (Usage Analytics).

**Linear Project Config:**
- Project Name: API Key Management
- Identifier: APIKEY
- Priority: High (6-week appetite, core infrastructure)
- Target Date: 2026-04-15 (from M3)

**Starter Issues for M1 (5 issues):**
1. "Set up API key domain model and database schema" — References Architecture Sketch data shape. S estimate.
2. "Implement key generation and hashing" — References Domain Scenarios for security requirements. M estimate.
3. "Add CRUD endpoints for API keys" — References Architecture Sketch API surface. M estimate.
4. "Integrate API key authentication middleware" — References Architecture Sketch integration points. L estimate.
5. "Write integration tests for key lifecycle" — References Domain Scenarios for happy path and edge cases. M estimate.

**What NOT to Import:** The Domain Scenarios list 14 scenarios including edge cases like "revoke key while request in flight." These are reference material, not issues. The building team uses them when implementing issues 1-5.

### Example 2: Workspace Onboarding Redesign

**Context:** Shaping complete for onboarding redesign. Pitch defines 4-week appetite. Milestone Map has 2 milestones: M1 (Streamlined Wizard), M2 (Contextual Guidance).

**Project Description (condensed from Pitch):**

Problem: New users take 12 minutes to complete workspace setup with 34% drop-off at integrations step. The current wizard requires all configuration before showing any value.

Solution: Restructure to deliver a usable workspace in under 3 minutes by deferring optional configuration. Required steps reduced from 7 to 3.

Appetite: 4 weeks. Limited to setup wizard; onboarding email sequence is separate.

**Starter Issues for M1 (4 issues):**
1. "Redesign setup wizard to 3-step flow" — References UX Breadboard. L estimate.
2. "Implement use-case selection step" — References Domain Scenarios. M estimate.
3. "Defer integration setup to post-wizard" — References Architecture Sketch. M estimate.
4. "Add setup completion analytics events" — References Milestone Map M2 dependencies. S estimate.