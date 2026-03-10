---
name: analyzing-docs-benchmarks
version: 1.0.0
description: Benchmark developer documentation sites against industry leaders (Stripe, Vercel, Kubernetes, AWS) using structured pattern extraction, innovation catalogs, anti-pattern catalogs, and scorecard templates. Use when comparing docs quality, extracting best practices from elite docs sites, or building a competitive docs analysis. Triggered by: docs benchmark, benchmark analysis, Stripe docs, Vercel docs, Kubernetes docs, AWS docs, docs comparison, best practices, innovation catalog, anti-pattern, docs scorecard, competitive docs, docs patterns, industry standard, docs quality comparison, elite docs.
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Analyzing Documentation Benchmarks

## Purpose

Perform structured benchmark analysis of developer documentation sites against industry leaders. This skill provides pattern extraction frameworks, scoring templates, and catalogs of innovations and anti-patterns observed across elite documentation sites. The output informs documentation strategy decisions by showing what best-in-class looks like.

This is a read-only analysis skill. It produces assessment documents but does not modify documentation (handled by docs-designer and other writing-focused skills).

## Benchmark Reference Sites

### Tier 1: Gold Standard

These sites consistently score highest across all dimensions:

| Site | Known For | Key Innovation |
|------|-----------|---------------|
| **Stripe** | API reference, interactive examples | "Try it" panels alongside every endpoint |
| **Vercel** | Getting started speed, framework guides | Framework-specific onboarding paths |
| **Twilio** | Code samples in 6+ languages | Language selector persists across pages |
| **MDN Web Docs** | Comprehensive reference, community | Browser compatibility tables |

### Tier 2: Strong Specific Dimensions

| Site | Excels At | Notable Pattern |
|------|-----------|----------------|
| **Kubernetes** | Concept explanations, architecture | Concept → Task → Reference layering |
| **AWS** | Comprehensive coverage, depth | Service-scoped docs with consistent template |
| **Tailwind CSS** | Search, utility reference | Instant search with visual previews |
| **Next.js** | Interactive tutorials, App Router | Interactive code playground in docs |
| **Supabase** | Onboarding, quickstarts | Framework-specific quickstart matrix |
| **Prisma** | Progressive disclosure, schema reference | Collapsible deep-dives within pages |

### Tier 3: Aspirational Patterns

| Site | Pattern Worth Studying |
|------|----------------------|
| **Linear** | Changelog as documentation, living changelog |
| **Resend** | Minimal docs with maximum clarity |
| **Railway** | Template-based getting started |
| **Planetscale** | Database branching explained visually |

## Benchmark Dimensions

Evaluate each docs site across 8 dimensions:

### 1. Onboarding Speed

How fast can a new developer go from zero to working code?

| Score | Criteria |
|-------|----------|
| 5 | < 5 minutes to working code, copy-paste quickstart |
| 4 | 5-10 minutes, clear steps, minimal prerequisites |
| 3 | 10-20 minutes, some ambiguity in setup |
| 2 | 20-30 minutes, significant prerequisites or friction |
| 1 | > 30 minutes, unclear starting point |

**What to extract:**
- Time to first working code (TTFS)
- Number of steps in quickstart
- Prerequisites count and clarity
- Whether output is shown alongside code

### 2. Navigation Clarity

How easily can users find what they need?

| Score | Criteria |
|-------|----------|
| 5 | Three-pane layout, < 20 sidebar items, cmd+k search, breadcrumbs |
| 4 | Good sidebar organization, working search, some breadcrumbs |
| 3 | Adequate sidebar, basic search, no breadcrumbs |
| 2 | Cluttered sidebar, poor search, confusing hierarchy |
| 1 | No clear navigation structure, search broken or absent |

**What to extract:**
- Layout pattern (three-pane, two-pane, single)
- Sidebar item count and depth
- Search quality and features
- Breadcrumb implementation

### 3. Content Architecture

How well is content categorized and structured?

| Score | Criteria |
|-------|----------|
| 5 | Clear Diataxis separation, consistent templates, logical hierarchy |
| 4 | Mostly separated content types, good hierarchy |
| 3 | Some mixing of content types, adequate structure |
| 2 | Significant mixing, unclear hierarchy |
| 1 | No discernible content architecture |

**What to extract:**
- Content type separation (tutorials vs guides vs reference)
- Template consistency across pages
- Cross-referencing between content types
- Taxonomy model (task-based, topic-based, hybrid)

### 4. Code Sample Quality

How useful and complete are code examples?

| Score | Criteria |
|-------|----------|
| 5 | Multi-language, copy-button, runnable, output shown, syntax highlighted |
| 4 | Primary language + 1-2 alternatives, copy-button, mostly complete |
| 3 | Single language, complete examples, syntax highlighting |
| 2 | Incomplete snippets, missing context, no copy button |
| 1 | Pseudocode, broken examples, no highlighting |

**What to extract:**
- Languages supported per example
- Copy button presence
- Whether examples are self-contained
- Output/result shown alongside code
- Interactive playground availability

### 5. Reference Completeness

How thorough is the API/SDK reference?

| Score | Criteria |
|-------|----------|
| 5 | Every endpoint/method documented, parameters typed, examples for all, error codes listed |
| 4 | Most endpoints documented, parameters listed, some examples |
| 3 | Core endpoints documented, basic parameter info |
| 2 | Partial coverage, missing parameters or types |
| 1 | Minimal or auto-generated without review |

**What to extract:**
- Coverage percentage (documented vs total endpoints)
- Parameter documentation completeness
- Error code documentation
- Request/response examples
- Authentication documentation

### 6. Search & Discovery

How effective is the search experience?

| Score | Criteria |
|-------|----------|
| 5 | Cmd+K, instant results, faceted, synonyms, zero-result recovery |
| 4 | Keyboard shortcut, fast results, basic faceting |
| 3 | Working search bar, adequate results |
| 2 | Search exists but slow or irrelevant results |
| 1 | No search or broken search |

**What to extract:**
- Search activation pattern (cmd+k, search bar)
- Result speed and quality
- Faceting/scoping capabilities
- Zero-result handling
- Synonym handling

### 7. Visual & Interactive Design

How polished is the visual presentation?

| Score | Criteria |
|-------|----------|
| 5 | Custom design system, dark mode, interactive elements, diagrams, consistent tokens |
| 4 | Polished design, dark mode, some interactive elements |
| 3 | Clean but template-based, adequate design |
| 2 | Basic styling, inconsistent design |
| 1 | Unstyled or broken design |

**What to extract:**
- Dark mode support
- Diagram/illustration usage
- Interactive elements (playgrounds, live examples)
- Visual consistency
- Mobile responsiveness

### 8. Governance Signals

Evidence of ongoing maintenance and governance.

| Score | Criteria |
|-------|----------|
| 5 | Version indicators, last-updated dates, changelog, no stale content visible |
| 4 | Version info, mostly current content, some dates |
| 3 | Adequate currency, some dated content |
| 2 | Noticeable stale content, no dates |
| 1 | Obviously outdated content, broken examples, dead links |

**What to extract:**
- Last-updated timestamps
- Version indicators
- Broken link prevalence
- Stale content visibility
- Changelog or "What's New" presence

## Benchmark Scorecard Template

```markdown
## Documentation Benchmark Scorecard

**Date**: [date]
**Evaluated by**: [evaluator]

| Dimension | Our Docs | Stripe | Vercel | [Comp 1] | [Comp 2] |
|-----------|----------|--------|--------|----------|----------|
| 1. Onboarding Speed | /5 | /5 | /5 | /5 | /5 |
| 2. Navigation Clarity | /5 | /5 | /5 | /5 | /5 |
| 3. Content Architecture | /5 | /5 | /5 | /5 | /5 |
| 4. Code Sample Quality | /5 | /5 | /5 | /5 | /5 |
| 5. Reference Completeness | /5 | /5 | /5 | /5 | /5 |
| 6. Search & Discovery | /5 | /5 | /5 | /5 | /5 |
| 7. Visual & Interactive | /5 | /5 | /5 | /5 | /5 |
| 8. Governance Signals | /5 | /5 | /5 | /5 | /5 |
| **TOTAL** | **/40** | **/40** | **/40** | **/40** | **/40** |

### Score Interpretation
| Range | Rating |
|-------|--------|
| 36-40 | World-class documentation |
| 28-35 | Strong documentation, minor gaps |
| 20-27 | Adequate, significant improvement areas |
| 12-19 | Below standard, major gaps |
| 0-11 | Critical issues, documentation needs overhaul |
```

## Innovation Catalog

Patterns worth adopting from benchmark sites:

| Innovation | Source | Description | Complexity |
|------------|--------|-------------|------------|
| Interactive API explorer | Stripe | Try API calls directly in docs with real responses | High |
| Language persistence | Twilio | Selected language persists across all code examples | Medium |
| Framework-specific quickstarts | Vercel, Supabase | Matrix of quickstarts per framework | Medium |
| Browser compatibility tables | MDN | Per-feature browser support grid | Medium |
| Visual schema explorer | Prisma | Interactive schema visualization | High |
| Copy button on all code blocks | Common | One-click copy for every code block | Low |
| Dark mode toggle | Common | User-selectable light/dark theme | Low |
| "Edit this page" link | Common | GitHub link to edit source directly | Low |
| Version selector | AWS, K8s | Switch docs version in top bar | Medium |
| Collapsible deep-dives | Prisma | Expandable sections for advanced details | Low |
| Live code playground | Next.js | Edit and run code within docs | High |
| API status indicator | Stripe | Live API status in docs header | Low |

## Anti-Pattern Catalog

Patterns to avoid, observed in real documentation:

| Anti-Pattern | Observed At | Problem | Fix |
|--------------|-------------|---------|-----|
| Wall of text quickstart | Various | Users bounce before step 1 | Numbered steps with code blocks |
| Auto-generated reference only | Various | No context, no examples | Add usage examples and descriptions |
| Single mega-page | Various | Scroll fatigue, poor findability | Split into focused pages |
| Stale "last updated 2 years ago" | Various | Erodes trust | Remove dates or keep current |
| Login wall before docs | Various | Blocks evaluation | Public docs, auth for advanced features only |
| Broken playground | Various | Worse than no playground | Remove or fix, don't ship broken |
| PDF-only docs | Legacy tools | Not searchable, not linkable | Convert to web-based docs |
| Separate API docs site | Various | Context switching, different search | Integrate into main docs |
| Undocumented errors | Various | Users stuck with opaque errors | Document all error codes |
| "Contact us" instead of docs | Various | Not self-service | Write the content |

## Instructions

When performing a benchmark analysis:

1. **Select benchmark set**: Choose 2-3 direct competitors + 2 aspirational sites from the reference list
2. **For each site**, evaluate across all 8 dimensions using the scoring criteria
3. **Fill in the scorecard** with scores and notes
4. **Identify innovations** from the innovation catalog that could be adopted
5. **Check for anti-patterns** in your own docs against the anti-pattern catalog
6. **Produce a gap analysis**: Where do you score lowest relative to the benchmark set?
7. **Prioritize improvements**: What would have the highest impact relative to effort?

### Prioritization Matrix

| Impact | Low Effort | Medium Effort | High Effort |
|--------|-----------|---------------|-------------|
| **High** | Do immediately | Plan this quarter | Plan next quarter |
| **Medium** | Do this sprint | Plan this quarter | Evaluate ROI |
| **Low** | Nice to have | Backlog | Skip |

## Output Format

### Benchmark Analysis: `{docs site name}`

**Scorecard Summary**:
[Filled scorecard table]

**Key Findings**:
1. [Strongest dimension and why]
2. [Weakest dimension and why]
3. [Biggest gap vs benchmark set]

**Innovations to Adopt**:
| Innovation | Source | Impact | Effort | Priority |
|------------|--------|--------|--------|----------|
| [innovation] | [site] | High/Med/Low | High/Med/Low | [priority] |

**Anti-Patterns Found**:
| Anti-Pattern | Location | Severity | Fix |
|--------------|----------|----------|-----|
| [pattern] | [where] | High/Med/Low | [fix] |

**Recommended Roadmap**:
1. **This sprint**: [quick wins]
2. **This quarter**: [medium effort, high impact]
3. **Next quarter**: [high effort improvements]

## Guidelines

- **DO**: Score against the specific criteria for each dimension (don't estimate)
- **DO**: Include at least one aspirational site outside your direct category
- **DO**: Check your own docs for anti-patterns, not just competitors
- **DON'T**: Score based on memory — visit the live site
- **DON'T**: Copy features without understanding the underlying design decision
- **DON'T**: Try to implement everything at once — prioritize by impact/effort

## Related Components

- **designing-landing-pages skill**: Improve onboarding speed dimension
- **designing-navigation-systems skill**: Improve navigation clarity dimension
- **designing-search-discovery skill**: Improve search & discovery dimension
- **auditing-docs-metrics skill**: Quantify your own docs performance for comparison
- **governing-docs-lifecycle skill**: Improve governance signals dimension
