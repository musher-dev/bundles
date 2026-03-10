---
name: designing-navigation-systems
version: 1.0.0
description: Design and audit developer documentation navigation systems including three-pane layouts, sidebar governance, breadcrumbs, contextual navigation switching, and depth limits. Use when structuring doc site navigation, reviewing sidebar organization, designing breadcrumb trails, or applying Hick's Law to navigation decisions. Triggered by: navigation, sidebar, breadcrumbs, three-pane, nav design, docs navigation, sidebar governance, nav depth, table of contents, contextual nav, docs structure, information architecture, nav switching, menu design.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Designing Documentation Navigation Systems

## Purpose

Design and audit the navigation architecture of a developer documentation site. Navigation is the skeleton of the docs — poor navigation makes even excellent content unfindable. This skill covers layout patterns, sidebar governance, breadcrumbs, and contextual navigation switching.

This skill handles structural navigation. Individual page structure is handled by docs-designer. Taxonomy decisions (how to categorize content) are handled by governing-taxonomy.

## Three-Pane Layout

The standard developer documentation layout uses three panes:

```
┌──────────────────────────────────────────────────────────┐
│  Top Bar: Product name, version selector, search, links  │
├────────────┬────────────────────────┬────────────────────┤
│            │                        │                    │
│  LEFT      │  CENTER                │  RIGHT             │
│  SIDEBAR   │  CONTENT               │  TABLE OF          │
│            │                        │  CONTENTS           │
│  Section   │  The actual page       │  On-page headings  │
│  navigation│  content               │  (auto-generated)  │
│            │                        │                    │
│  - Group 1 │                        │  - Section 1       │
│    - Page  │                        │  - Section 2       │
│    - Page  │                        │    - Sub 2.1       │
│  - Group 2 │                        │  - Section 3       │
│    - Page  │                        │                    │
│            │                        │                    │
├────────────┴────────────────────────┴────────────────────┤
│  Footer: Links, status, version info                     │
└──────────────────────────────────────────────────────────┘
```

### Pane Responsibilities

| Pane | Content | Width | Behavior |
|------|---------|-------|----------|
| Left Sidebar | Section-level navigation (pages within a docs area) | 240-280px | Sticky, scrollable independently, collapsible on mobile |
| Center Content | Page content, the thing the user came to read | Flexible, max 720-800px | Main scroll target |
| Right TOC | On-page heading navigation (auto-generated from H2/H3) | 200-240px | Sticky, highlights current section, hidden on mobile |

### Layout Rules

| Rule | Rationale |
|------|-----------|
| Left sidebar shows only the current section, not entire site | Prevents information overload |
| Right TOC auto-generates from page headings | Single source of truth, always accurate |
| Content area has a max-width (720-800px) | Optimal reading line length (60-80 characters) |
| All three panes are independently scrollable | User doesn't lose nav context when reading |

## Sidebar Governance

### The 20-Item Rule

**A sidebar group must never show more than 20 visible items (expanded) at once.** Beyond 20 items, scanning time increases non-linearly (Hick's Law: decision time = log2(n)).

| Items Visible | User Experience | Action |
|---------------|-----------------|--------|
| 1-10 | Fast scanning, easy selection | Ideal |
| 11-20 | Acceptable with good grouping | Monitor |
| 21-30 | Scanning fatigue, users miss items | Split into subgroups |
| 31+ | Navigation breakdown | Restructure immediately |

### Grouping Rules

| Rule | Example | Anti-Pattern |
|------|---------|--------------|
| Use separator labels to chunk items | `---Getting Started---` | Flat list of 25 pages |
| Group by user task, not product feature | "Authentication" (task) | "AuthModule" (internal name) |
| Most-used items at the top of each group | "Quickstart" first | Alphabetical ordering |
| Limit nesting to 3 levels maximum | Section > Group > Page | Section > Sub > Sub > Sub > Page |

### Depth Limits

```
Level 1: Section (e.g., "Platform")
  Level 2: Group (e.g., "Authentication")
    Level 3: Page (e.g., "Configure SSO")
      Level 4: STOP — never go deeper
```

**Why 3 levels max:**
- Level 4+ items are invisible to scanning users
- Deep nesting signals unclear taxonomy (fix the category, don't add depth)
- Breadcrumbs become unwieldy beyond 3 segments
- Mobile navigation collapses poorly with deep nesting

### Sidebar Ordering

| Position | Content Type | Rationale |
|----------|-------------|-----------|
| First | Getting Started / Quickstart | New users look here first |
| Early | Guides (most common tasks) | Serve the majority use case |
| Middle | Explanation / Concepts | Users who need "why" |
| Late | Reference | Users who already know the model |
| Last | Contributing / Changelog | Niche audience |

## Breadcrumb Design

Breadcrumbs provide location awareness and enable upward navigation.

### Breadcrumb Rules

| Rule | Example | Anti-Pattern |
|------|---------|--------------|
| Always show full path from docs root | `Docs > Platform > Auth > SSO` | `SSO` (no context) |
| Each segment is clickable | All segments are links | Only the last segment is a link |
| Current page is not a link | Last segment is plain text | Last segment links to itself |
| Use `>` or `/` as separator | `Docs > Guides > Deploy` | `Docs :: Guides :: Deploy` |
| Maximum 4 segments | `Docs > Section > Group > Page` | `Docs > A > B > C > D > E > Page` |

### Breadcrumb + Sidebar Consistency

The breadcrumb path must exactly match the sidebar hierarchy. If the breadcrumb shows `Platform > Authentication > SSO`, the sidebar must have an "Authentication" group containing an "SSO" page under the "Platform" section.

## Contextual Navigation Switching

When a docs site has multiple top-level areas (e.g., "Platform", "API", "CLI"), the sidebar should switch context based on which area the user is in.

### Context Switching Rules

| Rule | Implementation |
|------|---------------|
| Top-level areas are mutually exclusive in the sidebar | Entering "API" hides "Platform" sidebar items |
| Context switch is triggered by top bar navigation | Clicking "API Reference" in top bar switches sidebar |
| Current context is visually indicated | Highlighted tab or section in top bar |
| Cross-context links open in context | A link from Platform to API switches the sidebar |

### Navigation Consistency Checklist

- [ ] Every page is reachable from the sidebar (no orphan pages)
- [ ] Sidebar labels match page titles exactly
- [ ] Breadcrumbs match sidebar hierarchy
- [ ] No page appears in multiple sidebar locations
- [ ] Context switching doesn't lose scroll position in previously visited sections

## Hick's Law Applied to Navigation

Hick's Law: `Decision Time = a + b * log2(n)` where n = number of choices.

### Practical Application

| Navigation Element | Max Choices | Rationale |
|--------------------|-------------|-----------|
| Top bar links | 5-7 | Primary navigation, always visible |
| Sidebar groups per section | 4-8 | Chunk level for section scanning |
| Pages per sidebar group | 5-12 | Item level within a chunk |
| Right TOC headings | 5-10 visible | On-page navigation |

### When Hick's Law Signals a Problem

| Symptom | Diagnosis | Fix |
|---------|-----------|-----|
| 8+ top bar links | Too many top-level sections | Consolidate or use mega-menu |
| 10+ sidebar groups | Section too broad | Split into multiple sections |
| 15+ pages in one group | Group too broad | Add subgroups with separators |
| 15+ headings in TOC | Page too long | Split into multiple pages |

## Audit Checklist

### Layout
- [ ] Three-pane layout implemented (sidebar, content, TOC)
- [ ] Content area max-width is 720-800px
- [ ] All panes scroll independently
- [ ] Left sidebar is sticky
- [ ] Right TOC auto-generates from page headings
- [ ] Right TOC highlights current section on scroll

### Sidebar
- [ ] No sidebar group exceeds 20 visible items
- [ ] Maximum nesting depth is 3 levels
- [ ] Items are grouped by user task, not internal names
- [ ] Most-used items appear first in each group
- [ ] Separator labels chunk long lists

### Breadcrumbs
- [ ] Breadcrumbs present on every page
- [ ] Full path from docs root shown
- [ ] All segments except current page are clickable
- [ ] Maximum 4 segments
- [ ] Path matches sidebar hierarchy exactly

### Consistency
- [ ] Every page reachable from sidebar (no orphans)
- [ ] Sidebar labels match page titles
- [ ] Context switching works correctly between sections
- [ ] Cross-context links switch sidebar appropriately

### Mobile
- [ ] Sidebar collapses to hamburger menu
- [ ] Right TOC is hidden (or moves to collapsible top-of-page)
- [ ] Breadcrumbs remain visible and don't overflow
- [ ] Content fills full width

## Output Format

### Navigation Audit: `{site or path}`

**Overall Score**: [EXCELLENT | GOOD | NEEDS WORK | CRITICAL]

**Layout Assessment**:
| Element | Status | Issue |
|---------|--------|-------|
| Three-pane layout | Pass/Fail | [note] |
| Sidebar governance | Pass/Fail | [item count, depth] |
| Breadcrumbs | Pass/Fail | [note] |
| Context switching | Pass/Fail | [note] |
| Mobile adaptation | Pass/Fail | [note] |

**Hick's Law Violations**:
| Element | Current Count | Max Recommended | Severity |
|---------|---------------|-----------------|----------|
| [element] | [count] | [max] | High/Medium/Low |

**Priority Fixes**:
1. [Fix]
2. [Fix]
3. [Fix]

## Guidelines

- **DO**: Limit sidebar to 20 visible items per group
- **DO**: Use task-based labels, not internal product names
- **DO**: Keep nesting to 3 levels maximum
- **DON'T**: Show the entire site tree in one sidebar
- **DON'T**: Allow orphan pages unreachable from navigation
- **DON'T**: Nest deeper than 3 levels — restructure taxonomy instead

## Related Components

- **docs-designer agent**: Handles individual page structure
- **governing-taxonomy skill**: Handles categorization decisions that feed into sidebar structure
- **designing-landing-pages skill**: Handles the docs homepage design
