---
name: auditing-content-quality
version: 1.0.0
description: Audit documentation pages against a 5-dimension editorial quality rubric (Findability, Accuracy, Clarity, Task Orientation, Readability) and a pre-launch validation checklist. Use when reviewing docs before publish, running editorial quality gates, or scoring page quality. Triggered by: content quality, editorial review, quality rubric, docs audit, page review, pre-launch checklist, findability, accuracy, clarity, readability, quality gate, editorial quality, docs quality score, publish checklist.
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Auditing Content Quality

## Purpose

Apply a per-page editorial quality rubric and pre-launch validation checklist to developer documentation. This skill assesses qualitative editorial quality — whether a page communicates effectively — as a gate before publishing.

This skill is a read-only audit tool. It produces quality assessments but does not fix issues (handled by writing-focused skills and the docs-designer agent). It does not measure quantitative analytics KPIs (handled by auditing-docs-metrics).

## Quality Rubric: 5 Dimensions

Score each dimension 1-5. A page must score 3+ on every dimension and 20+ total to pass the quality gate.

### 1. Findability (Weight: High)

Can a user locate this page within 30 seconds?

| Score | Criteria |
|-------|----------|
| 5 | Appears in top 3 search results for its primary keyword; linked from navigation, related pages, and landing page |
| 4 | Appears in search results; linked from navigation and at least one other page |
| 3 | Appears in search results; present in sidebar navigation |
| 2 | In navigation but poor search ranking; title doesn't match search intent |
| 1 | Not in search results; not linked from navigation; orphan page |

**30-Second Test:** Starting from the docs homepage, can you reach this page in under 30 seconds using navigation or search? Time it.

**Evaluation Signals:**
- Page title matches common search queries for this topic
- Breadcrumbs provide clear location context
- Related pages link to and from this page
- URL slug is human-readable and keyword-relevant

### 2. Accuracy (Weight: Critical)

Does the content work when executed?

| Score | Criteria |
|-------|----------|
| 5 | All code examples verified in clean environment within the last 30 days; API responses match current version |
| 4 | Code examples work; minor version discrepancies in screenshots or UI references |
| 3 | Core examples work; some peripheral examples untested or slightly outdated |
| 2 | Multiple broken examples or outdated API references |
| 1 | Page describes deprecated behavior; code examples fail |

**Execution Test:** Copy every code block on the page and run it in a clean environment. Record pass/fail per block.

**Evaluation Signals:**
- API version referenced matches current production version
- Code examples produce the documented output
- Screenshots match current UI
- Links to external resources resolve (no 404s)
- Parameter names and types match current API schema

### 3. Clarity (Weight: High)

Is the content understandable without prior domain expertise beyond stated prerequisites?

| Score | Criteria |
|-------|----------|
| 5 | No jargon without definition; all prerequisites explicit; admonitions for every gotcha; single concept per paragraph |
| 4 | Minimal jargon; prerequisites listed; most gotchas flagged |
| 3 | Some unexplained jargon; prerequisites mostly stated; major gotchas covered |
| 2 | Frequent undefined jargon; implicit prerequisites; missing gotcha warnings |
| 1 | Assumes expert knowledge; no prerequisites; no admonitions |

**Evaluation Signals:**
- Every technical term is either defined on first use or linked to a glossary
- Prerequisites are listed with version numbers, not assumed
- Admonitions (warning, note, tip) flag non-obvious behavior
- Each paragraph conveys one idea
- Sentences average under 25 words

### 4. Task Orientation (Weight: High)

Does the page commit to a single Diataxis archetype without mixing?

| Score | Criteria |
|-------|----------|
| 5 | Pure archetype: page matches one archetype completely with no content from other types |
| 4 | Dominant archetype with minor cross-type content (e.g., a brief concept explanation in a how-to) |
| 3 | Identifiable archetype but noticeable mixing (concept paragraphs in a quickstart) |
| 2 | Unclear archetype; mixes tutorial steps with reference tables |
| 1 | No discernible archetype; rambling mixture of concepts, steps, and reference |

**Evaluation Signals:**
- Page title signals its archetype (verb-first for how-to, noun-first for reference)
- Content follows the anatomy defined in scaffolding-page-archetypes
- Conceptual explanations are linked to, not inlined
- Setup steps appear only in quickstarts, not repeated in how-tos

### 5. Readability (Weight: Medium)

Is the page physically comfortable to read for sustained periods?

| Score | Criteria |
|-------|----------|
| 5 | 45-90 character line measure; concise paragraphs (≤4 sentences); generous whitespace; clear heading hierarchy |
| 4 | Acceptable line length; mostly concise paragraphs; adequate whitespace |
| 3 | Slightly long lines or dense paragraphs; functional but fatiguing |
| 2 | Long lines (>100 chars), wall-of-text paragraphs, cramped layout |
| 1 | Unformatted text; no headings; no visual structure |

**Evaluation Signals:**
- Line measure is 45-90 characters (check with a ruler or dev tools)
- Paragraphs are 1-4 sentences
- Heading hierarchy is sequential (no skipped levels)
- Code blocks have adequate padding and don't overflow
- Lists are used instead of comma-separated inline enumerations

## Pre-Launch Validation Checklist

Before launching or relaunching a documentation site, verify:

### Content Completeness
- [ ] OpenAPI specification parsed into API reference pages for all endpoints
- [ ] llms.txt populated and validated (see generating-llm-indexes skill)
- [ ] Every quickstart code example verified in a clean environment
- [ ] Error code dictionary covers all API error codes

### Search & Discovery
- [ ] Search index populated with all published pages
- [ ] Zero-result queries have fallback suggestions configured
- [ ] Top 20 expected search queries return relevant results

### Accessibility & Performance
- [ ] Keyboard navigation functional: Cmd+K search, Tab through interactive elements
- [ ] WCAG AA compliance for all text and interactive elements
- [ ] Lighthouse scores: FCP < 1.8s, CLS < 0.1

### Freshness Tracking
- [ ] Page Usefulness Score widget deployed on every page (binary thumbs up/down)
- [ ] Freshness coverage baseline: % of pages reviewed within trailing 90 days

## Ongoing Quality Metrics

### Page Usefulness Score

Deploy a binary feedback widget ("Was this page helpful? Yes / No") on every documentation page.

| Metric | Target | Action if Below |
|--------|--------|----------------|
| Usefulness rate | > 80% "Yes" | Review and revise page content |
| Response rate | > 5% of page views | Ensure widget is visible and non-intrusive |
| Per-page trend | Stable or improving | Investigate drops correlating with content changes |

### Freshness Coverage

Track the percentage of pages reviewed (content verified accurate) within the trailing 90-day window.

| Coverage | Rating | Action |
|----------|--------|--------|
| > 90% | Healthy | Maintain cadence |
| 70-90% | Acceptable | Prioritize stale pages |
| < 70% | At risk | Dedicated content sprint |

## Scoring Template

```markdown
## Content Quality Audit: {page title}

**URL**: {url}
**Archetype**: {quickstart | how-to | reference | webhooks | troubleshooting}
**Auditor**: {name}
**Date**: {date}

| Dimension | Score | Notes |
|-----------|-------|-------|
| 1. Findability | /5 | |
| 2. Accuracy | /5 | |
| 3. Clarity | /5 | |
| 4. Task Orientation | /5 | |
| 5. Readability | /5 | |
| **Total** | **/25** | |

**Pass/Fail**: [Pass (≥20 and no dimension <3) | Fail]

**Issues**:
1. [Issue description + location + severity]

**Recommendations**:
1. [Prioritized fix]
```

## Output Format

### Content Quality Audit: `{page title}`

**Summary**: [Pass/Fail] — [total]/25, [dimension scores]

**Critical Issues** (score <3):
- [Dimension]: [issue and recommended fix]

**Improvement Opportunities** (score 3-4):
- [Dimension]: [specific suggestion]

**Pre-Launch Checklist** (if applicable):
- [Status of each checklist item]

## Guidelines

- **DO**: Run the execution test on every code block — don't assume it works
- **DO**: Time the 30-second findability test literally
- **DO**: Score each dimension independently — a perfect score on one doesn't compensate for another
- **DO**: Note the page archetype before scoring Task Orientation
- **DON'T**: Score based on how the page should work — score what's actually published
- **DON'T**: Combine this audit with quantitative metrics — use auditing-docs-metrics for analytics
- **DON'T**: Skip Readability because "content is correct" — typography affects comprehension

## Related Components

- **auditing-docs-metrics skill**: Quantitative KPI measurement (TTFS, bounce rate) — complements this qualitative assessment
- **scaffolding-page-archetypes skill**: Defines archetype anatomy that Task Orientation scores against
- **designing-typography-layout skill**: Typography standards that Readability scores against
- **designing-code-displays skill**: Code block patterns that Accuracy verification tests
- **docs-designer agent**: Can remediate issues found by this audit
- **governing-docs-lifecycle skill**: Freshness coverage feeds into lifecycle governance
