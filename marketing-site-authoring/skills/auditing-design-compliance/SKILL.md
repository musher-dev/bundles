---
name: auditing-design-compliance
user-invocable: false
description: Audit marketing site components for design system compliance using automated grep-based scans. Use when checking for design token violations, auditing typography, spacing, colors, or framework compliance after implementing sections. Triggered by: design audit, compliance check, design system violations, token audit, color audit, spacing audit, typography audit, Svelte 5 audit, framework compliance, grep violations, marketing QC, quality check, post-implementation audit, design lint, style violations, anti-pattern scan.
allowed-tools: Read, Grep, Glob, Bash
---

# Auditing Marketing Design Compliance

## Purpose

Run a structured, grep-based audit of all marketing site files after implementation. The design system rule (`design-system.md`) enforces tokens passively during editing -- this skill provides an active, post-implementation QC pass that systematically scans for violations.

## Source Documents

- **Design Rule**: `/workspace/.claude/rules/design-system.md` -- the canonical enforcement rules
- **Design Tokens**: `/workspace/apps/marketing-site/src/app.css` -- `@theme` block with all valid tokens
- **Target Files**: `apps/marketing-site/src/**/*.svelte` and `apps/marketing-site/src/**/*.ts`

## Audit Scope

Scan all files matching:
- `apps/marketing-site/src/**/*.svelte`
- `apps/marketing-site/src/**/*.ts`
- `apps/marketing-site/src/**/*.css` (excluding `app.css` `@theme` block itself)

## Audit Categories

### 1. Color Violations

**Raw hex colors outside `@theme`:**

```
Pattern: /#[0-9a-fA-F]{3,8}/
Scope: *.svelte files (outside @theme block)
Excludes: app.css @theme block, comments, SVG fill="none"
```

Grep command:
```bash
grep -rn '#[0-9a-fA-F]\{3,8\}' apps/marketing-site/src/lib/components/ apps/marketing-site/src/routes/ --include='*.svelte'
```

**Tailwind default palette usage:**

```
Pattern: (bg|text|border|ring|from|to|via)-(red|blue|green|yellow|gray|slate|zinc|neutral|stone|orange|amber|emerald|teal|cyan|sky|indigo|violet|purple|fuchsia|pink|rose)-\d+
```

Grep command:
```bash
grep -rn -E '(bg|text|border|ring|from|to|via)-(red|blue|green|yellow|gray|slate|zinc|neutral|stone|orange|amber|emerald|teal|cyan|sky|indigo|violet|purple|fuchsia|pink|rose)-[0-9]+' apps/marketing-site/src/ --include='*.svelte'
```

**Expected zero violations.** Every color must use project tokens: `bg-surface-*`, `text-text-*`, `text-signal-*`, `bg-signal-*`, `border-border-*`, `bg-interactive-*`.

### 2. Spacing Violations

**Arbitrary spacing values:**

```
Pattern: (p|m|gap|space|inset|top|right|bottom|left|w|h|max-w|min-w|max-h|min-h)-\[.*?(px|rem|em)
```

Grep command:
```bash
grep -rn -E '(p|m|gap|space|inset|top|right|bottom|left|w|h)-\[[0-9]+(px|rem|em)\]' apps/marketing-site/src/ --include='*.svelte'
```

**Allowlisted exceptions**: `max-w-[1200px]` (page max-width from design system), specific `clamp()` values in `@theme`.

**Expected**: Only `max-w-[1200px]` should appear. All other spacing must use Tailwind's 4px-based scale or the `@theme` spacing tokens.

### 3. Typography Violations

**Arbitrary font sizes:**

```
Pattern: text-\[\d+px\]
```

Grep command:
```bash
grep -rn 'text-\[' apps/marketing-site/src/ --include='*.svelte'
```

**Expected zero.** Use the defined type scale: `text-label`, `text-data`, `text-body`, `text-body-large`, `text-heading-sm`, `text-heading`, `text-title`, `text-display`, `text-display-lg`.

**Missing tracking on headings:**

Headings using `text-display-lg`, `text-display`, or `text-title` should have `tracking-tight` (the theme defines letter-spacing, but verify it's not overridden).

```bash
grep -rn 'text-display\|text-title' apps/marketing-site/src/ --include='*.svelte' | grep -v 'tracking-tight'
```

Note: The theme CSS already includes letter-spacing for these sizes, so `tracking-tight` is technically redundant but good for explicitness. Flag only if explicit `tracking-normal` or `tracking-wide` is used on display headings.

**Weight hierarchy check:**

- `text-display-lg` and `text-display` should use `font-bold` (700)
- `text-title` and `text-heading` should use `font-semibold` (600)
- `text-body` should use `font-normal` (400)

```bash
grep -rn 'text-display' apps/marketing-site/src/ --include='*.svelte' | grep -v 'font-bold'
```

### 4. Framework Compliance (Svelte 5 Runes)

**React-isms:**

```bash
# className instead of class
grep -rn 'className' apps/marketing-site/src/ --include='*.svelte'

# useState, useEffect, useMemo
grep -rn 'useState\|useEffect\|useMemo\|useCallback\|useRef' apps/marketing-site/src/ --include='*.svelte' --include='*.ts'

# React imports
grep -rn "from 'react'" apps/marketing-site/src/ --include='*.svelte' --include='*.ts'
```

**Svelte 4 syntax:**

```bash
# on:click instead of onclick
grep -rn 'on:click\|on:change\|on:input\|on:submit\|on:keydown\|on:focus\|on:blur\|on:mouseover\|on:mouseenter\|on:mouseleave' apps/marketing-site/src/ --include='*.svelte'

# export let instead of $props()
grep -rn 'export let ' apps/marketing-site/src/ --include='*.svelte'

# $: reactive declarations instead of $derived/$effect
grep -rn '^\s*\$:' apps/marketing-site/src/ --include='*.svelte'
```

**Expected zero violations for all framework checks.**

### 5. Transition Duration Violations

**Missing duration tokens on transitions:**

```bash
# transition-all without a duration class
grep -rn 'transition-all\|transition-colors\|transition-opacity\|transition-transform' apps/marketing-site/src/ --include='*.svelte' | grep -v 'duration-fast\|duration-normal\|duration-slow'
```

Note: Tailwind v4 defaults may handle this, but explicit duration tokens are preferred per the design system.

### 6. Component Reuse Violations

**Inline styles duplicating component classes:**

Check for inline card-like styling that should use `.card`, `.glass`, or `.bento-item`:

```bash
# Patterns that suggest a custom card instead of using .card
grep -rn 'bg-surface-02.*rounded.*border.*p-[46]' apps/marketing-site/src/ --include='*.svelte' | grep -v 'card\|glass\|bento'
```

**Inline button styling that should use `.btn`:**

```bash
# Custom button patterns instead of .btn
grep -rn 'inline-flex.*items-center.*justify-center.*gap.*px.*py' apps/marketing-site/src/ --include='*.svelte' | grep -v 'btn\|run-button'
```

### 7. Accessibility Quick Checks

**Images without alt text:**

```bash
grep -rn '<img' apps/marketing-site/src/ --include='*.svelte' | grep -v 'alt='
```

**Interactive elements without accessible names:**

```bash
# Buttons with only icons (no text, no aria-label)
grep -rn '<button' apps/marketing-site/src/ --include='*.svelte' | grep -v 'aria-label'
```

**Missing reduced-motion handling:**

```bash
# GSAP usage without reduced motion check
grep -rn 'gsap\.' apps/marketing-site/src/ --include='*.svelte' | head -20
# Then verify prefersReducedMotion is checked in the same component
```

### 8. Banned Vocabulary Scan

Scan all user-facing text in Svelte components:

```bash
grep -rni 'supercharge\|unleash\|seamless\|revolutionize\|empower\|robust\|unlock\|leverage\|ai-powered\|intelligent\|cutting-edge\|next-generation\|scalable\|game-changing\|innovative\|effortless\|holistic\|end-to-end\|disruptive\|frictionless' apps/marketing-site/src/ --include='*.svelte'
```

**Expected zero violations.** All copy must follow the Brand Packet voice rules.

## Audit Output Format

Produce a violation table after running all scans:

```markdown
## Design Compliance Audit Results

**Date**: [date]
**Scope**: apps/marketing-site/src/
**Files scanned**: [count]

### Summary

| Category | Violations | Status |
|----------|-----------|--------|
| Color tokens | 0 | PASS |
| Spacing scale | 1 | WARN |
| Typography scale | 0 | PASS |
| Framework (Svelte 5) | 0 | PASS |
| Transition durations | 2 | WARN |
| Component reuse | 0 | PASS |
| Accessibility | 1 | WARN |
| Banned vocabulary | 0 | PASS |

### Violations

| # | File | Line | Category | Violation | Fix |
|---|------|------|----------|-----------|-----|
| 1 | MechanismSection.svelte | 42 | Spacing | `p-[30px]` | Use `p-8` (32px) |
| 2 | ProofBlock.svelte | 18 | Duration | `transition-all` missing duration | Add `duration-fast` |
| 3 | ProofBlock.svelte | 55 | Duration | `transition-colors` missing duration | Add `duration-fast` |
| 4 | CtaBlock.svelte | 23 | Accessibility | `<img>` missing `alt` | Add descriptive alt text |
```

## Running the Full Audit

Execute all grep patterns above in sequence. For each violation found:

1. Record file, line number, category, and the violating pattern
2. Provide the specific fix (which token/class to use instead)
3. Rate severity: **FAIL** (must fix before shipping) or **WARN** (should fix, not blocking)

### Severity Guide

| Category | FAIL conditions | WARN conditions |
|----------|----------------|-----------------|
| Color tokens | Any raw hex or Tailwind default palette in markup | Raw hex in SVG `fill` attributes (may be intentional) |
| Spacing | Arbitrary values not in allowlist | Values close to scale (e.g., `p-[30px]` where `p-8` = 32px) |
| Typography | Arbitrary font sizes | Missing `tracking-tight` on headings (theme handles it) |
| Framework | Any React-ism or Svelte 4 syntax | N/A -- all framework violations are FAIL |
| Duration | Missing duration on interactive transitions | Missing duration on non-interactive transitions |
| Component reuse | N/A (always WARN) | Inline styles duplicating existing classes |
| Accessibility | Missing alt on content images | Missing aria-label on icon-only buttons |
| Banned vocabulary | Any banned word in visible copy | Banned word in code comments |

## Allowlist

These patterns are expected and should not be flagged:

| Pattern | Location | Reason |
|---------|----------|--------|
| `max-w-[1200px]` | Any section container | Design system specifies 1200px max-width |
| Hex values in `app.css` `@theme` block | `app.css` lines 10-183 | Token definitions themselves |
| Hex values in CSS `linear-gradient`, `radial-gradient` | `app.css` | Gradient definitions |
| Hex values in `rgba()` | `app.css` shadow/glow definitions | Shadow token values |
| `style="filter: drop-shadow(...)"` | SVG elements | SVG-specific inline styles |
| `style="--delay: ..."` | Animation stagger delays | CSS custom property for stagger timing |

## Automated Scan Script

For convenience, here's a complete scan sequence to run from the repo root:

```bash
echo "=== COLOR: Raw hex ==="
grep -rn '#[0-9a-fA-F]\{3,8\}' apps/marketing-site/src/lib/components/ apps/marketing-site/src/routes/ --include='*.svelte' | grep -v 'svg\|filter\|drop-shadow\|url(' || echo "PASS"

echo "=== COLOR: Tailwind defaults ==="
grep -rn -E '(bg|text|border|ring)-(red|blue|green|yellow|gray|slate|zinc|neutral|stone)-[0-9]+' apps/marketing-site/src/ --include='*.svelte' || echo "PASS"

echo "=== SPACING: Arbitrary ==="
grep -rn -E '(p|m|gap)-\[[0-9]+(px|rem)\]' apps/marketing-site/src/ --include='*.svelte' | grep -v 'max-w-\[1200px\]' || echo "PASS"

echo "=== TYPOGRAPHY: Arbitrary sizes ==="
grep -rn 'text-\[' apps/marketing-site/src/ --include='*.svelte' || echo "PASS"

echo "=== FRAMEWORK: React ==="
grep -rn 'className\|useState\|useEffect' apps/marketing-site/src/ --include='*.svelte' || echo "PASS"

echo "=== FRAMEWORK: Svelte 4 ==="
grep -rn 'on:click\|on:change\|export let ' apps/marketing-site/src/ --include='*.svelte' || echo "PASS"

echo "=== VOCABULARY: Banned words ==="
grep -rni 'supercharge\|unleash\|seamless\|revolutionize\|empower\|robust\|unlock\|leverage\|ai-powered\|cutting-edge\|next-generation\|scalable\|game-changing\|innovative\|effortless\|holistic\|end-to-end\|disruptive\|frictionless' apps/marketing-site/src/ --include='*.svelte' || echo "PASS"
```

## Related Skills

- **`design-system.md` rule** -- the passive enforcement rule that prevents violations during editing
- **`scaffolding-sections`** -- use correct tokens from the start
- **`auditing-page-velocity`** -- performance audit (separate concern from design compliance)
- **`auditing-trust-engineering`** -- trust signal audit
