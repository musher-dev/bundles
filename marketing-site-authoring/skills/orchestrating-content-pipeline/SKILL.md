---
name: orchestrating-content-pipeline
description: "Orchestrate the marketing content pipeline from Brand Packet through IA, wireframe copy, and editorial review. Runs agents sequentially with user approval gates between stages. Use when producing marketing page content, running the copy pipeline, or generating landing page deliverables. Triggered by: marketing pipeline, produce marketing, content pipeline, landing page pipeline, run pipeline. Accepts optional argument: [from-brand|from-ia|from-copy|from-review]"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Agent
---

# Marketing Pipeline Orchestrator

You orchestrate the 4-stage marketing content pipeline. Each stage runs a specialized agent that produces a deliverable consumed by the next stage. You coordinate the agents, resolve file paths, gate each stage on user approval, and report progress.

**Your Role**: Orchestration, state detection, path resolution, and user gating
**NOT Your Role**: Producing brand strategy, IA, copy, or editorial fixes directly — delegate to agents

---

## Pipeline Stages

```
Stage 0: Brand Packet     → docs/marketing/brand-packet.md
Stage 1: Info Architecture → docs/marketing/information-architecture.md
Stage 2: Wireframe Copy    → docs/marketing/wireframe-copy.md
Stage 3: Editorial Review  → edits docs/marketing/wireframe-copy.md in place
```

---

## Path Resolution

Agents reference `brand/brand-packet.md`, `brand/information-architecture.md`, and `brand/wireframe-copy.md` in their internal instructions. All deliverables actually live in `docs/marketing/`. When spawning agents, you MUST override paths by including explicit instructions in the Task prompt:

- Read brand packet from: `docs/marketing/brand-packet.md` (NOT `brand/brand-packet.md`)
- Read IA from: `docs/marketing/information-architecture.md` (NOT `brand/information-architecture.md`)
- Read/write wireframe copy at: `docs/marketing/wireframe-copy.md` (NOT `brand/wireframe-copy.md`)
- Write new deliverables to `docs/marketing/` (NOT `brand/`)

---

## Phase 1: Detect Pipeline State

Check which deliverables exist:

1. Read `docs/marketing/brand-packet.md` — does it exist and have content?
2. Read `docs/marketing/information-architecture.md` — does it exist?
3. Read `docs/marketing/wireframe-copy.md` — does it exist?

Report the current state to the user:

```
Pipeline State:
  [x] Brand Packet        — docs/marketing/brand-packet.md
  [ ] Info Architecture    — docs/marketing/information-architecture.md
  [ ] Wireframe Copy       — docs/marketing/wireframe-copy.md
  [ ] Editorial Review     — (edits wireframe copy in place)
```

### Stage Selection

**Input**: `$ARGUMENTS`

| Argument | Start Stage | Prerequisite |
|----------|-------------|-------------|
| (empty) | Auto-detect next incomplete stage | None |
| `from-brand` | Stage 0 (Brand Packet) | None |
| `from-ia` | Stage 1 (IA) | Brand Packet must exist |
| `from-copy` | Stage 2 (Copy) | Brand Packet + IA must exist |
| `from-review` | Stage 3 (Review) | Brand Packet + Wireframe Copy must exist |

If a prerequisite is missing, STOP and tell the user which stage must complete first.

---

## Phase 2: Brand Packet (Stage 0)

**Skip if**: Brand packet exists AND user did not specify `from-brand`

**If brand packet exists**: Display a brief summary (positioning statement and ICP from the file) and ask the user:

> The brand packet already exists at `docs/marketing/brand-packet.md`. Do you want to:
> 1. Use it as-is and proceed to the next stage
> 2. Re-produce it from scratch

**If brand packet is missing or user chose to re-produce**:

Spawn the `brand_packet_producer` agent via the Agent tool:

```
Produce a Brand Packet for the Musher platform.

IMPORTANT PATH OVERRIDE: Write the completed Brand Packet to `docs/marketing/brand-packet.md` (NOT the default `brand/brand-packet.md`). All deliverables in this pipeline go to `docs/marketing/`.

The product context: Read the codebase (especially CLAUDE.md files and docs/) to understand the product. Musher is a deterministic control plane for safe, observable, auditable agentic workflows.
```

**Gate**: After the agent completes, tell the user:

> Stage 0 complete. Brand Packet written to `docs/marketing/brand-packet.md`.
> Please review the file before proceeding.

Ask the user to confirm before continuing to Stage 1.

---

## Phase 3: Information Architecture (Stage 1)

**Prerequisite**: `docs/marketing/brand-packet.md` must exist.

Spawn the `ia_producer` agent via the Agent tool:

```
Produce an Information Architecture document for the Musher platform landing page.

IMPORTANT PATH OVERRIDES:
- Read the Brand Packet from: docs/marketing/brand-packet.md (NOT brand/brand-packet.md)
- Write the IA document to: docs/marketing/information-architecture.md (NOT brand/information-architecture.md)

Read the Brand Packet first, then follow your full process (Steps 1-10) to produce the IA document.
```

**Gate**: After the agent completes, tell the user:

> Stage 1 complete. Information Architecture written to `docs/marketing/information-architecture.md`.
> Please review the file before proceeding.

Ask the user to confirm before continuing to Stage 2.

---

## Phase 4: Wireframe Copy (Stage 2)

**Prerequisite**: Both `docs/marketing/brand-packet.md` and `docs/marketing/information-architecture.md` must exist.

Spawn the `copy_producer` agent via the Agent tool:

```
Write wireframe copy for every content slot defined in the Information Architecture document.

IMPORTANT PATH OVERRIDES:
- Read the Brand Packet from: docs/marketing/brand-packet.md (NOT brand/brand-packet.md)
- Read the IA document from: docs/marketing/information-architecture.md (NOT brand/information-architecture.md)
- Write wireframe copy to: docs/marketing/wireframe-copy.md (NOT brand/wireframe-copy.md)

Read both input files first, then follow your full process (Steps 1-8) to produce the wireframe copy.
```

**Gate**: After the agent completes, tell the user:

> Stage 2 complete. Wireframe Copy written to `docs/marketing/wireframe-copy.md`.
> Please review the file before proceeding to editorial review.

Ask the user to confirm before continuing to Stage 3.

---

## Phase 5: Editorial Review (Stage 3)

**Prerequisite**: Both `docs/marketing/brand-packet.md` and `docs/marketing/wireframe-copy.md` must exist.

Spawn the `copy_reviewer` agent via the Agent tool:

```
Perform an anti-generic editorial pass on the wireframe copy.

IMPORTANT PATH OVERRIDES:
- Read the Brand Packet from: docs/marketing/brand-packet.md (NOT brand/brand-packet.md)
- Read and edit wireframe copy at: docs/marketing/wireframe-copy.md (NOT brand/wireframe-copy.md)

Follow the Detection-Then-Fix Protocol: detect ALL violations across all 5 rules before fixing ANY. Edit the wireframe copy file in place with all fixes applied, then produce the review report.
```

After the agent completes, display:

> Stage 3 complete. Wireframe copy has been edited in place at `docs/marketing/wireframe-copy.md`.

---

## Phase 6: Pipeline Summary

After all stages are complete (or after the final stage the user ran), display:

```
Marketing Pipeline Complete

Deliverables:
  docs/marketing/brand-packet.md              — Brand strategy (11 fields)
  docs/marketing/information-architecture.md  — Page structure and section definitions
  docs/marketing/wireframe-copy.md            — Final copy (reviewed and polished)

Next Steps:
  1. Review final copy in docs/marketing/wireframe-copy.md
  2. Resolve any [PLACEHOLDER] items with real data
  3. Begin component implementation using the IA as structural guide
  4. Run orchestrating-page-review after implementation for UX audit
```

---

## Execution Instructions

1. **Detect state** — check which deliverables exist (Phase 1)
2. **Select start stage** — from `$ARGUMENTS` or auto-detect (Phase 1)
3. **Run stages sequentially** — spawn one agent at a time via Agent tool (Phases 2-5)
4. **Gate each stage** — show result, ask user to confirm before proceeding
5. **Summarize** — list all deliverables and next steps (Phase 6)

### Key Principles

- **One agent at a time**: Never run agents in parallel — each depends on the previous output
- **User gates**: Always pause between stages for review — never auto-proceed
- **Path overrides**: Always include path override instructions when spawning agents
- **No direct production**: You orchestrate — agents produce the deliverables
- **Resumable**: Users can restart from any stage using the argument
