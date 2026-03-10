---
name: orchestrating-site-implementation
description: "Orchestrate the marketing site implementation pipeline: build section components, add animations, assemble page, and audit for design compliance. Supports full build, audit-only, and targeted refactor modes. Use when implementing the landing page, building sections, running a design audit, or refactoring specific sections. Triggered by: marketing site, implement site, build site, site audit, refactor section, implement landing page, stage 5. Accepts optional argument: [implement|audit|refactor <section-name>...]"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Agent
---

# Marketing Site Orchestrator

You orchestrate the 4-agent marketing site implementation pipeline. Each agent runs sequentially to produce the landing page from content deliverables. You coordinate agents, resolve file paths, gate each stage on user approval, and report progress.

**Your Role**: Orchestration, state detection, path resolution, and user gating
**NOT Your Role**: Building components, adding animations, assembling pages, or auditing directly — delegate to agents

---

## Pipeline Agents

```
Agent 1: section_builder      → 7 Svelte 5 section components
Agent 2: animation_engineer   → scroll animations on those components
Agent 3: page_assembler       → +page.svelte with SEO meta
Agent 4: design_auditor       → read-only compliance report
```

## Modes

| Argument | Mode | Agents | Description |
|----------|------|--------|-------------|
| (empty) | Auto-detect | Depends on state | Scans filesystem, picks appropriate mode |
| `implement` | Full Build | All 4 sequential | Build → animate → assemble → audit |
| `audit` | Audit Only | Agent 4 only | Read-only compliance scan |
| `refactor <section>...` | Targeted Refactor | Agents 1, 2, 4 | Rebuild specific section(s), re-animate, re-audit |

---

## Phase 1: Validate Input

Parse `$ARGUMENTS` to determine the requested mode.

**If empty**: Mode will be auto-detected in Phase 3.

**If `implement`**: Set mode to `implement`.

**If `audit`**: Set mode to `audit`.

**If `refactor` followed by section name(s)**: Set mode to `refactor` and capture the section name(s).

Valid section names (case-insensitive, match against these):

| Short Name | Component File | IA Section |
|------------|---------------|------------|
| `category-stake` | `CategoryStake.svelte` | 1: Category Stake |
| `credibility-bar` | `CredibilityBar.svelte` | 2: Credibility Signal Bar |
| `mechanism` | `MechanismSection.svelte` | 3: How the Control Plane Works |
| `value-grid` | `ValueGrid.svelte` | 4: What You Control |
| `proof-block` | `ProofBlock.svelte` | 5: Built by Infrastructure Engineers |
| `community` | `CommunityBlock.svelte` | 6: Join the Build |
| `cta` | `CtaBlock.svelte` | 7: Start Controlling Your Agents |

If `refactor` is specified without section names, STOP and ask the user which section(s) to refactor, listing the valid names above.

If an unrecognized section name is given, STOP and show the valid names.

If an unrecognized argument is given, STOP and show usage:

> **Usage:** `orchestrating-site-implementation [implement|audit|refactor <section-name>...]`
>
> **Examples:**
> - `orchestrating-site-implementation` — auto-detect mode from filesystem state
> - `orchestrating-site-implementation implement` — full build: sections → animations → page → audit
> - `orchestrating-site-implementation audit` — design compliance scan only
> - `orchestrating-site-implementation refactor mechanism` — rebuild mechanism section, re-animate, re-audit
> - `orchestrating-site-implementation refactor value-grid proof-block` — rebuild multiple sections

---

## Phase 2: Detect State

Check which deliverables and implementation files exist.

### Content Prerequisites

Check these files exist (required for implement/refactor modes):

1. `docs/marketing/brand-packet.md` — Brand Packet
2. `docs/marketing/information-architecture.md` — IA document
3. `docs/marketing/wireframe-copy.md` — Wireframe copy

### Design Rule

4. `.claude/rules/marketing-design-system.md` — Design system rule

### Section Components

Check `apps/marketing-site/src/lib/components/` for each of the 7 section files:

5. `CategoryStake.svelte`
6. `CredibilityBar.svelte`
7. `MechanismSection.svelte`
8. `ValueGrid.svelte`
9. `ProofBlock.svelte`
10. `CommunityBlock.svelte`
11. `CtaBlock.svelte`

### Page Assembly

12. `apps/marketing-site/src/routes/+page.svelte` — check if it imports section components (not just the default SvelteKit page)

### Design Token Source

13. `apps/marketing-site/src/app.css` — must exist (design tokens)

### Report State

Display the state table:

```
Implementation State:
  Content Pipeline:
    [x] Brand Packet           — docs/marketing/brand-packet.md
    [x] Information Architecture — docs/marketing/information-architecture.md
    [x] Wireframe Copy          — docs/marketing/wireframe-copy.md
  Design Rule:
    [ ] Design System Rule      — .claude/rules/marketing-design-system.md
  Section Components (apps/marketing-site/src/lib/components/):
    [ ] CategoryStake.svelte
    [ ] CredibilityBar.svelte
    [ ] MechanismSection.svelte
    [ ] ValueGrid.svelte
    [ ] ProofBlock.svelte
    [ ] CommunityBlock.svelte
    [ ] CtaBlock.svelte
  Page Assembly:
    [ ] +page.svelte (with section imports)
```

Use `[x]` for files that exist and `[ ]` for missing.

### Prerequisite Checks

**For implement mode**: All 3 content files (brand packet, IA, wireframe copy) MUST exist. The design system rule SHOULD exist (warn if missing, suggest running `generating-design-rules` first, but allow proceeding). If any content file is missing, STOP and tell the user which upstream deliverable is needed and suggest running `orchestrating-content-pipeline`.

**For refactor mode**: All 3 content files MUST exist, AND the targeted section component file(s) MUST already exist (you're refactoring, not creating from scratch for the first time — if no sections exist, suggest `implement` mode instead).

**For audit mode**: At least some `.svelte` files must exist in the marketing site. No content prerequisites.

---

## Phase 3: Execution Plan

### Auto-Detection (when no argument provided)

If the user did not specify a mode, auto-detect based on filesystem state:

| Condition | Auto-Selected Mode | Rationale |
|-----------|-------------------|-----------|
| No section components exist | `implement` | Nothing built yet — full build needed |
| Section components exist but `+page.svelte` doesn't import them | `implement` | Build incomplete — run full pipeline |
| Section components AND assembled page exist | `audit` | Implementation exists — run QC |

Report the auto-detected mode:
```
Mode: {mode} (auto-detected: {rationale})
```

Or if user-specified:
```
Mode: {mode} (user-specified)
```

### Show Execution Plan

Display the plan and wait for user confirmation:

**For implement mode:**
```
Execution Plan: Full Build
==============================

  Stage 1: section_builder
    → Creates 7 section components in apps/marketing-site/src/lib/components/
    → User gate after completion

  Stage 2: animation_engineer
    → Adds scroll animations to section components
    → User gate after completion

  Stage 3: page_assembler
    → Composes sections into apps/marketing-site/src/routes/+page.svelte
    → User gate after completion

  Stage 4: design_auditor
    → Read-only compliance scan, produces violation report

  Verification: Type check on marketing site

Proceed?
```

**For audit mode:**
```
Execution Plan: Audit Only
==============================

  Stage 1: design_auditor
    → Read-only compliance scan of apps/marketing-site/src/
    → Produces violation report with severity ratings and fixes

  No verification step (read-only mode).

Proceed?
```

**For refactor mode:**
```
Execution Plan: Targeted Refactor
==============================

  Target section(s): {section-name(s)}
  Target file(s): {ComponentFile.svelte(s)}

  Stage 1: section_builder
    → Rebuilds targeted section(s) only
    → User gate after completion

  Stage 2: animation_engineer
    → Re-animates targeted file(s) only
    → User gate after completion

  Stage 3: design_auditor
    → Full compliance scan (all marketing files)

  Verification: Type check on marketing site

  Note: Page assembler is NOT re-run (page structure unchanged for section refactors).

Proceed?
```

Use AskUserQuestion to get confirmation before proceeding to Phase 4.

---

## Phase 4: Agent Pipeline

### Implement Mode (all 4 agents, sequential)

#### Stage 1: Section Builder

Spawn the `section_builder` agent via the Agent tool:

```
Build all 7 section components for the Musher marketing site landing page.

IMPORTANT PATH CONTEXT:
- Read IA from: /workspace/docs/marketing/information-architecture.md
- Read wireframe copy from: /workspace/docs/marketing/wireframe-copy.md
- Read brand packet from: /workspace/docs/marketing/brand-packet.md
- Read design system rule from: /workspace/.claude/rules/marketing-design-system.md
- Read design tokens from: /workspace/apps/marketing-site/src/app.css
- Read existing components from: /workspace/apps/marketing-site/src/lib/components/
- Write section components to: /workspace/apps/marketing-site/src/lib/components/

Follow your full process (Steps 1-7). Build all 7 sections in IA conviction arc order.
```

**Gate**: After the agent completes, tell the user:

> Stage 1 complete. Section components created in `apps/marketing-site/src/lib/components/`.

List the files created/modified. Ask the user to review and confirm before proceeding.

#### Stage 2: Animation Engineer

Spawn the `animation_engineer` agent via the Agent tool:

```
Add scroll-triggered animations to all section components in the Musher marketing site.

IMPORTANT PATH CONTEXT:
- Section components are in: /workspace/apps/marketing-site/src/lib/components/
- GSAP config is at: /workspace/apps/marketing-site/src/lib/gsap-config.ts
- Design tokens / CSS reveal classes are in: /workspace/apps/marketing-site/src/app.css

Follow your full process (Steps 1-6). Apply the correct animation tier to each section.
```

**Gate**: After the agent completes, tell the user:

> Stage 2 complete. Scroll animations added to section components.

List the animation tiers applied per section. Ask the user to review and confirm before proceeding.

#### Stage 3: Page Assembler

Spawn the `page_assembler` agent via the Agent tool:

```
Compose all 7 section components into the landing page for the Musher marketing site.

IMPORTANT PATH CONTEXT:
- Section components are in: /workspace/apps/marketing-site/src/lib/components/
- Page file: /workspace/apps/marketing-site/src/routes/+page.svelte
- Layout file: /workspace/apps/marketing-site/src/routes/+layout.svelte
- Read wireframe copy from: /workspace/docs/marketing/wireframe-copy.md (for SEO meta derivation)

Follow your full process (Steps 1-6). Import sections in IA conviction arc order, add full SEO meta block.
```

**Gate**: After the agent completes, tell the user:

> Stage 3 complete. Landing page assembled at `apps/marketing-site/src/routes/+page.svelte`.

Show the section import order and SEO meta summary. Ask the user to review and confirm before proceeding.

#### Stage 4: Design Auditor

Spawn the `design_auditor` agent via the Agent tool:

```
Run a full design system compliance audit on the Musher marketing site.

IMPORTANT PATH CONTEXT:
- Audit scope: /workspace/apps/marketing-site/src/
- Design tokens: /workspace/apps/marketing-site/src/app.css
- Design system rule: /workspace/.claude/rules/marketing-design-system.md

Follow your full process (Steps 1-5). Run all 8 audit categories. Produce the full violation report.
```

No gate after the auditor — it is the final stage. Display its report directly.

---

### Audit Mode (agent 4 only)

Spawn the `design_auditor` agent via the Agent tool with the same prompt as Stage 4 above.

Display the violation report directly. Skip verification (read-only).

---

### Refactor Mode (agents 1, 2, 4 — targeted)

#### Stage 1: Section Builder (targeted)

Spawn the `section_builder` agent via the Agent tool:

```
Rebuild the following section component(s) for the Musher marketing site:
{list of targeted section names and their component files}

IMPORTANT PATH CONTEXT:
- Read IA from: /workspace/docs/marketing/information-architecture.md
- Read wireframe copy from: /workspace/docs/marketing/wireframe-copy.md
- Read brand packet from: /workspace/docs/marketing/brand-packet.md
- Read design system rule from: /workspace/.claude/rules/marketing-design-system.md
- Read design tokens from: /workspace/apps/marketing-site/src/app.css
- Read existing components from: /workspace/apps/marketing-site/src/lib/components/
- Write section components to: /workspace/apps/marketing-site/src/lib/components/

IMPORTANT: Only rebuild the targeted section(s) listed above. Do NOT modify other section components.

Follow your full process but scoped to only these section(s). Read all inputs, then rebuild only the targeted component file(s).
```

**Gate**: Show files modified, ask user to confirm.

#### Stage 2: Animation Engineer (targeted)

Spawn the `animation_engineer` agent via the Agent tool:

```
Re-apply scroll animations to the following section component(s) that were just rebuilt:
{list of targeted section names and their component files}

IMPORTANT PATH CONTEXT:
- Section components are in: /workspace/apps/marketing-site/src/lib/components/
- GSAP config is at: /workspace/apps/marketing-site/src/lib/gsap-config.ts
- Design tokens / CSS reveal classes are in: /workspace/apps/marketing-site/src/app.css

IMPORTANT: Only add animations to the targeted file(s) listed above. Do NOT modify other section components.

Follow your full process but scoped to only these section(s). Read the targeted files, apply the correct animation tier, verify.
```

**Gate**: Show animations applied, ask user to confirm.

#### Stage 3: Design Auditor (full scan)

Spawn the `design_auditor` agent with the same prompt as the implement mode Stage 4 (full scan, not targeted — the auditor always scans everything).

Display the violation report directly.

**Note**: The page assembler is NOT re-run in refactor mode. Page structure does not change when individual sections are rebuilt.

---

## Phase 5: Verification

**Skip for audit mode** (read-only, no files changed).

For implement and refactor modes, run type checking on the marketing site:

```bash
cd /workspace && task marketing:check
```

If `task marketing:check` is not available, fall back to:

```bash
cd /workspace/apps/marketing-site && npx svelte-check --tsconfig ./tsconfig.json 2>&1 | head -100
```

Report the results:
- If clean: "Type check passed."
- If errors: List the errors and suggest fixes.

---

## Phase 6: Summary

### Implement Mode Summary

```
Marketing Site Implementation Complete
======================================

  Files Created/Modified:
    apps/marketing-site/src/lib/components/CategoryStake.svelte     — Section 1: Hero
    apps/marketing-site/src/lib/components/CredibilityBar.svelte    — Section 2: Social Proof
    apps/marketing-site/src/lib/components/MechanismSection.svelte  — Section 3: Mechanism
    apps/marketing-site/src/lib/components/ValueGrid.svelte         — Section 4: Value Grid
    apps/marketing-site/src/lib/components/ProofBlock.svelte        — Section 5: Proof
    apps/marketing-site/src/lib/components/CommunityBlock.svelte    — Section 6: Community
    apps/marketing-site/src/lib/components/CtaBlock.svelte          — Section 7: CTA
    apps/marketing-site/src/routes/+page.svelte                     — Assembled page

  Audit Results:
    {summary from design auditor: N violations (N FAIL, N WARN)}

  Type Check:
    {pass or error count}

  Next Steps:
    1. Review the assembled page at apps/marketing-site/src/routes/+page.svelte
    2. Fix any FAIL violations from the audit report
    3. Run `task marketing:dev` to preview locally
    4. Run `orchestrating-site-implementation audit` after fixes to re-check compliance
    5. Run orchestrating-page-review for a UX review
```

### Audit Mode Summary

```
Marketing Site Audit Complete
=============================

  Audit Results:
    {full violation report from design auditor}

  Next Steps:
    1. Fix FAIL violations first (must-fix)
    2. Address WARN violations (should-fix)
    3. Run `orchestrating-site-implementation audit` again after fixes
    4. Run `orchestrating-site-implementation refactor <section>` to rebuild specific sections
```

### Refactor Mode Summary

```
Marketing Site Refactor Complete
================================

  Refactored Section(s): {section-name(s)}

  Files Modified:
    {list of component files rebuilt and re-animated}

  Audit Results:
    {summary from design auditor}

  Type Check:
    {pass or error count}

  Next Steps:
    1. Review the refactored section(s)
    2. Fix any FAIL violations from the audit report
    3. Run `task marketing:dev` to preview locally
    4. Run `orchestrating-site-implementation audit` to re-check full compliance
```

---

## Execution Instructions

1. **Validate input** — parse mode and section names from `$ARGUMENTS` (Phase 1)
2. **Detect state** — check all deliverables and implementation files (Phase 2)
3. **Plan execution** — auto-detect mode if needed, show plan, get confirmation (Phase 3)
4. **Run agents** — spawn one agent at a time via Agent tool, gate between stages (Phase 4)
5. **Verify** — run type checking on the marketing site (Phase 5)
6. **Summarize** — list all deliverables, audit results, and next steps (Phase 6)

### Key Principles

- **One agent at a time**: Never run agents in parallel — each depends on the previous output
- **User gates**: Always pause between stages for review — never auto-proceed
- **Path context**: Always include explicit path context when spawning agents (agents may have default paths that differ from this project's layout)
- **No direct production**: You orchestrate — agents produce the deliverables
- **Scoped refactors**: Refactor mode targets only the specified section(s) — do not rebuild everything
- **Full audits**: The design auditor always scans the full marketing site, even in refactor mode
- **Resumable**: Users can run audit or refactor modes independently at any time
