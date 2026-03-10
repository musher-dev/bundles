---
name: auditing-docs-metrics
version: 1.0.0
description: Audit developer documentation using KPIs including Time to First Success (TTFS), search-to-click ratio, zero-result rate, bounce rate, deflection rate, and usability testing protocols. Use when measuring docs effectiveness, designing measurement frameworks, running usability tests, or analyzing documentation quality metrics. Triggered by: docs metrics, TTFS, time to first success, search analytics, bounce rate, deflection, usability testing, NASA-TLX, docs KPI, measurement, docs quality, search-to-click, zero-result rate, documentation effectiveness, docs audit metrics.
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Auditing Documentation Metrics

## Purpose

Define, measure, and audit KPIs for developer documentation effectiveness. Good docs are measurable — this skill provides the metrics framework, measurement methods, and usability testing protocols to evaluate whether documentation is actually helping users succeed.

This is a read-only auditing skill. It defines what to measure and how to interpret results, but does not modify documentation content (handled by docs-designer and concept-designer).

## Primary KPIs

### 1. Time to First Success (TTFS)

The most important documentation metric: how long from first docs visit to completing the getting-started tutorial.

| Rating | TTFS | Interpretation |
|--------|------|---------------|
| Excellent | < 5 minutes | Docs are highly optimized |
| Good | 5-15 minutes | Standard for moderate-complexity products |
| Needs Work | 15-30 minutes | Friction points exist |
| Critical | > 30 minutes | Docs are actively harming adoption |

**How to Measure:**
1. Instrument the getting-started tutorial with start/end events
2. Track page-to-page navigation timing
3. Measure time from first docs pageview to tutorial completion event
4. Segment by: new vs returning, referral source, device

**TTFS Decomposition:**
```
TTFS = T(find) + T(read) + T(execute) + T(verify)

T(find)    = Time to locate the getting-started page
T(read)    = Time spent reading before first action
T(execute) = Time spent executing steps
T(verify)  = Time to confirm success
```

Each component reveals different problems:
- High T(find): Landing page routing is unclear
- High T(read): Prerequisites or context is missing
- High T(execute): Steps are unclear or error-prone
- High T(verify): Verification section is inadequate

### 2. Search-to-Click Ratio

What percentage of searches result in a click on a search result.

| Rating | Ratio | Interpretation |
|--------|-------|---------------|
| Excellent | > 80% | Search results are relevant |
| Good | 60-80% | Mostly relevant, some gaps |
| Needs Work | 40-60% | Significant relevance gaps |
| Critical | < 40% | Search is broken or content is missing |

**What Low Ratios Mean:**
- Results don't match user intent → Fix search ranking or synonyms
- Content doesn't exist for the query → Content gap to fill
- Result titles/snippets are unclear → Fix page titles and descriptions

### 3. Zero-Result Rate

What percentage of searches return zero results.

| Rating | Rate | Interpretation |
|--------|------|---------------|
| Excellent | < 5% | Content coverage is comprehensive |
| Good | 5-10% | Minor gaps |
| Needs Work | 10-20% | Significant content gaps |
| Critical | > 20% | Major content or synonym gaps |

**Remediation:**
1. Log all zero-result queries
2. Categorize: content gap vs. synonym gap vs. typo
3. For content gaps: prioritize by query frequency
4. For synonym gaps: add to search synonym dictionary
5. For typos: implement fuzzy matching

### 4. Bounce Rate (by Page Type)

| Page Type | Acceptable Bounce | High Bounce Signal |
|-----------|-------------------|-------------------|
| Landing page | < 40% | Routing is unclear |
| Tutorial | < 30% | Content doesn't match expectations |
| How-to guide | < 50% | User found answer quickly (may be good) |
| Reference | < 60% | User looked up one thing (expected) |
| Explanation | < 50% | Content isn't engaging |

**Bounce Rate Nuance:** High bounce on reference pages is normal (user looked up a parameter and left). High bounce on tutorials is a problem (user gave up).

### 5. Deflection Rate

What percentage of support tickets are avoided because docs answered the question.

| Rating | Rate | Interpretation |
|--------|------|---------------|
| Excellent | > 70% | Docs handle most common questions |
| Good | 50-70% | Docs cover the basics well |
| Needs Work | 30-50% | Significant gaps in troubleshooting docs |
| Critical | < 30% | Docs are not serving as self-service |

**How to Measure:**
1. Compare top support ticket topics to existing docs coverage
2. Track "Did this help?" widget responses
3. Monitor support ticket volume after publishing new docs
4. Track search-to-support-ticket funnel

## Usability Testing Protocol

### Test Types

| Test Type | What It Measures | Task Example |
|-----------|-----------------|--------------|
| **Foraging** | Can users find the right page? | "Find how to configure SSO" |
| **Instructional** | Can users follow the steps? | "Complete the getting-started tutorial" |
| **Troubleshooting** | Can users solve a problem? | "Your deploy failed with error X. Fix it." |

### Test Design

**Participants:** 5 users per round (diminishing returns beyond 5 for usability testing)

**Recruitment:**
- 2 beginners (never used the product)
- 2 intermediate (used the product but not this feature)
- 1 expert (power user, tests for expertise reversal)

### Foraging Tasks

Test whether users can navigate to the right content.

| Task Structure | Example |
|----------------|---------|
| "Find the page that explains [concept]" | "Find the page that explains how authentication works" |
| "You want to [task]. Where would you start?" | "You want to deploy to AWS. Where would you start?" |

**Metrics:**
- Success rate (found the page)
- Time to find
- Path taken (click trail)
- Wrong turns (visited irrelevant pages)

### Instructional Tasks

Test whether users can follow documentation to complete a task.

| Task Structure | Example |
|----------------|---------|
| "Follow the tutorial to [outcome]" | "Follow the quickstart to deploy your first app" |
| "Using the docs, [perform task]" | "Using the docs, add a team member to your workspace" |

**Metrics:**
- Completion rate
- Time to complete
- Errors encountered
- Steps where users got stuck
- Whether users read prerequisites

### Troubleshooting Tasks

Test whether users can solve problems using docs.

| Task Structure | Example |
|----------------|---------|
| "You see this error: [error]. Fix it." | "You see 'Error: Invalid API key'. Fix it using the docs." |
| "[Scenario]. What would you do?" | "Your deploy is stuck at 'Pending'. What would you do?" |

**Metrics:**
- Resolution rate
- Time to resolution
- Resources consulted (search, sidebar, external)
- Whether error message text was searchable

### NASA-TLX (Task Load Index)

After each task, measure perceived cognitive load using the NASA-TLX scale:

| Dimension | Question | Scale |
|-----------|----------|-------|
| Mental Demand | "How mentally demanding was this task?" | 1-7 (Low-High) |
| Temporal Demand | "How hurried or rushed did you feel?" | 1-7 (Low-High) |
| Performance | "How successful were you?" | 1-7 (Failure-Perfect) |
| Effort | "How hard did you have to work?" | 1-7 (Low-High) |
| Frustration | "How frustrated were you?" | 1-7 (Low-High) |

**Interpreting Results:**
- Average score > 5 on any dimension: task needs attention
- High Mental Demand + Low Performance: content is unclear
- High Frustration + High Effort: navigation or findability issue
- High Temporal Demand: too many steps or prerequisites

## Measurement Dashboard Template

```
┌─────────────────────────────────────────────────────────┐
│           DOCUMENTATION HEALTH DASHBOARD                 │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  TTFS              Search-to-Click    Zero-Result Rate   │
│  ┌──────┐          ┌──────┐           ┌──────┐          │
│  │ 8min │ GOOD     │ 72%  │ GOOD      │  8%  │ GOOD    │
│  └──────┘          └──────┘           └──────┘          │
│                                                          │
│  Bounce (Tutorials) Deflection Rate   NASA-TLX Avg      │
│  ┌──────┐          ┌──────┐           ┌──────┐          │
│  │ 25%  │ GOOD     │ 55%  │ GOOD      │ 3.2  │ GOOD    │
│  └──────┘          └──────┘           └──────┘          │
│                                                          │
│  Trend: ↑ TTFS improving, ↓ Zero-results declining      │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

## Audit Process

### Step 1: Inventory Available Data

Identify which metrics are currently being collected:
- [ ] Page analytics (views, bounce, time on page)
- [ ] Search analytics (queries, clicks, zero-results)
- [ ] Tutorial completion tracking
- [ ] Support ticket categorization
- [ ] User feedback widgets

### Step 2: Compute KPIs

Calculate each KPI from available data:
1. TTFS (if instrumented) or estimate from page timing data
2. Search-to-click ratio from search analytics
3. Zero-result rate from search logs
4. Bounce rate by page type from analytics
5. Deflection rate from support ticket analysis

### Step 3: Rate Each KPI

Apply the rating tables from this document.

### Step 4: Identify Priority Issues

| KPI Rating | Priority |
|------------|----------|
| Critical on any KPI | P0 — fix immediately |
| Needs Work on TTFS or Search | P1 — fix this quarter |
| Needs Work on other KPIs | P2 — plan improvement |
| Good on all KPIs | Maintain and refine |

### Step 5: Recommend Usability Testing

If data gaps exist, recommend specific usability test types:
- Can't measure findability? → Foraging tasks
- Can't measure tutorial quality? → Instructional tasks
- High support volume? → Troubleshooting tasks

## Output Format

### Documentation Metrics Audit: `{scope}`

**Data Availability**:
| Metric | Available | Source | Quality |
|--------|-----------|--------|---------|
| Page analytics | Yes/No | [tool] | High/Medium/Low |
| Search analytics | Yes/No | [tool] | High/Medium/Low |
| Tutorial tracking | Yes/No | [tool] | High/Medium/Low |
| Support tickets | Yes/No | [tool] | High/Medium/Low |

**KPI Summary**:
| KPI | Value | Rating | Trend |
|-----|-------|--------|-------|
| TTFS | [value] | [rating] | [up/down/flat] |
| Search-to-Click | [value] | [rating] | [up/down/flat] |
| Zero-Result Rate | [value] | [rating] | [up/down/flat] |
| Bounce (Tutorials) | [value] | [rating] | [up/down/flat] |
| Deflection | [value] | [rating] | [up/down/flat] |

**Priority Issues**:
1. [P0/P1/P2]: [Issue] → [Recommended action]

**Recommended Testing**:
- [Test type]: [Rationale]

## Guidelines

- **DO**: Decompose TTFS into sub-components to find specific friction points
- **DO**: Segment metrics by page type (tutorial bounce vs reference bounce)
- **DO**: Use NASA-TLX to capture perceived cognitive load
- **DON'T**: Treat high bounce on reference pages as a problem
- **DON'T**: Test with more than 5 users per round (diminishing returns)
- **DON'T**: Conflate search problems with content problems — diagnose separately

## Related Components

- **designing-search-discovery skill**: Fixes search-related metric issues
- **designing-landing-pages skill**: Fixes T(find) component of TTFS
- **docs-designer agent**: Fixes T(read) and T(execute) components via page quality
- **governing-docs-lifecycle skill**: Tracks stale content that degrades metrics
