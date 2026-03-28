# Project Shaping Orchestration

Shape projects before committing to build. This bundle orchestrates a multi-agent pipeline that takes raw project ideas through structured problem framing, solution design, risk assessment, and scoping — producing decision-ready artifacts like Pitches, Architecture Sketches, and Milestone Maps that can be exported directly to Linear.

## Quick Start

Install via the [Mush CLI](https://github.com/musher-dev/mush):

```sh
mush bundle install musher-dev/project-shaping-orchestration
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Shape this project idea: [describe the feature or initiative you want to build]
```

## What's Inside

The bundle ships six agents and fifteen skills organized as a sequential shaping pipeline. Each agent produces specific artifacts that feed into the next stage.

**Stage 1 — Analysis.** The **shaping-analyst** takes raw project input and produces a **Context Brief** (structured facts, glossary, assumptions, unknowns) and a **Problem Frame** (trigger, impact, success criteria, appetite, non-goals).

**Stage 2 — Design.** The **shaping-designer** translates the problem into concrete user interactions, producing **Domain Scenarios** (personas, workflows, edge cases, domain vocabulary) and **UX Breadboards** (user flows, screen states, error states, navigation maps).

**Stage 3 — Architecture.** The **shaping-architect** maps the technical shape of the solution, producing an **Architecture Sketch** (components, data shapes, integration points, hard parts) and a **Risk Register** (rabbit holes, external dependencies, unknowables, fallback plans).

**Stage 4 — Strategy.** The **shaping-strategist** synthesizes all upstream artifacts into a persuasive **Pitch** and an actionable **Milestone Map** with phased exit criteria.

**Stage 5 — Review.** The **shaping-reviewer** validates artifacts against quality standards across six dimensions: API contracts, data models, security, test strategy, release plans, and codebase fit.

**Stage 6 — Export.** The **shaping-formatter** transforms completed artifacts into a structured **Linear Output Package** with project configuration, milestones, and starter issues.

## Usage Examples

**Shape a new feature end-to-end**
```
Shape this project: We need to add team-based permissions to our API so organizations can control member access to resources
```
Runs the full pipeline from context brief through pitch and milestone map.

**Review a shaped project**
```
Review the architecture sketch and API contracts in our shaping artifacts for security, testability, and codebase fit
```
The shaping-reviewer runs targeted reviews across the six quality dimensions.

**Export to Linear**
```
Format our completed shaping artifacts as a Linear output package with milestones and starter issues
```
The shaping-formatter produces a structured package ready for import into Linear.

**Write a specific artifact**
```
Write a problem frame for this initiative — define the trigger, impact, success criteria, appetite, and non-goals
```
Individual skills can be invoked directly without running the full pipeline.

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `writing-context-briefs` | Raw inputs → structured facts, references, glossary, assumptions, unknowns |
| `writing-problem-frames` | Trigger, impact, success criteria, appetite, non-goals, constraints |
| `writing-domain-scenarios` | Personas, mental models, workflows, edge cases, domain vocabulary |
| `writing-ux-breadboards` | User flows, screen states, error states, navigation maps, affordances |
| `writing-architecture-sketches` | Solution boundary, components, data shapes, integration points, hard parts |
| `writing-risk-registers` | Rabbit holes, external dependencies, unknowables, fallback plans |
| `writing-pitches` | Synthesized persuasive pitch for the betting table |
| `writing-milestones` | Phased delivery with exit criteria, sequencing rationale, scope-at-risk |
| `reviewing-api-contracts` | Endpoint coverage, naming consistency, error responses, authorization |
| `reviewing-data-models` | Normalization, referential integrity, naming conventions, migration safety |
| `reviewing-security` | Threat model, authentication, authorization, PII handling, audit logging |
| `reviewing-test-strategy` | Scenario coverage, test type distribution, edge cases, testability |
| `reviewing-release-plans` | Rollout strategy, migration safety, observability, rollback procedures |
| `reviewing-codebase-fit` | Architecture assumptions vs actual codebase — patterns, naming, complexity |
| `formatting-linear-output` | Artifacts → Linear project with milestones, documents, and starter issues |

### Agents

| Agent | Role |
|-------|------|
| `shaping-analyst` | Produces Context Brief and Problem Frame from raw project input |
| `shaping-designer` | Produces Domain Scenarios and UX Breadboards from the problem frame |
| `shaping-architect` | Produces Architecture Sketch and Risk Register |
| `shaping-strategist` | Synthesizes upstream artifacts into Pitch and Milestone Map |
| `shaping-reviewer` | Validates artifacts across 6 quality dimensions |
| `shaping-formatter` | Formats artifacts as structured Linear Output Package |

</details>

---

**Domain:** project-shaping · **Technologies:** Harness-agnostic · **Aliases:** shaping, shape-up, project-shaping
