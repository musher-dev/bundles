---
name: designing-onboarding-journeys
version: 1.0.0
description: Design multi-page onboarding journeys for developer documentation including progressive complexity sequencing, persona-based path branching, milestone placement, time-to-first-success optimization, and handoff patterns between page types. Use when designing getting-started flows, tutorial sequences, or optimizing first-run developer experience. Triggered by: onboarding journey, tutorial sequence, getting started flow, progressive complexity, persona branching, time to first success, TTFS, developer onboarding, first-run experience, quickstart flow, onboarding path, tutorial progression.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Designing Onboarding Journeys

## Purpose

Design multi-page onboarding sequences that take a developer from zero knowledge to productive use. While scaffolding-page-archetypes handles individual page anatomy and designing-landing-pages handles the entry point, this skill addresses the full journey: how pages connect, complexity progression, persona-based branching, milestone placement, and the handoff patterns between page types.

This skill handles the sequencing and flow design across multiple pages. Individual page content structure is handled by scaffolding-page-archetypes. Landing page entry points are handled by designing-landing-pages.

## Time-to-First-Success (TTFS) Framework

TTFS is the primary metric for onboarding quality. Every design decision optimizes for minimizing TTFS while ensuring the success is meaningful.

### Defining First Success

| Product Type | First Success | NOT First Success |
|-------------|---------------|-------------------|
| API | Successful authenticated API call returning real data | Viewing API key in dashboard |
| SDK/Library | Running a working code example that uses core functionality | Installing the package |
| Platform | Deploying a working instance that serves traffic | Creating an account |
| CLI Tool | Executing a command that produces useful output | Completing `--help` |
| Database | Writing and reading back data | Connecting to the server |

**Rule:** First success must involve the product doing its core job, not just setup.

### TTFS Targets

| Complexity Tier | Target TTFS | Max Steps to First Success |
|----------------|-------------|---------------------------|
| Simple (CLI tool, small library) | < 5 minutes | 5 steps |
| Medium (API, SDK) | < 15 minutes | 10 steps |
| Complex (Platform, infrastructure) | < 30 minutes | 15 steps |

## Journey Architecture

### The Onboarding Spine

Every onboarding journey follows a spine — a linear sequence of milestones that the developer must reach:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  MILESTONE 0 │───▶│  MILESTONE 1 │───▶│  MILESTONE 2 │───▶│  MILESTONE 3 │
│  Environment │    │ First Success│    │  Core Loop   │    │  Production  │
│  Ready       │    │  (TTFS)      │    │  Mastered    │    │  Ready       │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
     │                   │                   │                   │
   Install            Hello World         Real workflow       Deploy, auth,
   Configure          Working example     Error handling      monitoring
   Authenticate       "It works!"         Customization       Scaling
```

### Milestone Definitions

| Milestone | Exit Criteria | Page Type | Emotional State |
|-----------|--------------|-----------|-----------------|
| M0: Environment Ready | Product installed, configured, authenticated | Quickstart (setup section) | Cautious optimism |
| M1: First Success | Core functionality demonstrated with real output | Quickstart (hello-world section) | Excitement — "it works!" |
| M2: Core Loop Mastered | Developer can perform the primary workflow independently | How-To guides (2-3 guides) | Confidence — "I can use this" |
| M3: Production Ready | App deployed with auth, error handling, monitoring | How-To guides + Reference | Competence — "I can ship this" |

### Progressive Complexity Curve

Each page in the journey adds exactly one new concept. The complexity curve should feel like a gentle ramp, not a staircase.

```
Complexity
    ▲
    │                                    ┌── M3: Production
    │                              ┌─────┘
    │                        ┌─────┘
    │                  ┌─────┘ M2: Core Loop
    │            ┌─────┘
    │      ┌─────┘
    │ ┌────┘ M1: First Success
    │─┘
    │ M0: Setup
    └──────────────────────────────────────▶ Pages
```

**Complexity Budget Per Page:**

| Element | Budget |
|---------|--------|
| New concepts introduced | 1 per page |
| New API calls/methods | 1-2 per page |
| New configuration | 1 setting per page |
| Prerequisites beyond previous page | 0 (each page builds only on previous) |

## Persona-Based Path Branching

### Branch Point Design

After M1 (First Success), onboarding paths diverge based on developer persona:

```
                    ┌─────────────────┐
                    │  M1: First      │
                    │  Success         │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │ Backend  │  │ Frontend │  │ Full-Stack│
        │ Developer│  │ Developer│  │ Developer │
        └────┬─────┘  └────┬─────┘  └────┬─────┘
             │              │              │
        [API guides]  [SDK guides]   [Platform   ]
        [Webhooks  ]  [Components]   [  guides   ]
        [CLI tools ]  [Styling   ]   [Integration]
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                    ┌─────────────────┐
                    │  M3: Production │
                    │  Ready          │
                    └─────────────────┘
```

### Branch Point Implementation

| Element | Requirement |
|---------|------------|
| Placement | Immediately after M1 (First Success), not before |
| Presentation | Self-selection cards with persona descriptions |
| Card count | 2-4 paths (Hick's Law) |
| Labels | Describe what the developer wants to build, not who they are |
| Default path | Highlight the most common path as "Recommended" |
| Convergence | All paths converge at M3 (Production Ready) |

**Branch Card Template:**

```markdown
## Choose Your Path

You've completed the quickstart. Choose the guide that matches what you're building:

- **Build an API Integration** — Connect your backend to [Product] with REST or GraphQL
- **Add [Product] to Your Frontend** — Drop-in components for React, Vue, or Svelte
- **Deploy a Full Platform** — Set up [Product] as your complete [domain] solution
```

## Page Sequencing Rules

### Sequencing Principles

| Principle | Rule | Anti-Pattern |
|-----------|------|--------------|
| Linear prerequisite chain | Each page lists only the previous page as prerequisite | Page requires 3 prior pages from different sections |
| One new concept per page | Introduce exactly one new idea | Page introduces auth AND webhooks AND error handling |
| Build on previous output | Each page starts with the artifact from the previous page | Page starts from scratch with new setup |
| Show before explain | Demonstrate the feature working before explaining how | Two pages of concepts before any code |
| Verify before advance | Each page ends with verification before linking to next | "Continue to next guide" with no verification |

### Handoff Patterns Between Page Types

| From | To | Handoff Mechanism |
|------|-----|-------------------|
| Quickstart → How-To | "You've built X. Now learn to customize it." | Link at end of quickstart verification section |
| How-To → How-To | "Your [artifact] now has [feature]. Next, add [feature]." | Link in Next Steps, building on same artifact |
| How-To → Reference | "For all available options, see the [Resource] reference." | Inline link when mentioning configurable parameters |
| How-To → Explanation | "To understand why [feature] works this way, see [Concept]." | Callout box for curious readers, not required to proceed |
| Tutorial → Branch Point | "Choose your path based on what you're building." | Self-selection cards |

### Anti-Patterns in Sequencing

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Concept dump before action | Developer reads 3 pages before writing code | Move concepts to callout boxes; lead with action |
| Setup marathon | 5 pages of installation/configuration before any feature | Minimize setup to one page; defer optional config |
| Reference as onboarding | Pointing new users to API reference instead of guides | Create a quickstart that uses the API; link to reference from there |
| Dead-end pages | Guide ends without Next Steps | Every page must link forward to the next logical page |
| Circular dependencies | Guide A requires Guide B which requires Guide A | Map prerequisite graph; ensure it's a DAG |

## Milestone Placement

### Milestone Signals

Each milestone must include visible confirmation that the developer has reached it:

| Milestone | Signal Type | Example |
|-----------|------------|---------|
| M0: Environment Ready | Verification command | `$ product --version` outputs `v2.1.0` |
| M1: First Success | Output display | API response JSON showing real data |
| M2: Core Loop | Working artifact | "Your app now handles [workflow]" |
| M3: Production Ready | Deployment confirmation | "Your app is live at [URL]" |

### Progress Indicators

| Pattern | Implementation | When to Use |
|---------|---------------|-------------|
| Step counter | "Step 3 of 5" in page header | Within a single quickstart |
| Journey progress | "Quickstart > Guides > Production" breadcrumb | Across multi-page journey |
| Milestone badge | Checkmark icon with milestone name | At verification sections |
| Time estimate | "~5 minutes remaining" per page | Every page in the journey |

## Journey Map Template

```markdown
## Onboarding Journey: {Product Name}

### Overview
- **Target TTFS**: {minutes}
- **Total pages**: {count}
- **Estimated total time**: {minutes}

### Spine

| # | Page | Type | Milestone | New Concept | Time | Prerequisites |
|---|------|------|-----------|-------------|------|---------------|
| 1 | Install & Configure | Quickstart | M0 | Setup | 3 min | None |
| 2 | Your First [Action] | Quickstart | M1 (TTFS) | Core API call | 5 min | Page 1 |
| 3 | [Branch Point] | Routing | — | — | 1 min | Page 2 |
| 4a | Build [Path A] | How-To | — | [Concept A] | 10 min | Page 2 |
| 4b | Build [Path B] | How-To | — | [Concept B] | 10 min | Page 2 |
| 5 | [Core Workflow] | How-To | M2 | [Concept] | 10 min | Page 4a or 4b |
| 6 | Deploy to Production | How-To | M3 | Deploy config | 15 min | Page 5 |

### Persona Paths

| Persona | Path | Pages | Total Time |
|---------|------|-------|------------|
| {Persona A} | 1 → 2 → 3 → 4a → 5 → 6 | 6 | 44 min |
| {Persona B} | 1 → 2 → 3 → 4b → 5 → 6 | 6 | 44 min |

### Prerequisite Graph

Page 1 → Page 2 → Page 3 → Page 4a ──┐
                          └─→ Page 4b ──┼─→ Page 5 → Page 6
```

## Audit Checklist

### TTFS Optimization
- [ ] First Success is defined as the product doing its core job (not just setup)
- [ ] TTFS target is set appropriate to product complexity tier
- [ ] Steps to first success are within budget (5/10/15 by tier)
- [ ] No concepts-only pages before M1

### Journey Structure
- [ ] Onboarding spine has exactly 4 milestones (M0-M3)
- [ ] Each page introduces exactly 1 new concept
- [ ] Each page builds on the output of the previous page
- [ ] Each page ends with verification before linking to next
- [ ] No circular dependencies in prerequisite graph
- [ ] No dead-end pages (every page has Next Steps)

### Progressive Complexity
- [ ] Complexity curve is a gentle ramp, not a staircase
- [ ] No page requires more than 1-2 new API calls/methods
- [ ] Setup is minimized to 1 page before first success
- [ ] Optional configuration is deferred past M1

### Persona Branching
- [ ] Branch point appears after M1, not before
- [ ] 2-4 branch paths (not more)
- [ ] Branch labels describe what to build, not who the user is
- [ ] Most common path is highlighted as recommended
- [ ] All paths converge at M3

### Handoff Quality
- [ ] Quickstart → How-To handoffs reference the artifact built
- [ ] How-To → How-To handoffs build on the same artifact
- [ ] Reference links appear inline when parameters are mentioned
- [ ] Explanation links appear in callout boxes, not required to proceed
- [ ] Time estimates appear on every page

## Output Format

### Onboarding Journey Design: `{product or scope}`

**TTFS Target**: {minutes} | **Complexity Tier**: {Simple/Medium/Complex}

**Journey Spine**:
| Milestone | Page(s) | Exit Criteria | Time |
|-----------|---------|---------------|------|
| M0 | {page} | {criteria} | {min} |
| M1 | {page} | {criteria} | {min} |
| M2 | {pages} | {criteria} | {min} |
| M3 | {pages} | {criteria} | {min} |

**Persona Paths**: {count} paths from branch point

**Issues Found**:
- {issue and recommendation}

**Proposed Sequence**:
```
{Journey map with page sequence and handoff patterns}
```

## Guidelines

- **DO**: Define First Success before designing any pages — it's the anchor for the entire journey
- **DO**: Keep the path from M0 to M1 as short as possible — front-load the dopamine hit
- **DO**: Use verification sections as milestone signals — developers need confirmation they're on track
- **DO**: Design branch points as self-selection cards with build-oriented labels
- **DON'T**: Front-load concepts — lead with action, explain as needed
- **DON'T**: Branch before M1 — every developer needs the same First Success
- **DON'T**: Create dead-end pages — every page must link forward
- **DON'T**: Skip time estimates — they set expectations and reduce abandonment

## Related Components

- **scaffolding-page-archetypes skill**: Defines the structural anatomy of individual page types this skill sequences
- **designing-landing-pages skill**: Handles the landing page entry point; Getting Started Block previews the journey this skill designs
- **auditing-docs-metrics skill**: TTFS is a key metric this skill optimizes for
- **governing-docs-lifecycle skill**: Pages in the journey follow lifecycle states defined there
- **docs-designer agent**: Ensures individual pages in the journey meet quality standards
- **page-scaffolder agent**: Generates individual pages that this skill sequences into a journey
