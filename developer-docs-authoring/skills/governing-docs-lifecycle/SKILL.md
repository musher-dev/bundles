---
name: governing-docs-lifecycle
version: 1.0.0
description: Govern documentation lifecycle including docs-as-code workflows, quarterly content audits, page lifecycle states (draft/published/stale/deprecated/archived), stale content detection, contributor onboarding, and scaling governance. Use when managing docs freshness, setting up review cadences, defining content lifecycle states, or onboarding doc contributors. Triggered by: docs lifecycle, stale content, content audit, docs-as-code, deprecation, archived, draft, published, content governance, contributor onboarding, docs workflow, content freshness, review cadence, docs scaling, content retirement.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Governing Documentation Lifecycle

## Purpose

Define and enforce the lifecycle of documentation pages from creation through archival. As documentation scales, ungoverned content becomes stale, contradictory, and harmful. This skill provides the workflows, states, detection rules, and governance cadence to keep docs healthy over time.

This skill handles content lifecycle and governance processes. Content categorization is handled by governing-taxonomy. Content quality on individual pages is handled by docs-designer.

## Page Lifecycle States

Every documentation page exists in one of five states:

```
                ┌──────────┐
                │  DRAFT   │
                │ (hidden) │
                └────┬─────┘
                     │ publish
                     ▼
                ┌──────────┐
         ┌─────│ PUBLISHED │◄──────── refresh ──┐
         │     │ (visible) │                     │
         │     └────┬─────┘                      │
         │          │ 90 days no update           │
         │          ▼                             │
         │     ┌──────────┐                      │
         │     │  STALE   │──── review ──────────┘
         │     │ (warned) │
         │     └────┬─────┘
         │          │ decision: obsolete
         │          ▼
         │     ┌────────────┐
         │     │ DEPRECATED │
         │     │ (bannered) │
         │     └────┬───────┘
         │          │ 30 days
         │          ▼
         │     ┌──────────┐
         └────►│ ARCHIVED │
               │ (removed)│
               └──────────┘
```

### State Definitions

| State | Visibility | Banner | Search | Sidebar |
|-------|------------|--------|--------|---------|
| **Draft** | Hidden from public | None (not visible) | Excluded | Excluded |
| **Published** | Fully visible | None | Included | Included |
| **Stale** | Visible with warning | "This page may be outdated. Last reviewed: [date]" | Included, deprioritized | Included |
| **Deprecated** | Visible with redirect notice | "This page is deprecated. See [replacement]." | Excluded | Excluded |
| **Archived** | Removed from site | N/A | Excluded | Excluded |

### State Transitions

| Transition | Trigger | Who Decides |
|------------|---------|-------------|
| Draft → Published | Author marks ready, reviewer approves | Author + Reviewer |
| Published → Stale | 90 days since last update (automated) | Automated |
| Stale → Published | Content reviewed and confirmed current | Reviewer |
| Stale → Deprecated | Content is obsolete, replacement exists | Docs lead |
| Deprecated → Archived | 30 days after deprecation | Automated |
| Any → Draft | Major rewrite needed | Author |
| Archived → Published | Content becomes relevant again | Docs lead |

### Implementing Lifecycle States

Add lifecycle metadata to page frontmatter:

```yaml
---
title: "Configure SSO"
status: published          # draft | published | stale | deprecated | archived
last-reviewed: 2025-01-15  # date of last content review
deprecated-by: "/guides/authentication/configure-sso-v2"  # replacement URL
deprecated-date: 2025-02-01  # when deprecated
---
```

## Stale Content Detection

### Automated Detection Rules

| Signal | Threshold | Action |
|--------|-----------|--------|
| Days since last content update | > 90 days | Mark as stale |
| Days since last review | > 180 days | Flag for review even if recently updated |
| Referenced API version is not current | Version mismatch | Flag for review |
| Broken internal links | Any | Flag immediately |
| High bounce rate + low time-on-page | Above section average | Investigate quality |

### Stale Content Scan

Run quarterly to identify pages needing attention:

1. **List all pages with `last-reviewed` > 90 days ago**
2. **List all pages with no `last-reviewed` field** (never formally reviewed)
3. **Cross-reference with git log**: pages not touched in 6+ months
4. **Check for broken links** within the docs corpus
5. **Compare code examples** against current API version

### Stale Content Report Template

```markdown
## Quarterly Content Freshness Report: Q[N] [Year]

**Total Pages**: [count]
**By Status**:
| Status | Count | % |
|--------|-------|---|
| Published | [n] | [%] |
| Stale | [n] | [%] |
| Deprecated | [n] | [%] |
| Draft | [n] | [%] |

**Stale Pages Requiring Review**:
| Page | Last Updated | Last Reviewed | Section | Priority |
|------|-------------|---------------|---------|----------|
| [path] | [date] | [date] | [section] | High/Medium/Low |

**Broken Links Found**: [count]
| Source Page | Broken Link | Suggested Fix |
|-------------|-------------|---------------|
| [page] | [link] | [fix or remove] |

**Actions Required**:
1. [Review: page list]
2. [Deprecate: page list]
3. [Fix broken links: count]
```

## Docs-as-Code Workflow

### PR-Based Documentation Workflow

```
Author writes/edits content
         │
         ▼
    Create branch
         │
         ▼
    Open Pull Request
         │
         ├──→ Automated checks:
         │    - Lint (markdown, frontmatter)
         │    - Link validation
         │    - Build preview
         │
         ├──→ Human review:
         │    - Technical accuracy (engineer)
         │    - Content quality (docs reviewer)
         │    - Diataxis compliance (docs-designer)
         │
         ▼
    Merge to main
         │
         ▼
    Auto-deploy to docs site
```

### PR Review Checklist for Docs

| Category | Check | Required Reviewer |
|----------|-------|-------------------|
| Accuracy | Code examples work against current API | Engineer |
| Structure | Correct Diataxis quadrant, no mixing | Docs reviewer |
| Style | Follows writing style guide | Docs reviewer |
| Metadata | Frontmatter complete (title, description, status) | Automated |
| Links | All internal links resolve | Automated |
| Navigation | Page added to meta.json if new | Docs reviewer |

### Automated Checks

| Check | Tool/Method | Fail Behavior |
|-------|-------------|---------------|
| Markdown lint | markdownlint or similar | Block merge |
| Frontmatter validation | Custom schema check | Block merge |
| Internal link check | Build-time validation | Block merge |
| Spelling | cspell or similar | Warning |
| Build preview | Deploy preview branch | Required for visual review |

## Quarterly Content Audit

### Audit Process

| Week | Activity |
|------|----------|
| Week 1 | Run stale content scan, generate report |
| Week 2 | Section owners review flagged pages |
| Week 3 | Authors update, deprecate, or confirm pages |
| Week 4 | Docs lead reviews changes, updates status metadata |

### Audit Dimensions

| Dimension | Question | Action if No |
|-----------|----------|-------------|
| Accuracy | Does this still reflect the current product? | Update or deprecate |
| Relevance | Do users still need this? | Archive or deprecate |
| Completeness | Are there gaps users are asking about? | Create new content |
| Quality | Does this meet current writing standards? | Rewrite or edit |
| Findability | Can users find this via search and navigation? | Fix taxonomy/search |

## Contributor Onboarding

### For Internal Contributors

| Step | Content | Format |
|------|---------|--------|
| 1 | Writing style guide overview | 15-min read |
| 2 | Diataxis framework + which quadrant to use | Decision tree |
| 3 | Frontmatter requirements | Template + validation |
| 4 | PR workflow (branch, preview, review) | Walkthrough |
| 5 | First contribution: fix a typo or update a code example | Guided task |

### Contributor Guide Template

```markdown
# Contributing to Documentation

## Quick Start

1. Fork/branch the docs repo
2. Find the page you want to edit in `content/docs/`
3. Edit the MDX file
4. Ensure frontmatter is complete:
   ```yaml
   ---
   title: "Your Title"
   description: "Brief summary"
   ---
   ```
5. Open a PR with the label `docs`
6. A docs reviewer will be auto-assigned

## Writing Checklist

- [ ] Uses second person ("you") not first ("we")
- [ ] Uses present tense ("returns") not future ("will return")
- [ ] Active voice over passive
- [ ] One action per step
- [ ] Code examples are complete and runnable
- [ ] Verification section shows expected output

## What NOT to Do

- Don't mix Diataxis quadrants (tutorial + reference on same page)
- Don't add pages without updating meta.json
- Don't use "we" or "our" in user-facing content
- Don't commit without previewing the build
```

## Scaling Governance

### Team Size → Governance Model

| Team Size | Governance Model | Review Process |
|-----------|-----------------|----------------|
| 1-3 contributors | Lightweight: author self-reviews | PR with 1 reviewer |
| 4-10 contributors | Standard: assigned reviewers | PR with docs-team review |
| 11+ contributors | Formal: CODEOWNERS, automated checks | PR with automated + human review |

### CODEOWNERS for Docs

```
# .github/CODEOWNERS
/content/docs/getting-started/    @docs-team
/content/docs/guides/             @docs-team
/content/docs/reference/          @api-team @docs-team
/content/docs/explanation/        @docs-team
```

## Audit Checklist

### Lifecycle States
- [ ] All pages have `status` in frontmatter
- [ ] All pages have `last-reviewed` date
- [ ] Stale pages (>90 days) have warning banners
- [ ] Deprecated pages have redirect notices
- [ ] Archived pages are removed from site and search

### Workflow
- [ ] PR-based workflow is documented
- [ ] Automated checks run on docs PRs (lint, links, build)
- [ ] Review checklist exists and is followed
- [ ] Deploy previews are available for review

### Governance
- [ ] Quarterly audit cadence is scheduled
- [ ] Stale content report is generated
- [ ] Section owners are assigned
- [ ] Contributor guide exists and is current
- [ ] CODEOWNERS covers all docs sections

## Output Format

### Lifecycle Audit: `{scope}`

**Page Status Distribution**:
| Status | Count | % | Target |
|--------|-------|---|--------|
| Published | [n] | [%] | > 80% |
| Stale | [n] | [%] | < 15% |
| Draft | [n] | [%] | < 5% |
| Deprecated | [n] | [%] | < 5% |

**Governance Health**:
| Process | Status | Notes |
|---------|--------|-------|
| PR workflow | Active/Inactive | [note] |
| Automated checks | Active/Inactive | [note] |
| Quarterly audits | Active/Inactive | [note] |
| Contributor guide | Current/Stale | [note] |

**Priority Fixes**:
1. [Fix]
2. [Fix]
3. [Fix]

## Guidelines

- **DO**: Automate stale detection based on `last-reviewed` dates
- **DO**: Show warning banners on stale pages
- **DO**: Provide a clear contributor onboarding path
- **DON'T**: Delete stale content without reviewing — it may just need a refresh
- **DON'T**: Skip the deprecation state — jumping from published to archived breaks bookmarks
- **DON'T**: Let the quarterly audit slip — stale content compounds

## Related Components

- **governing-taxonomy skill**: Taxonomy metadata feeds into lifecycle tracking
- **auditing-docs-metrics skill**: Metrics identify pages with quality issues
- **docs-designer agent**: Ensures refreshed content meets quality standards
- **designing-search-discovery skill**: Stale/deprecated pages should be deprioritized in search
