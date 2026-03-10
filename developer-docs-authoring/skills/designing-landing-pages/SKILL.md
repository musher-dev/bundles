---
name: designing-landing-pages
version: 1.0.0
description: Design and audit developer documentation landing pages for hero sections, self-selection routing, hello-world visuals, getting-started blocks, architecture overviews, and trust signals. Use when creating docs homepages, designing entry points, or optimizing first-visit experience. Triggered by: docs landing page, docs homepage, hero section, self-selection, getting started, docs entry point, first visit, docs home, documentation portal, developer portal, docs index page.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Designing Documentation Landing Pages

## Purpose

Design and audit the landing page (homepage) of a developer documentation site. The docs landing page is the highest-traffic, highest-stakes page — it must orient new visitors in seconds, route them to the right content, and build confidence that the docs will solve their problem.

This skill covers the structural blueprint, not the writing style (handled by docs-designer) or concept clarity (handled by concept-designer).

## Landing Page Blueprint

A documentation landing page has 7 sections in a fixed order. Each section has a specific job.

```
┌──────────────────────────────────────────┐
│  1. HERO: Identity + Value Prop          │
│  ─ What is this product?                 │
│  ─ One-sentence value proposition        │
│  ─ Primary CTA → Getting Started         │
├──────────────────────────────────────────┤
│  2. HELLO-WORLD VISUAL                   │
│  ─ 3-5 line code snippet or terminal     │
│  ─ Shows the simplest possible success   │
│  ─ Must be copy-pasteable                │
├──────────────────────────────────────────┤
│  3. SELF-SELECTION ROUTING               │
│  ─ 3-5 cards for distinct user paths     │
│  ─ Labels match user intent, not product │
│  ─ "I want to..." framing               │
├──────────────────────────────────────────┤
│  4. GETTING STARTED BLOCK                │
│  ─ Step preview (3-5 steps, titles only) │
│  ─ Time estimate ("5 minutes")           │
│  ─ Prerequisites summary                 │
├──────────────────────────────────────────┤
│  5. ARCHITECTURE OVERVIEW                │
│  ─ System diagram or concept map         │
│  ─ Shows how pieces fit together         │
│  ─ Links to Explanation pages            │
├──────────────────────────────────────────┤
│  6. COMMUNITY / SUPPORT                  │
│  ─ Links to Discord, GitHub, forums      │
│  ─ Shows community is alive (metrics)    │
│  ─ Support channel for paid users        │
├──────────────────────────────────────────┤
│  7. TRUST SIGNALS                        │
│  ─ GitHub stars, contributor count        │
│  ─ Logo bar (if applicable)              │
│  ─ "Used by X developers" metric         │
└──────────────────────────────────────────┘
```

## Section Design Rules

### 1. Hero Section

| Element | Rule | Anti-Pattern |
|---------|------|--------------|
| Headline | Product name + one-clause value prop | "Welcome to the docs" |
| Subheadline | What the product does in <=15 words | Feature list or buzzword soup |
| Primary CTA | Links to getting-started tutorial | Links to API reference |
| Visual | Product screenshot or code snippet | Stock photo or abstract illustration |

**Headline formula:** `[Product] — [what it does for the user]`

Examples:
- "Stripe — Financial infrastructure for the internet"
- "Tailwind CSS — Rapidly build modern websites without ever leaving your HTML"

### 2. Hello-World Visual

The hello-world visual answers: "What does using this product look like?"

| Rule | Rationale |
|------|-----------|
| Show the simplest possible success | Reduces perceived complexity |
| Must be copy-pasteable code | Enables immediate action |
| 3-5 lines maximum | Fits in viewport without scrolling |
| Show output alongside input | Proves it works |
| Use syntax highlighting | Reduces cognitive load |

**Template:**
```
[Installation command - 1 line]
[Configuration/setup - 1-2 lines]
[The "do the thing" command - 1 line]
[Output showing success - 1-2 lines]
```

### 3. Self-Selection Routing

Route visitors by intent, not by product taxonomy.

| Approach | Good | Bad |
|----------|------|-----|
| Label framing | "I want to deploy my app" | "Deployment Module" |
| Card count | 3-5 cards | 8+ cards |
| Card content | Icon + title + one-sentence description | Title only, or walls of text |
| Link target | Tutorial or guide landing | API reference or concept page |

**Common routing patterns:**

| Card | Target User | Points To |
|------|-------------|-----------|
| "Get Started" | New user, just evaluating | Tutorial (getting-started/) |
| "Guides" | User with a specific task | Guide index page |
| "API Reference" | User who knows the API model | Reference index |
| "Examples" | User who learns by example | Examples gallery |
| "Migrate from X" | User switching from competitor | Migration guide |

### 4. Getting Started Block

Preview the onboarding journey without requiring commitment.

| Element | Rule |
|---------|------|
| Step count | 3-5 steps, titles only (not full instructions) |
| Time estimate | Must be present ("Complete in 5 minutes") |
| Prerequisites | One line listing what's needed |
| CTA | "Start the tutorial" linking to full page |

### 5. Architecture Overview

| Element | Rule | Anti-Pattern |
|---------|------|--------------|
| Diagram type | System context or component diagram | Class diagrams or ER diagrams |
| Complexity | 4-7 boxes maximum | 15+ boxes with all internal details |
| Labels | Use product vocabulary, not generic CS terms | "Service Mesh" when product calls it "Network" |
| Interactivity | Clickable boxes linking to Explanation pages | Static image with no links |

### 6. Community / Support

| Signal | Minimum Bar | Premium |
|--------|-------------|---------|
| GitHub link | Repository link | Stars count + "Star us" CTA |
| Chat | Discord/Slack invite link | Member count + recent activity |
| Forum | Link to discussions | "X questions answered this month" |
| Support | Support email or docs | Tiered: community / priority / enterprise |

### 7. Trust Signals

| Signal Type | Implementation | Placement |
|-------------|---------------|-----------|
| Usage metric | "Used by X developers" or "X API calls/day" | Hero-adjacent or below routing cards |
| Logo bar | 4-8 recognizable logos | Below hero or at page bottom |
| GitHub stars | Live count badge | Hero or community section |
| Compliance | SOC2, GDPR badges if applicable | Footer or dedicated trust row |

## Mobile Responsiveness Rules

| Desktop Layout | Mobile Adaptation |
|----------------|-------------------|
| Side-by-side hero (text + code) | Stack vertically, code below |
| 3-5 routing cards in a row | Single column, scrollable |
| Architecture diagram (wide) | Simplified version or horizontal scroll |
| Community section (3 columns) | Single column stack |

## Audit Checklist

### Structure
- [ ] All 7 sections present in correct order
- [ ] Hero has headline, subheadline, and CTA
- [ ] Hello-world visual is copy-pasteable and <=5 lines
- [ ] Self-selection cards number 3-5 and use intent-based labels
- [ ] Getting-started block shows step preview with time estimate
- [ ] Architecture overview has <=7 boxes with clickable links
- [ ] Community section has at least 2 active channels
- [ ] Trust signals include at least 1 quantified metric

### User Experience
- [ ] A new visitor can identify the product's purpose within 5 seconds
- [ ] The primary CTA leads to getting-started tutorial (not API reference)
- [ ] Self-selection routing matches user intent, not product internals
- [ ] Page works on mobile without horizontal scrolling
- [ ] No section requires scrolling more than 1 viewport height

### Anti-Patterns
- [ ] No "Welcome to the docs" as the hero headline
- [ ] No feature matrix on the landing page (belongs in comparison pages)
- [ ] No changelog or release notes on the landing page
- [ ] No login/signup wall before any content
- [ ] Routing cards do not exceed 5
- [ ] No broken diagram links

## Output Format

When designing or auditing a docs landing page:

### Landing Page Audit: `{url or path}`

**Overall Score**: [EXCELLENT | GOOD | NEEDS WORK | CRITICAL]

**Section Assessment**:
| Section | Present | Quality | Issue |
|---------|---------|---------|-------|
| Hero | Yes/No | 1-5 | [brief note] |
| Hello-World | Yes/No | 1-5 | [brief note] |
| Self-Selection | Yes/No | 1-5 | [brief note] |
| Getting Started | Yes/No | 1-5 | [brief note] |
| Architecture | Yes/No | 1-5 | [brief note] |
| Community | Yes/No | 1-5 | [brief note] |
| Trust Signals | Yes/No | 1-5 | [brief note] |

**Priority Fixes**:
1. [Most impactful fix]
2. [Second fix]
3. [Third fix]

**Proposed Content**:
```markdown
[Concrete landing page content or structural changes]
```

## Guidelines

- **DO**: Optimize for the first 5 seconds of a new visitor's experience
- **DO**: Use intent-based labels ("I want to...") for routing cards
- **DO**: Show a working code snippet, not a product screenshot
- **DON'T**: Put API reference as the primary CTA
- **DON'T**: Use more than 5 routing cards (Hick's Law)
- **DON'T**: Skip the time estimate on getting-started previews

## Related Components

- **docs-designer agent**: Handles individual page structure and writing quality
- **concept-designer agent**: Handles concept clarity within pages
- **designing-navigation-systems skill**: Handles sidebar and breadcrumb navigation
