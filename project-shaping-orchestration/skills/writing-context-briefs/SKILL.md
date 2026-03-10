---
name: shaping-writing-context-briefs
version: 1.0.0
user-invocable: false
description: Write context brief documents that organize raw inputs into structured facts, references, glossary, assumptions, and unknowns. Use when curating source material, gathering facts, building glossaries, cataloging assumptions, identifying unknowns, summarizing raw inputs, or preparing context for downstream shaping work. Triggered by: context brief, raw inputs, facts gathering, glossary, assumptions, unknowns, source material, context curation.
allowed-tools:
  - Read
---

# Context Briefs — Facts, References, Glossary, Assumptions & Unknowns

Expert guidance for producing context brief artifacts during the shaping process.

## Purpose

A context brief is the first artifact in the shaping pipeline. It transforms noisy, scattered raw inputs — conversations, documents, feature requests, support tickets, meeting notes, and reference material — into a structured, verifiable foundation that downstream agents (Problem Framer, Domain Proxy, Technical Shaper) can rely on without re-reading the original sources.

The context brief exists because every subsequent shaping decision is only as good as the facts it rests on. By separating confirmed facts from assumptions and surfacing unknowns early, the brief prevents the most expensive class of shaping error: building a solution to a misunderstood problem.

Write a context brief whenever new shaping work begins, when the problem space is unfamiliar, or when raw inputs arrive from multiple sources that need reconciliation before framing can start.

---

## Output Format

The context brief follows a fixed six-section structure. Every section is mandatory even if its content is minimal. Omitting a section signals the brief is incomplete.

```markdown
# Context Brief: <Topic Title>

**Prepared by:** <Agent or Author>
**Date:** <YYYY-MM-DD>
**Status:** Draft | Under Review | Accepted
**Input Sources:** <comma-separated list of source types consumed>

---

## 1. Verified Facts

Confirmed, source-backed facts relevant to the problem space. Each fact is a single declarative sentence with an inline source citation.

1. <Fact statement>. _Source: <document name, section, URL, or conversation reference>_
2. <Fact statement>. _Source: <reference>_
3. <Fact statement>. _Source: <reference>_
...

---

## 2. Reference Links

External and internal references consulted during context curation.

| # | Name | URL | Relevance | Last Verified |
|---|------|-----|-----------|---------------|
| 1 | <Resource name> | <URL or file path> | <One sentence on why this matters> | <YYYY-MM-DD> |
| 2 | <Resource name> | <URL or file path> | <Relevance statement> | <YYYY-MM-DD> |
...

---

## 3. Glossary

Domain-specific vocabulary encountered in the raw inputs. Definitions must be precise enough that someone outside the domain can use the term correctly.

| Term | Definition | Source |
|------|-----------|--------|
| <Term> | <Clear, concise definition in plain language> | <Where this term was defined or used> |
| <Term> | <Definition> | <Source> |
...

---

## 4. Assumptions

Statements treated as true for purposes of this shaping effort but not yet verified. Each includes a risk statement describing the consequence if the assumption is wrong.

1. **<Assumption statement>.**
   _Risk if wrong: <What breaks, changes, or must be revisited if this is false>_
2. **<Assumption statement>.**
   _Risk if wrong: <Consequence>_
...

---

## 5. Unknowns

Questions that could not be answered from the available inputs. Each includes a suggested approach for resolving the unknown.

1. **<Question or knowledge gap>**
   _Suggested resolution: <Who to ask, what to research, or what experiment to run>_
2. **<Question or knowledge gap>**
   _Suggested resolution: <Approach>_
...

---

## 6. Raw Input Summary

A condensed, faithful summary of the original inputs. Preserves the intent and key details of each source without editorializing. Organized by source.

### Source: <Source name or type>
<2-5 sentence summary of this source's key content and intent>

### Source: <Source name or type>
<2-5 sentence summary>

...
```

---

## Instructions

Follow these five steps in order. Do not skip steps or reorder them.

1. **Read all raw inputs thoroughly.** Consume every provided document, conversation excerpt, link, note, and reference. Do not skim. Read each source completely before beginning extraction. If a source is a URL, note its content and last-verified date. If a source is a conversation, identify the participants and their roles.

2. **Extract and classify facts.** Walk through each source sentence by sentence. For each claim, determine whether it is a verifiable fact (can be confirmed from the source) or an assumption (stated or implied but not backed by evidence). Record verified facts with precise source attribution. Be strict: if a statement cannot be traced to a specific source, it is an assumption, not a fact.

3. **Build the glossary from domain-specific terms.** Identify every term that a general audience might not understand or that has a specific meaning within this problem domain. Define each term in plain language. If two sources use different terms for the same concept, note the preferred term and list alternatives in the definition. If a term is used inconsistently across sources, flag this as an unknown.

4. **Catalog assumptions with risk assessments.** For each assumption, articulate what would change in the shaping effort if the assumption turns out to be false. Rank assumptions by the severity of their risk-if-wrong. High-severity assumptions (those that would invalidate the problem frame or require a fundamentally different approach) should appear first.

5. **Identify unknowns and propose resolution paths.** For each unknown, suggest a concrete next step: a person to consult, a document to locate, a prototype to build, or a metric to measure. Unknowns that block downstream shaping work (problem framing or solution sketching) should be flagged as blocking and listed first.

---

## Quality Checklist

Before considering the context brief complete, verify every condition below.

- [ ] Every fact in Verified Facts has an explicit, traceable source citation
- [ ] The glossary covers every domain-specific term found in the raw inputs, with no jargon left undefined
- [ ] Every assumption includes a concrete risk-if-wrong statement, not a vague "might cause issues"
- [ ] Every unknown includes a specific, actionable resolution approach, not "needs further investigation"
- [ ] The Raw Input Summary is faithful to the original sources — no new claims, interpretations, or opinions were added
- [ ] No fact appears in both Verified Facts and Assumptions — each statement is classified as one or the other
- [ ] The Reference Links table includes every external source consulted, with a last-verified date

---

## Anti-Patterns

### 1. Editorializing in the Summary
The Raw Input Summary restates what the sources say, not what the curator thinks about them. Adding opinions, recommendations, or evaluations turns the brief into an advocacy document rather than a neutral foundation. Save interpretation for the Problem Frame.

### 2. Vague Source Attribution
Citing "from a conversation" or "per the team" is insufficient. Each fact must link to a specific document, message, meeting, or person so that reviewers can verify it. If a fact cannot be attributed precisely, demote it to an assumption.

### 3. Missing Risk Assessments on Assumptions
Listing assumptions without risk-if-wrong statements defeats the purpose. The risk assessment is what tells downstream agents which assumptions to validate first. An assumption without a risk statement is just a guess with no priority signal.

### 4. Conflating Facts and Assumptions
Treating "the team seems to want X" as a verified fact inflates confidence in the brief. The distinction between confirmed and assumed is the brief's primary value. When in doubt, classify as an assumption — it is always safer to verify a claim than to act on a false certainty.

### 5. Ignoring Contradictions Between Sources
When two sources disagree, the brief must surface the contradiction rather than silently choosing one version. Record the conflicting claims as separate facts with their respective sources, and add an unknown asking which version is authoritative. Contradictions that are buried become wrong facts.

---

## Examples

### Example 1: API Rate Limiting Context Brief

A product manager shares a support ticket about customers hitting rate limits, a Slack thread where engineers debated throttling strategies, and a link to the current rate-limiting documentation.

**How the artifact looks in practice (abbreviated):**

```markdown
# Context Brief: API Rate Limiting Concerns

**Prepared by:** Context Curator
**Date:** 2026-02-15
**Status:** Draft
**Input Sources:** Support ticket #4821, Slack thread #eng-platform 2026-02-12, Rate Limiting Docs v2.3

---

## 1. Verified Facts

1. Customer Acme Corp received HTTP 429 responses 847 times in the past 7 days. _Source: Support ticket #4821, attached logs_
2. The current global rate limit is 1,000 requests per minute per API key. _Source: Rate Limiting Docs v2.3, Section 3.1_
3. Three engineers proposed per-endpoint rate limiting as an alternative to global limits. _Source: Slack thread #eng-platform, messages from 2026-02-12_

## 4. Assumptions

1. **Acme Corp's traffic pattern is representative of other high-volume customers.**
   _Risk if wrong: A solution tuned for Acme's access pattern may not help other customers, requiring per-customer analysis._
2. **The current rate-limiting infrastructure can support per-endpoint configuration without architectural changes.**
   _Risk if wrong: The effort moves from a configuration change to a platform rebuild, exceeding the shaping appetite._

## 5. Unknowns

1. **What percentage of 429 responses result in customer-visible errors versus silent retries?**
   _Suggested resolution: Ask the SDK team whether the official client libraries implement automatic retry with backoff._
```

### Example 2: Onboarding Flow Redesign Context Brief

A designer shares user research interview transcripts, a product brief outlining conversion goals, and analytics data showing drop-off rates in the current onboarding funnel.

**How the artifact looks in practice (abbreviated):**

```markdown
# Context Brief: Onboarding Flow Redesign

**Prepared by:** Context Curator
**Date:** 2026-02-10
**Status:** Under Review
**Input Sources:** User research transcripts (5 interviews), Product brief v1.2, Funnel analytics export Jan 2026

---

## 1. Verified Facts

1. 62% of new signups abandon onboarding before completing the workspace setup step. _Source: Funnel analytics, Jan 2026 cohort_
2. 4 of 5 interview participants described the workspace concept as "confusing" or "unclear." _Source: User research transcripts, interviews 1, 2, 3, 5_
3. The product brief targets a 40% completion rate for the full onboarding flow by Q2 2026. _Source: Product brief v1.2, Success Metrics section_

## 3. Glossary

| Term | Definition | Source |
|------|-----------|--------|
| Workspace | A container that groups projects, members, and settings under a single billing entity | Product brief v1.2 |
| Activation event | The first action a new user takes that correlates with long-term retention (currently: creating a second project) | Funnel analytics definitions |

## 5. Unknowns

1. **Do users who abandon at the workspace step ever return, or are they permanently lost?**
   _Suggested resolution: Query the analytics warehouse for 30-day return rates among users who dropped off at the workspace step._
2. **Is the "workspace" concept itself the problem, or just its presentation in the current UI?**
   _Suggested resolution: Run a concept test with 3-5 users using a simplified explanation of workspaces before redesigning the flow._
```