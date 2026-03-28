# Developer Docs Authoring

Design, scaffold, and audit developer documentation — from navigation architecture and page structure to content quality scoring and accessibility compliance. This bundle covers the full documentation lifecycle with specialized agents for structure, clarity, scaffolding, quality gating, LLM indexing, and site-wide architecture analysis.

## Quick Start

Install via the [Mush CLI](https://github.com/musher-dev/mush):

```sh
mush bundle install musher-dev/developer-docs-authoring
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Audit our developer docs site — check navigation, content quality, accessibility, and coverage gaps
```

## What's Inside

The bundle ships six agents and fifteen skills that cover documentation design, generation, and quality assurance.

The **docs-designer** agent is the primary workhorse — it audits and designs technical documentation for structure, clarity, user empathy, and Diataxis compliance. The **concept-designer** focuses specifically on terminology consistency and cognitive load, ensuring primitives are explained clearly and concepts build on each other.

The **page-scaffolder** agent generates structurally correct pages from Diataxis-aligned archetypes (Quickstart, How-To, API Reference, Webhooks, Troubleshooting) with production-quality code displays. The **quality-reviewer** scores finished pages against a 5-dimension rubric (Findability, Accuracy, Clarity, Task Orientation, Readability) and issues pass/fail verdicts as a pre-publish gate.

The **site-architect** agent analyzes the full documentation site for content coverage gaps, cross-section navigation coherence, sitemap completeness, orphan pages, and end-to-end user journey tracing. The **index-generator** produces machine-readable `llms.txt` and `llms-full.txt` files optimized for AI assistant consumption and RAG ingestion.

Skills cover navigation systems (three-pane layouts, sidebar governance, breadcrumbs), taxonomy decisions, search/discovery UX, onboarding journey design, code display patterns, typography standards, accessibility compliance, versioning documentation, lifecycle governance, and benchmarking against industry leaders like Stripe, Vercel, and Kubernetes.

## Usage Examples

**Audit documentation quality**
```
Score our API reference pages against the editorial quality rubric — check findability, accuracy, clarity, and readability
```
The quality-reviewer produces dimension scores and a pass/fail verdict with specific remediation guidance.

**Scaffold a new quickstart page**
```
Scaffold a quickstart page for our Python SDK with Diataxis-aligned structure and working code examples
```
The page-scaffolder generates a structurally correct page with proper anatomy for the quickstart archetype.

**Find coverage gaps**
```
Analyze our docs site architecture — find orphan pages, coverage gaps, and broken navigation paths
```
The site-architect maps the full content graph and identifies structural issues.

**Design the docs landing page**
```
Design the landing page for our developer docs — hero section, self-selection routing, getting-started block, and trust signals
```
Applies the 7-section landing page blueprint with hello-world visuals and persona-based routing.

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `designing-landing-pages` | Docs landing page architecture — hero, self-selection, getting-started, trust signals |
| `designing-navigation-systems` | Three-pane layout, sidebar governance, breadcrumbs, contextual nav switching |
| `governing-taxonomy` | Task-based vs topic-based categorization, hybrid models, split/nest rules |
| `auditing-docs-metrics` | KPIs — TTFS, search-to-click ratio, zero-result rate, bounce rate, usability testing |
| `designing-search-discovery` | Federated search, CMD+K, synonym handling, zero-result recovery |
| `governing-docs-lifecycle` | Docs-as-code workflows, quarterly audits, page lifecycle states, stale detection |
| `analyzing-docs-benchmarks` | Benchmark against Stripe/Vercel/K8s/AWS with pattern extraction and scorecards |
| `scaffolding-page-archetypes` | Diataxis-aligned structural templates — Quickstart, How-To, Reference, Webhooks, Troubleshooting |
| `designing-code-displays` | Syntax highlighting, copy buttons, language tabs, interactive playgrounds, API key injection |
| `generating-llm-indexes` | `llms.txt` / `llms-full.txt` generation for AI assistant consumption and RAG |
| `auditing-content-quality` | 5-dimension editorial rubric with pre-launch validation checklist |
| `designing-typography-layout` | Line length, leading, baseline grid, whitespace density, heading hierarchy |
| `auditing-accessibility-compliance` | WCAG AA — keyboard navigation, screen readers, color contrast, ARIA, focus management |
| `designing-onboarding-journeys` | Multi-page flows, progressive complexity, persona branching, TTFS optimization |
| `governing-version-migrations` | Changelog formats, deprecation notices, migration guides, version selector UX |

### Agents

| Agent | Role |
|-------|------|
| `docs-designer` | Audits and designs documentation structure, clarity, and Diataxis compliance |
| `concept-designer` | Improves concept clarity, terminology consistency, and cognitive load |
| `page-scaffolder` | Generates structurally correct pages from Diataxis archetypes |
| `quality-reviewer` | Scores pages against 5-dimension rubric, issues pre-publish pass/fail |
| `index-generator` | Produces `llms.txt` / `llms-full.txt` machine-readable indexes |
| `site-architect` | Analyzes site-wide coverage, navigation coherence, and user journeys |

</details>

---

**Domain:** developer-docs · **Technologies:** Harness-agnostic · **Aliases:** dev-docs, docs-authoring, developer-documentation
