---
name: generating-design-rules
description: "Generate a Claude Code rule file that enforces the marketing site design system (spacing, typography, color tokens, component patterns). Use after pipeline Stages 0-3 are complete. Triggered by: design system rule, design tokens, marketing rule, stage 4. Accepts optional argument: [--force]"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
---

# Design Rules Generator (Stage 4)

You generate a `.claude/rules/marketing-design-system.md` file by extracting live design tokens from the marketing site's `app.css` and voice constraints from the Brand Packet. This rule file auto-loads when Claude Code edits marketing site files, enforcing the design system at authoring time.

**Your Role**: Extract tokens, generate rule file, verify output
**NOT Your Role**: Modifying `app.css`, changing the Brand Packet, or implementing marketing pages

---

## Phase 1: Validate Prerequisites

Check that both source files exist:

1. Read `docs/marketing/brand-packet.md` — must exist with content
2. Read `apps/marketing-site/src/app.css` — must exist with a `@theme` block

If either is missing, STOP and tell the user:

> Missing prerequisite: `[file]`. Run `orchestrating-content-pipeline` to produce the Brand Packet first, or verify `app.css` exists.

Check if `.claude/rules/marketing-design-system.md` already exists:

- **Input**: `$ARGUMENTS`
- If file exists AND `$ARGUMENTS` does NOT contain `--force`: ask the user whether to overwrite or abort
- If file exists AND `$ARGUMENTS` contains `--force`: proceed (overwrite without asking)
- If file does not exist: proceed

---

## Phase 2: Extract Design Tokens

Read `apps/marketing-site/src/app.css` and extract from the `@theme {}` block:

### 2a. Typography Tokens

Extract:
- `--font-family-sans` and `--font-family-mono` values
- All `--text-*` size definitions with their `--line-height` and `--letter-spacing` companions
- Note which sizes use `clamp()` (fluid typography)

### 2b. Color Tokens

Extract all token groups:
- `--color-surface-*` (surface palette)
- `--color-border-*` (border hierarchy)
- `--color-text-*` (text hierarchy)
- `--color-signal-*` (signal colors)
- `--color-interactive-*` (interactive states)

### 2c. Spacing Tokens

Extract:
- `--spacing-grid-*` values (the spacing scale)
- `--spacing-icon-*` values

### 2d. Other Tokens

Extract:
- `--radius-*` (border radii)
- `--shadow-*` (box shadows)
- `--transition-*` (timing functions, durations)
- `--opacity-*` (opacity values)

### 2e. Component Classes

Scan the `@layer components` block for defined classes:
- `.btn`, `.btn-primary`, `.btn-secondary`, `.btn-sm`, `.btn-lg`
- `.card`, `.glass`, `.card-hover`, `.card-link`, `.showcase-card`
- `.badge`, `.badge-muted`, `.badge-success`
- `.bento-grid`, `.bento-item`, `.bento-large`, `.bento-tall`
- `.terminal-interactive`, `.terminal-line`, `.terminal-cursor`
- `.gradient-text`, `.gradient-text-action`
- `.input`
- `.reveal`, `.reveal-left`, `.reveal-right`, `.reveal-scale`
- `.exec-line`, `.exec-node`, `.pulse-dot`
- `.run-button`

### 2f. Brand Voice Constraints

Read `docs/marketing/brand-packet.md` and extract:
- **Field 7: Banned Vocabulary** — both default banned and product-specific banned word tables
- **Field 5: Vibe** — the 3 adjectives and archetype blend
- **Field 6: Anti-Vibe** — the 3 boundary statements

---

## Phase 3: Generate Rule File

Write `.claude/rules/marketing-design-system.md` using the extracted tokens. The rule file MUST follow these conventions:

- YAML frontmatter with `paths:` scoped to marketing site files only
- H2 sections for each concern area
- MUST/MUST NOT tone consistent with other `.claude/rules/` files
- Checklists where appropriate
- Actual token values from Phase 2 (not hardcoded guesses)

### Rule File Template

Write the following content to `.claude/rules/marketing-design-system.md`, replacing all `{extracted_*}` placeholders with actual values from Phase 2:

````markdown
---
paths:
  - "apps/marketing-site/**/*.svelte"
  - "apps/marketing-site/**/*.ts"
  - "apps/marketing-site/**/*.css"
---

# Marketing Site Design System

These rules enforce the Musher marketing site's design system. All values are extracted from `apps/marketing-site/src/app.css` and `docs/marketing/brand-packet.md`.

## Spatial System

**Base grid**: 4px. All spacing MUST come from the project's spacing scale.

**Spacing scale** (from `@theme`):
{extracted spacing scale formatted as: `--spacing-grid-1: 4px` through all values}

**Section padding**: Use `py-24` (96px) or `py-32` (128px) for major section vertical spacing. Use `py-12` (48px) or `py-16` (64px) for sub-sections.

**Grid gaps**: Use `gap-4` (16px), `gap-6` (24px), or `gap-8` (32px) for grid layouts.

**Max content width**: 1200px. Use a centered container with `max-w-[1200px] mx-auto`.

**Side padding**: `px-6` (24px) on mobile, `px-10` (40px) on tablet, `px-16` (64px) on desktop.

**MUST NOT** use arbitrary spacing values (e.g., `p-[37px]`, `gap-[55px]`). All spacing must come from the scale above or Tailwind's default 4px-based scale.

## Typography

**Font families** (from `@theme`):
- Sans: `{extracted --font-family-sans value}`
- Mono: `{extracted --font-family-mono value}`

**Type scale** (from `@theme`):
{extracted type scale as a table: name | size | line-height | letter-spacing}

**MUST** use `tracking-tight` (or the extracted letter-spacing) on `text-display-lg`, `text-display`, and `text-title` headings.

**MUST** use `font-bold` (700) for display/hero headings, `font-semibold` (600) for section headings, `font-normal` (400) for body text.

**MUST** use `font-mono` for code blocks, CLI output, and technical identifiers.

**MUST NOT** use arbitrary font sizes (e.g., `text-[17px]`). Use the defined scale.

## Color Tokens

All colors MUST use CSS custom properties via Tailwind classes. Never use raw hex values.

**Surfaces**:
{extracted surface colors as: `bg-surface-01` (#hex) — role}

**Text hierarchy**:
{extracted text colors as: `text-text-primary` (#hex) — role}

**Borders**:
{extracted border colors}

**Signal colors**:
{extracted signal colors as: `text-signal-action` (#hex) — role}

**Interactive states**:
{extracted interactive colors}

**MUST NOT** use raw hex colors (e.g., `bg-[#161616]`). Use token classes: `bg-surface-01`, `text-text-primary`, `border-border-subtle`.

**MUST NOT** use Tailwind's default color palette (e.g., `bg-gray-900`, `text-blue-500`). Use project tokens exclusively.

## Component Library

Reuse existing component classes defined in `app.css` before creating new styles:

**Buttons**: `.btn`, `.btn-primary`, `.btn-secondary`, `.btn-sm`, `.btn-lg`
**Cards**: `.card`, `.glass`, `.card-hover`, `.card-link`, `.showcase-card`
**Badges**: `.badge`, `.badge-muted`, `.badge-success`
**Layout**: `.bento-grid`, `.bento-item`, `.bento-large`, `.bento-tall`
**Terminal**: `.terminal-interactive`, `.terminal-line`, `.terminal-cursor`
**Text effects**: `.gradient-text`, `.gradient-text-action`
**Inputs**: `.input`
**Animations**: `.reveal`, `.reveal-left`, `.reveal-right`, `.reveal-scale`
**Execution paths**: `.exec-line`, `.exec-node`, `.pulse-dot`, `.run-button`

**MUST** check `app.css` for an existing class before writing custom styles.

**MUST** use `.btn-primary` for primary CTAs, `.btn-secondary` for secondary actions.

**MUST** use `.card` or `.glass` for elevated content panels — do not recreate card styles inline.

## Depth & Motion

**Shadow tokens** (from `@theme`):
{extracted shadow tokens}

**Transitions**:
- Timing function: `{extracted --transition-timing-function-cockpit}` — use for all hover/interaction transitions
- Fast: `{extracted --transition-duration-fast}` — hover states
- Normal: `{extracted --transition-duration-normal}` — dropdowns, panels
- Slow: `{extracted --transition-duration-slow}` — modals, page transitions

**MUST** use `duration-fast` for hover states, `duration-normal` for panel transitions, `duration-slow` for page transitions.

**MUST** respect `prefers-reduced-motion` — all animations in `app.css` already handle this.

## Responsive

**Breakpoints**: mobile (<640px), tablet (640-1024px), desktop (1024px+), wide (1200px+).

**Fluid typography**: `text-display-lg`, `text-display`, and `text-title` already use `clamp()` — no additional responsive overrides needed for these.

**Touch targets**: All interactive elements MUST be at least 44px on mobile.

**Mobile padding**: `px-6` (24px). Tablet: `px-10` (40px). Desktop: `px-16` (64px).

**MUST** test layouts at 375px, 768px, and 1280px widths.

## Framework Constraints

This is a **SvelteKit 2.x + Svelte 5** project. NOT React.

**MUST** use Svelte 5 runes:
- `$state()` for reactive state — NOT `useState` or `let` reactivity
- `$derived()` for computed values — NOT `useMemo` or `$:`
- `$effect()` for side effects — NOT `useEffect` or `$:`
- `$props()` for component props — NOT `export let`

**MUST** use `class` attribute — NOT `className`
**MUST** use `onclick` — NOT `on:click`
**MUST** use `{#if}` / `{#each}` — NOT JSX conditionals
**MUST** use `{@html}` sparingly and only for trusted content

**MUST NOT** import from React, Vue, or other frameworks.

## Copy Voice

**Archetype**: Technical Founder (precise, direct, understated) + Challenger Brand (opinionated, concise).

**Vibe**: Candid, Capable, Deliberate.

**Banned vocabulary** — MUST NOT use these words in any marketing copy:
{extracted banned vocabulary as a single comma-separated list}

**Replacement strategy**: Instead of banned words, name the specific improvement, capability, or experience. Use concrete metrics where available.

**Proof requirements**: Every claim MUST be backed by architecture proof or a specific metric. No unverifiable superlatives.

## Anti-Patterns

| Forbidden | Correct | Why |
|-----------|---------|-----|
| `bg-[#161616]` | `bg-surface-01` | Use design tokens, not raw hex |
| `bg-gray-900` | `bg-surface-01` | Use project tokens, not Tailwind defaults |
| `text-[17px]` | `text-body` | Use the defined type scale |
| `p-[37px]` | `p-8` or `p-10` | Use the spacing scale |
| `className` | `class` | Svelte uses `class`, not `className` |
| `useState()` | `$state()` | Svelte 5 runes, not React hooks |
| `on:click` | `onclick` | Svelte 5 event syntax |
| `export let prop` | `let { prop } = $props()` | Svelte 5 props syntax |
| Custom card styles | `.card` or `.glass` | Reuse existing component classes |
| `transition-all` without duration | `duration-fast transition-all` | Always specify duration token |
| "seamless" / "unleash" | Specific description | Banned vocabulary |
````

---

## Phase 4: Verify

After writing the rule file:

1. Read `.claude/rules/marketing-design-system.md` back and confirm:
   - YAML frontmatter has correct `paths:` globs
   - All `{extracted_*}` placeholders are replaced with real values
   - No raw hex values appear outside of comment/reference contexts
   - Banned vocabulary list is complete (20 words from brand packet)
2. Glob for `apps/marketing-site/src/**/*.svelte` to confirm the paths pattern would match actual files

---

## Phase 5: Output Summary

Display to the user:

```
Design System Rule Generated

  File: .claude/rules/marketing-design-system.md
  Scope: apps/marketing-site/**/*.{svelte,ts,css}

  Extracted from:
    apps/marketing-site/src/app.css    — [N] color tokens, [N] spacing tokens, [N] type scale entries, [N] component classes
    docs/marketing/brand-packet.md     — [N] banned words, 3 vibe adjectives, archetype blend

  Rule sections:
    1. Spatial System       — 4px grid, spacing scale, section padding
    2. Typography           — Inter + JetBrains Mono, type scale, weight hierarchy
    3. Color Tokens         — Surface, text, border, signal, interactive palettes
    4. Component Library    — Buttons, cards, badges, bento grid, terminal, animations
    5. Depth & Motion       — Shadow tokens, cockpit easing, duration scale
    6. Responsive           — Breakpoints, fluid type, touch targets, mobile padding
    7. Framework Constraints — Svelte 5 runes, SvelteKit conventions
    8. Copy Voice           — Banned vocabulary, vibe/anti-vibe, proof requirements
    9. Anti-Patterns        — Forbidden → correct pattern table

  Next steps:
    1. The rule auto-loads when editing any file in apps/marketing-site/
    2. Begin implementing marketing pages — the rule enforces design system compliance
    3. Run orchestrating-page-review after implementation for UX audit
```
