---
name: designing-typography-layout
version: 1.0.0
description: Define typography and spatial layout standards for developer documentation including line length, leading, baseline grid, whitespace density, heading hierarchy, and density management per page type. Use when setting typography rules, auditing layout spacing, or improving reading comfort. Triggered by: typography, line length, line height, leading, baseline grid, whitespace, spacing, heading hierarchy, reading comfort, layout density, measure, font size, paragraph spacing, content width, typographic scale.
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Designing Typography Layout

## Purpose

Define typography and spatial layout standards that optimize sustained technical reading in developer documentation. This skill provides concrete specifications for line length, leading, grid systems, whitespace distribution, and density management — translating typographic principles into measurable implementation rules.

This is a read-only audit and specification skill. It produces design standards and assesses compliance but does not implement CSS or modify templates (handled by implementation-focused skills). It does not govern navigation layout (handled by designing-navigation-systems).

## Core Principle

Beauty without structure is fashion. Functional typography improves comprehension, reduces fatigue, and increases time-on-page. Every specification below exists because research demonstrates its effect on reading performance, not because it looks better.

## Typography Specifications

### 1. Line Length (Measure)

**Rule:** 45-90 characters per line, targeting 65-75 characters for body text.

| Context | Measure | Rationale |
|---------|---------|-----------|
| Body text | 65-75 characters | Optimal saccade length for sustained reading |
| Code blocks | 80-100 characters | Matches standard terminal width, avoids wrapping |
| Sidebar navigation | 25-35 characters | Short labels, scannable |
| Admonitions/callouts | Same as body | Consistent reading rhythm |

**Implementation:**
- Set `max-width` on content container, not the viewport
- Use `ch` units for character-based measure: `max-width: 75ch` for body
- Code blocks may exceed body measure but must not require horizontal scrolling

**Audit Test:** Select any body paragraph. Count characters in the longest line. Fail if >90 or <45.

### 2. Line Height (Leading)

**Rule:** Minimum 1.5x font size for body text. Heading leading is tighter.

| Element | Line Height | Rationale |
|---------|-------------|-----------|
| Body text | 1.5-1.7 | Comfortable reading at standard font sizes |
| Code blocks | 1.4-1.5 | Denser is acceptable for monospace, aids scanning |
| Headings (H1-H2) | 1.1-1.3 | Tight leading for display text prevents float |
| Headings (H3-H6) | 1.3-1.4 | Slightly more than display, less than body |
| Lists | 1.4-1.5 | Between code and body density |

**Rules:**
- Never set line height below 1.4 for any readable text
- Line height should increase if line length increases (wider measure needs more leading)
- Use unitless values (`line-height: 1.5`), not fixed units (`line-height: 24px`)

### 3. Baseline Grid

**Rule:** Align all vertical spacing to a 4dp or 8dp baseline grid.

**Grid Implementation:**

| Base Unit | Use Case |
|-----------|----------|
| 4dp | Fine-grained control: inline spacing, icon alignment |
| 8dp | Standard spacing: margins, padding, section gaps |

**Spacing Scale (8dp base):**

| Token | Value | Use |
|-------|-------|-----|
| `space-xs` | 4px | Inline element gaps |
| `space-sm` | 8px | Related element spacing, list item gap |
| `space-md` | 16px | Paragraph spacing, card padding |
| `space-lg` | 24px | Section sub-element spacing |
| `space-xl` | 32px | Between major content blocks |
| `space-2xl` | 48px | Section boundaries |
| `space-3xl` | 64px | Page-level section divisions |

**Rules:**
- All vertical spacing values must be multiples of the base unit
- Avoid arbitrary spacing values (17px, 22px, 35px)
- Use the spacing scale tokens, not raw pixel values

### 4. Whitespace Density

**Rule:** Headings get 1.5x more space above than below. Whitespace density varies by page type.

**Heading Spacing Pattern:**

| Element | Space Above | Space Below | Ratio |
|---------|------------|------------|-------|
| H1 | N/A (page top) | `space-xl` (32px) | — |
| H2 | `space-2xl` (48px) | `space-lg` (24px) | 2:1 |
| H3 | `space-xl` (32px) | `space-md` (16px) | 2:1 |
| H4-H6 | `space-lg` (24px) | `space-sm` (8px) | 3:1 |

**Rationale:** More space above headings creates visual separation from preceding content. Less space below creates association with the content the heading introduces.

**Density by Page Type:**

| Page Type | Density | Characteristics |
|-----------|---------|----------------|
| API Reference | Dense | Compact tables, minimal prose, tight spacing between endpoints |
| Quickstart/Tutorial | Airy | Generous spacing between steps, breathing room around code blocks |
| How-To Guide | Medium | Standard spacing, focused on scan-to-step pattern |
| Conceptual/Explanation | Airy | Wide margins, generous paragraph spacing for sustained reading |
| Troubleshooting | Dense | Compact error entries, scan-friendly tables |

### 5. Heading Hierarchy

**Rule:** Sequential heading levels with consistent visual weight reduction.

**Visual Weight Scale:**

| Level | Size Ratio | Weight | Use |
|-------|-----------|--------|-----|
| H1 | 2.0-2.5x body | Bold (700) | Page title only — one per page |
| H2 | 1.5-1.75x body | Bold (700) or Semibold (600) | Major sections |
| H3 | 1.25-1.4x body | Semibold (600) | Subsections |
| H4 | 1.1-1.2x body | Semibold (600) or Medium (500) | Sub-subsections |
| H5-H6 | 1.0x body | Bold (700) at body size | Rare; consider restructuring if needed |

**Rules:**
- Never skip heading levels (H2 → H4)
- One H1 per page (the page title)
- If H5-H6 are needed, the page should probably be split
- Headings must be distinguishable from body text by more than just size — use weight and spacing

### 6. Font Selection

**Rules for documentation sites:**

| Role | Recommendation | Rationale |
|------|---------------|-----------|
| Body | System font stack or high-quality sans-serif | Fast loading, familiar, optimized for screen |
| Code | Monospace with coding ligatures optional | Clear character distinction (Il1, O0) |
| Headings | Same as body or complementary sans-serif | Consistency; avoid decorative faces |

**Character Distinction Test:** In the chosen monospace font, verify these pairs are clearly distinguishable:
- `I` (capital i) vs `l` (lowercase L) vs `1` (one)
- `O` (capital o) vs `0` (zero)
- `rn` vs `m`
- `` ` `` (backtick) vs `'` (apostrophe)

## Audit Checklist

For any documentation page, verify:

- [ ] Body text line length is 45-90 characters
- [ ] Body text line height is ≥1.5
- [ ] Code block line height is ≥1.4
- [ ] All spacing values are multiples of the baseline grid unit
- [ ] Headings have more space above than below (≥1.5x ratio)
- [ ] Heading levels are sequential (no skipped levels)
- [ ] Only one H1 per page
- [ ] Monospace font passes character distinction test
- [ ] Page density matches its archetype (reference=dense, tutorial=airy)
- [ ] No horizontal scrolling required for body or code content

## Output Format

### Typography Audit: `{page or site}`

**Compliance Summary**:
| Specification | Status | Measured Value | Target |
|--------------|--------|---------------|--------|
| Line length | Pass/Fail | [chars] | 45-90 |
| Body leading | Pass/Fail | [value] | ≥1.5 |
| Code leading | Pass/Fail | [value] | ≥1.4 |
| Baseline grid | Pass/Fail | [violations] | 0 |
| Heading spacing ratio | Pass/Fail | [ratio] | ≥1.5:1 above:below |
| Heading sequence | Pass/Fail | [skips] | 0 |

**Issues**:
| Issue | Location | Severity | Fix |
|-------|----------|----------|-----|
| [issue] | [selector/element] | High/Med/Low | [CSS fix] |

**Density Assessment**: [Dense/Medium/Airy] — [appropriate/inappropriate] for [page archetype]

## Guidelines

- **DO**: Measure actual character counts, not estimates
- **DO**: Use unitless line-height values for scalability
- **DO**: Vary density by page type — reference pages should be denser than tutorials
- **DO**: Test with real content, not lorem ipsum
- **DON'T**: Optimize typography for aesthetics over readability
- **DON'T**: Use fixed pixel values for line-height
- **DON'T**: Apply uniform spacing to all page types — density should match content purpose

## Related Components

- **designing-navigation-systems skill**: Navigation layout constraints (sidebar width) affect content area measure
- **auditing-content-quality skill**: Readability dimension references these typography standards
- **designing-code-displays skill**: Code block typography (monospace, line height) governed here
- **scaffolding-page-archetypes skill**: Archetype determines appropriate density level
- **analyzing-docs-benchmarks skill**: Visual & Interactive dimension includes typography assessment
