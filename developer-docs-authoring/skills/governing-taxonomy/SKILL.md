---
name: governing-taxonomy
version: 1.0.0
description: Design and govern documentation taxonomy decisions including task-based vs topic-based categorization, hybrid models, split/nest rules, content model metadata schemas, controlled vocabulary for labels, and taxonomy governance boards. Use when organizing docs categories, deciding how to split or nest content, defining content models, or establishing labeling conventions. Triggered by: taxonomy, categorization, content model, split or nest, task-based, topic-based, labels, vocabulary, metadata schema, docs organization, content structure, IA, information architecture, classification, tagging.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Governing Documentation Taxonomy

## Purpose

Make and enforce decisions about how documentation content is categorized, labeled, and organized. Taxonomy is the invisible skeleton beneath navigation — when it's wrong, users can't find content even when navigation looks correct. This skill provides decision frameworks for categorization, split/nest rules, and controlled vocabulary governance.

This skill handles categorization logic. Physical navigation structure is handled by designing-navigation-systems. Individual page structure is handled by docs-designer.

## Taxonomy Models

### Task-Based Taxonomy

Organize by what the user wants to accomplish.

```
├── get-started/
├── deploy/
├── authenticate/
├── monitor/
├── migrate/
└── troubleshoot/
```

| Strength | Weakness |
|----------|----------|
| Matches user intent directly | Doesn't scale when tasks cross domains |
| Easy for new users to navigate | Expert users may find it redundant |
| Maps cleanly to Diataxis How-to quadrant | Hard to place concept/explanation content |

### Topic-Based Taxonomy

Organize by product domain or feature area.

```
├── workspaces/
├── authentication/
├── billing/
├── api/
├── cli/
└── webhooks/
```

| Strength | Weakness |
|----------|----------|
| Scales well with product complexity | New users don't know which "topic" to visit |
| Maps to product architecture | Cross-cutting concerns lack a home |
| Experts can navigate directly | Assumes domain knowledge |

### Hybrid Model (Recommended)

Use task-based at the top level, topic-based within sections.

```
├── get-started/              ← Task-based (top level)
│   ├── quickstart.mdx
│   └── first-deployment.mdx
├── guides/                   ← Task-based (top level)
│   ├── authentication/       ← Topic-based (second level)
│   │   ├── setup-sso.mdx
│   │   └── manage-api-keys.mdx
│   ├── deployment/           ← Topic-based (second level)
│   │   ├── deploy-to-aws.mdx
│   │   └── configure-ci.mdx
│   └── billing/
│       └── manage-plans.mdx
├── reference/                ← Diataxis quadrant (top level)
│   ├── api/                  ← Topic-based (second level)
│   └── cli/
└── explanation/              ← Diataxis quadrant (top level)
    ├── architecture.mdx
    └── security-model.mdx
```

**Hybrid rules:**
- Top level: Diataxis quadrants + task-based entry points
- Second level: Topic-based grouping within each quadrant
- Third level: Individual pages (never deeper)

## Split / Nest Decision Framework

When a section grows, you face a choice: split into siblings or nest as children.

### Decision Tree

```
Is the content about the SAME concept at DIFFERENT depths?
  → YES → NEST (parent-child)
  → NO  → Are they PEER concepts at the SAME depth?
            → YES → SPLIT (siblings)
            → NO  → RECLASSIFY (wrong section)
```

### Split vs Nest Rules

| Scenario | Decision | Example |
|----------|----------|---------|
| "Authentication" has SSO, API Keys, OAuth | NEST under Authentication | `auth/sso.mdx`, `auth/api-keys.mdx` |
| "Authentication" and "Authorization" are peers | SPLIT as siblings | `auth/`, `authorization/` |
| "Webhooks" appears under both API and Events | RECLASSIFY — pick one home | `api/webhooks/` with cross-link from Events |
| A single page exceeds 2000 words | SPLIT into subpages | `deploy/` → `deploy/aws.mdx`, `deploy/gcp.mdx` |
| A section has only 1-2 pages | MERGE upward into parent | Don't create `monitoring/` for just one page |

### Split Threshold Rules

| Metric | Threshold | Action |
|--------|-----------|--------|
| Pages in a group | > 12 | Split into subgroups |
| Word count per page | > 2000 | Split into subpages |
| Sections at one level | > 8 | Nest some as subgroups |
| Pages in a section | < 3 | Merge into parent section |
| Nesting depth | > 3 | Flatten — restructure taxonomy |

## Content Model Metadata Schema

Every documentation page should have metadata that enables governance, search, and lifecycle management.

### Required Metadata

```yaml
---
title: "Page Title"                    # Display title, max 60 chars
description: "Brief summary"           # SEO description, max 160 chars
quadrant: tutorial | guide | reference | explanation  # Diataxis classification
---
```

### Recommended Metadata

```yaml
---
audience: beginner | intermediate | advanced   # Expertise level
product-area: platform | api | cli | billing   # Topic domain
last-reviewed: 2025-01-15                       # Governance tracking
status: draft | published | stale | deprecated  # Lifecycle state
tags:                                           # Discovery labels
  - authentication
  - sso
---
```

### Metadata Governance Rules

| Field | Who Sets It | When Updated |
|-------|------------|--------------|
| title, description | Author at creation | On any content change |
| quadrant | Author, verified by reviewer | Rarely — only if page purpose changes |
| audience | Author at creation | On major rewrite |
| product-area | Author at creation | If product reorganizes |
| last-reviewed | Automated or reviewer | At each quarterly review |
| status | Automated + governance board | Per lifecycle rules |
| tags | Author, governed by controlled vocabulary | On content change |

## Controlled Vocabulary

### Label Governance

All tags and labels must come from a controlled vocabulary. Ungoverned tags devolve into chaos (synonyms, typos, inconsistent granularity).

### Establishing a Controlled Vocabulary

1. **Audit existing labels**: Collect all tags/labels currently in use
2. **Normalize synonyms**: Pick one canonical term for each concept
3. **Define hierarchy**: Group labels into facets (domain, audience, content-type)
4. **Document the vocabulary**: Publish the allowed labels list
5. **Enforce at PR review**: Reject PRs that introduce unlisted labels

### Label Facets

| Facet | Purpose | Example Values |
|-------|---------|---------------|
| Domain | Product area | `authentication`, `billing`, `deployment`, `api` |
| Audience | Expertise level | `beginner`, `intermediate`, `advanced` |
| Content-Type | Diataxis quadrant | `tutorial`, `guide`, `reference`, `explanation` |
| Platform | Target environment | `aws`, `gcp`, `azure`, `self-hosted` |
| Language | Code language | `python`, `javascript`, `go`, `rust` |

### Synonym Resolution Table Template

| Canonical Term | Rejected Synonyms | Notes |
|----------------|-------------------|-------|
| `authentication` | `auth`, `login`, `sign-in` | Use full word |
| `deployment` | `deploy`, `shipping`, `release` | Noun form |
| `api` | `rest-api`, `endpoints`, `routes` | Lowercase, no qualifier |
| `getting-started` | `quickstart`, `intro`, `onboarding` | Hyphenated |

## Taxonomy Governance Board

### Purpose

A lightweight process to prevent taxonomy drift as docs grow.

### Board Composition

| Role | Responsibility |
|------|---------------|
| Docs lead | Final decision on taxonomy disputes |
| Product representative | Ensures alignment with product model |
| Developer advocate | Represents user perspective |

### Governance Cadence

| Event | Frequency | Action |
|-------|-----------|--------|
| New section proposal | As needed | Board reviews and approves |
| Label addition request | As needed | Check against vocabulary, approve or merge |
| Taxonomy audit | Quarterly | Review sidebar item counts, depth, orphan pages |
| Vocabulary review | Semi-annually | Prune unused labels, resolve new synonyms |

### Decision Log Template

```markdown
## Taxonomy Decision: [Date]

**Proposed Change**: [What was proposed]
**Decision**: [Approved / Rejected / Modified]
**Rationale**: [Why]
**Impact**: [Pages affected, navigation changes]
**Action Items**: [Who does what]
```

## Audit Checklist

### Taxonomy Model
- [ ] Top-level organization follows hybrid model (task-based + Diataxis)
- [ ] Second-level uses topic-based grouping
- [ ] No section deeper than 3 levels
- [ ] No section with fewer than 3 pages (merge candidates)
- [ ] No group with more than 12 pages (split candidates)

### Content Model
- [ ] All pages have required metadata (title, description, quadrant)
- [ ] Quadrant assignment is correct (no mixed-quadrant pages)
- [ ] Tags come from the controlled vocabulary (no rogue labels)
- [ ] No synonym drift in labels

### Governance
- [ ] Controlled vocabulary is documented and accessible
- [ ] New labels go through approval process
- [ ] Quarterly taxonomy audit is scheduled
- [ ] Decision log exists and is maintained

## Output Format

### Taxonomy Audit: `{scope}`

**Model Assessment**: [Hybrid / Task-Based / Topic-Based / Mixed (needs work)]

**Split/Nest Issues**:
| Section | Pages | Depth | Issue | Recommendation |
|---------|-------|-------|-------|----------------|
| [section] | [count] | [depth] | [issue] | Split / Merge / Restructure |

**Label Health**:
| Issue | Count | Examples |
|-------|-------|---------|
| Unlisted labels | [n] | [examples] |
| Synonym drift | [n] | [canonical → found variants] |
| Unused labels | [n] | [examples] |

**Priority Fixes**:
1. [Fix]
2. [Fix]
3. [Fix]

## Guidelines

- **DO**: Use the hybrid model (task-based top + topic-based within)
- **DO**: Maintain a controlled vocabulary for all labels
- **DO**: Follow split/nest rules based on thresholds
- **DON'T**: Let taxonomy be ad-hoc — every section needs a rationale
- **DON'T**: Create sections with fewer than 3 pages
- **DON'T**: Nest deeper than 3 levels — flatten instead

## Related Components

- **designing-navigation-systems skill**: Consumes taxonomy decisions to build sidebar structure
- **docs-designer agent**: Enforces Diataxis quadrant assignments on individual pages
- **governing-docs-lifecycle skill**: Uses metadata schema for lifecycle management
