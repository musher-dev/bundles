---
name: designing-search-discovery
version: 1.0.0
description: Design and audit documentation search and discovery systems including federated search, scoped search, CMD+K patterns, synonym handling, zero-result recovery, and search analytics feedback loops. Use when building docs search, reviewing search UX, fixing zero-result rates, or optimizing content discovery. Triggered by: search, docs search, federated search, CMD+K, search UX, synonym, zero results, search analytics, content discovery, search design, scoped search, search-to-click, findability, search bar, autocomplete, search relevance.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Designing Search & Discovery

## Purpose

Design and audit the search and discovery systems of a developer documentation site. Search is the escape hatch when navigation fails — and for 40-60% of developer docs users, search is the primary navigation method. This skill covers search UX patterns, relevance tuning, synonym management, and analytics feedback loops.

This skill handles search and discovery mechanisms. Navigation structure is handled by designing-navigation-systems. Content categorization is handled by governing-taxonomy.

## User Behavior Models

### Link-Dominant vs Search-Dominant Users

Research shows developers split into two navigation styles:

| Style | Behavior | Percentage | Implication |
|-------|----------|------------|-------------|
| **Link-dominant** | Navigates via sidebar, links, breadcrumbs | ~40-50% | Invest in navigation structure |
| **Search-dominant** | Goes directly to search, ignores sidebar | ~40-50% | Invest in search quality |
| **Mixed** | Uses both depending on familiarity | ~10-20% | Both systems must work |

**Key insight:** You cannot choose between good navigation and good search. Both are required.

## Search UX Patterns

### CMD+K Pattern

The modern developer docs search pattern: keyboard-activated modal search.

| Element | Specification |
|---------|--------------|
| Activation | `Cmd+K` (Mac) / `Ctrl+K` (Windows/Linux) |
| Visual | Modal overlay with input field, centered |
| Placeholder | "Search docs..." or "Search [product] docs..." |
| Results | Appear inline as user types (no page reload) |
| Navigation | Arrow keys to select, Enter to navigate, Esc to close |
| Shortcut hint | Show `⌘K` in the top bar search input |

**CMD+K Requirements:**
- [ ] Activates on keyboard shortcut
- [ ] Also activatable by clicking the search bar
- [ ] Shows results within 100ms of keystroke
- [ ] Supports arrow key navigation of results
- [ ] Esc closes the modal
- [ ] Shows the keyboard shortcut hint in the search bar

### Search Result Design

| Element | Rule | Anti-Pattern |
|---------|------|--------------|
| Title | Page title, highlighted match | Full URL path |
| Snippet | Context around the match, bolded keywords | First 100 chars of page |
| Breadcrumb | Show section path below snippet | No location context |
| Icon | Indicate content type (guide, reference, API) | No visual differentiation |
| Grouping | Group by section or content type | Flat ungrouped list |

### Result Ranking

| Factor | Weight | Rationale |
|--------|--------|-----------|
| Title match | Highest | User often searches for page titles |
| Heading match | High | Section headings indicate topic coverage |
| Content match | Medium | Full-text relevance |
| Page popularity | Low boost | Popular pages are more likely targets |
| Recency | Low boost | Recently updated content may be more relevant |
| Content type | Contextual | Tutorials rank higher for beginner queries |

## Scoped Search

Allow users to narrow search to a specific docs section.

### Scope Options

| Scope | When Useful | Implementation |
|-------|-------------|---------------|
| All docs | Default, broadest search | No filter applied |
| Current section | User is already in the right area | Filter by sidebar section |
| API Reference | Looking for specific endpoint/parameter | Filter by `/reference/` path |
| Guides | Looking for how-to instructions | Filter by `/guides/` path |

### Scoped Search UX

```
┌─────────────────────────────────────────┐
│  🔍  Search docs...          ⌘K        │
│  ┌─────────────────────────────────────┐│
│  │  Scope: [All Docs ▼]               ││
│  │         ├── All Docs                ││
│  │         ├── Current Section         ││
│  │         ├── API Reference           ││
│  │         ├── Guides                  ││
│  │         └── Tutorials               ││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

## Federated Search

For products with docs spread across multiple sources (docs site, API reference, changelog, blog, community forum).

### Federated Search Architecture

```
User Query
    │
    ├──→ Docs Index ──────→ Results (primary)
    ├──→ API Reference ───→ Results (secondary)
    ├──→ Community/Forum ─→ Results (tertiary)
    └──→ Blog/Changelog ──→ Results (lowest priority)
    │
    └──→ Merged, Ranked, Grouped Results
```

### Federated Search Rules

| Rule | Rationale |
|------|-----------|
| Docs results always appear first | Core docs are the primary resource |
| API results in separate group | Users know when they want API vs prose |
| Community results labeled clearly | User-generated content has different authority |
| Blog results appear last | Often outdated, not the canonical source |
| Each source shows max 3-5 results | Prevents one source from drowning others |

## Synonym Handling

### Why Synonyms Matter

Users search with their vocabulary, not the product's vocabulary. Without synonym handling, a search for "login" returns nothing when the docs use "authenticate."

### Synonym Types

| Type | Example | Handling |
|------|---------|---------|
| **Conceptual synonym** | "login" ↔ "authenticate" | Bidirectional mapping |
| **Abbreviation** | "k8s" ↔ "kubernetes" | Expand abbreviation |
| **Version name** | "v2" ↔ "current" | Map to canonical version |
| **Colloquial** | "env var" ↔ "environment variable" | Expand informal term |
| **Competitor term** | "Heroku dyno" ↔ "[product] worker" | Map competitor vocabulary |

### Synonym Dictionary Template

```yaml
synonyms:
  - canonical: authentication
    aliases: [auth, login, sign-in, sign in, log in]
  - canonical: environment variable
    aliases: [env var, envvar, env, environment config]
  - canonical: deployment
    aliases: [deploy, ship, release, push to prod]
  - canonical: kubernetes
    aliases: [k8s, kube]
  - canonical: command line
    aliases: [cli, terminal, shell, console]
```

### Building the Synonym Dictionary

1. **Mine zero-result queries**: Most common zero-result searches reveal missing synonyms
2. **Mine support tickets**: Terms users use when asking for help
3. **Mine competitor docs**: Terms users bring from other products
4. **Review quarterly**: Add new synonyms, prune unused ones

## Zero-Result Recovery

When search returns zero results, the UX must help the user recover.

### Zero-Result UX Hierarchy

```
No results found for "[query]"

1. Did you mean: [spelling suggestion]?     ← Fuzzy match
2. Related topics:                           ← Semantic fallback
   - [Related page 1]
   - [Related page 2]
3. Popular pages:                            ← Popularity fallback
   - Getting Started
   - API Reference
4. Still stuck?                              ← Human fallback
   - Ask in our community forum
   - Contact support
```

### Zero-Result Recovery Rules

| Recovery Level | Trigger | Implementation |
|----------------|---------|---------------|
| Spelling correction | Edit distance <= 2 | "Did you mean [corrected]?" |
| Semantic fallback | Zero exact matches | Show pages with related terms |
| Category suggestion | Query matches a section name | "Browse [section]" |
| Popular pages | All else fails | Show top 5 most-visited pages |
| Human fallback | Always present | Link to community/support |

## Search Analytics Feedback Loop

### Metrics to Track

| Metric | What It Reveals | Target |
|--------|----------------|--------|
| Query volume | Overall search reliance | Track trend, not target |
| Zero-result rate | Content or synonym gaps | < 5% |
| Search-to-click ratio | Result relevance | > 80% |
| Click position | How far users scroll | > 70% click position 1-3 |
| Search exit rate | Users who leave after search | < 20% |
| Refinement rate | Users who modify their query | < 30% |

### Feedback Loop Process

```
     Track Queries
          │
          ▼
     Identify Zero-Results
          │
          ├──→ Content Gap? ──→ Create missing page
          │
          ├──→ Synonym Gap? ──→ Add synonym mapping
          │
          └──→ Ranking Issue? ──→ Adjust relevance weights
          │
          ▼
     Measure Impact (next period)
          │
          └──→ Repeat quarterly
```

### Weekly Search Report Template

```markdown
## Search Health Report: Week of [date]

**Query Volume**: [count] (+/- [%] vs last week)
**Zero-Result Rate**: [%] (target: < 5%)
**Search-to-Click**: [%] (target: > 80%)

### Top 10 Queries
| Rank | Query | Count | Clicked | Result Position |
|------|-------|-------|---------|-----------------|
| 1 | [query] | [n] | [%] | [avg position] |

### Top Zero-Result Queries
| Query | Count | Diagnosis | Action |
|-------|-------|-----------|--------|
| [query] | [n] | Content gap / Synonym / Typo | [action] |

### Actions Taken
- Added synonym: [X] → [Y]
- Created page: [title]
- Updated ranking for: [query]
```

## Audit Checklist

### Search UX
- [ ] CMD+K keyboard shortcut works
- [ ] Search bar visible in top navigation
- [ ] Results appear within 100ms
- [ ] Results show title, snippet, and breadcrumb path
- [ ] Arrow key navigation works in results
- [ ] Esc closes search modal
- [ ] Results grouped by section or content type

### Relevance
- [ ] Title matches rank highest
- [ ] Heading matches rank above body text
- [ ] Search-to-click ratio > 80%
- [ ] Most clicks on positions 1-3
- [ ] Zero-result rate < 5%

### Synonyms
- [ ] Synonym dictionary exists
- [ ] Common abbreviations are mapped
- [ ] Competitor terms are mapped
- [ ] Dictionary is reviewed quarterly

### Zero-Result Recovery
- [ ] Spelling suggestions shown for typos
- [ ] Related topics shown for zero results
- [ ] Popular pages shown as fallback
- [ ] Link to community/support always present

### Analytics
- [ ] Search queries are logged
- [ ] Click-through is tracked per query
- [ ] Zero-result queries are flagged
- [ ] Weekly/monthly search report exists

## Output Format

### Search & Discovery Audit: `{site or scope}`

**Search UX Assessment**:
| Feature | Status | Notes |
|---------|--------|-------|
| CMD+K pattern | Present/Missing | [note] |
| Result quality | Good/Needs Work | [note] |
| Scoped search | Present/Missing | [note] |
| Zero-result UX | Present/Missing | [note] |

**Relevance Metrics**:
| Metric | Value | Rating |
|--------|-------|--------|
| Search-to-click | [%] | [rating] |
| Zero-result rate | [%] | [rating] |
| Click position avg | [pos] | [rating] |

**Top Issues**:
1. [Issue] → [Fix]
2. [Issue] → [Fix]

## Guidelines

- **DO**: Implement CMD+K as the primary search entry point
- **DO**: Maintain a synonym dictionary updated from zero-result queries
- **DO**: Show breadcrumb paths in search results for context
- **DON'T**: Return only "No results found" with no recovery options
- **DON'T**: Ignore search analytics — they reveal content gaps
- **DON'T**: Rank blog posts equally with core documentation

## Related Components

- **auditing-docs-metrics skill**: Defines search-to-click and zero-result KPIs
- **governing-taxonomy skill**: Taxonomy labels feed into search facets
- **designing-navigation-systems skill**: Navigation for link-dominant users
- **governing-docs-lifecycle skill**: Stale pages should be deprioritized in search
