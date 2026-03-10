---
name: writing-brand-positioning
version: 1.0.0
user-invocable: false
description: Craft positioning statements, ICP profiles, Anti-ICP boundaries, and validated differentiators for SaaS products. Use when defining product positioning, identifying target audience, writing ICP, establishing differentiators, or building the strategic foundation of a Brand Packet. Triggered by: brand positioning, positioning statement, ICP, ideal customer profile, anti-ICP, differentiators, value proposition, product category, target audience, competitive positioning, market positioning.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Brand Positioning

## Purpose

Define the strategic foundation of a Brand Packet: positioning statement, ICP, Anti-ICP, and differentiators. These four elements form Fields 1-4 of the Brand Packet and govern all downstream decisions — copy voice, visual identity, trust signal selection, and page structure.

Strong positioning eliminates ambiguity. Every word on a marketing page should be traceable back to these four fields.

---

## Positioning Statement

### Formula

```
┌─────────────────────────────────────────────────────────────────┐
│                    POSITIONING STATEMENT FORMULA                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  [Product] is a [category] that helps [audience]                 │
│  achieve [outcome] by [mechanism].                               │
│                                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                       │
│  │ CATEGORY │  │ AUDIENCE │  │ OUTCOME  │                       │
│  │ What     │  │ Who      │  │ Why      │                       │
│  │ shelf    │  │ benefits │  │ they     │                       │
│  │ it sits  │  │          │  │ care     │                       │
│  │ on       │  │          │  │          │                       │
│  └──────────┘  └──────────┘  └──────────┘                       │
│                   ┌──────────┐                                   │
│                   │MECHANISM │                                   │
│                   │ How it   │                                   │
│                   │ works    │                                   │
│                   │ (unique) │                                   │
│                   └──────────┘                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Slot Definitions

| Slot | Definition | Quality Test |
|------|-----------|-------------|
| **Product** | Your product name | Unambiguous, no tagline embedded |
| **Category** | Existing market category | Would a buyer search for this category? If not, too invented |
| **Audience** | Specific role + context | Can you picture a single person? If not, too broad |
| **Outcome** | Measurable result | Could you prove this happened? If not, too vague |
| **Mechanism** | How the product delivers | Is this unique to your product? If not, not a mechanism |

### Strong vs. Weak Positioning Examples

| Product | Positioning | Score | Rationale |
|---------|------------|-------|-----------|
| **Stripe** | "Stripe is a payments infrastructure that helps internet businesses accept and manage payments by providing developer-first APIs and prebuilt UIs" | 5/5 | Clear category, specific audience, concrete mechanism |
| **Linear** | "Linear is a project management tool that helps product teams ship software faster by streamlining issue tracking with keyboard-first workflows" | 5/5 | Specific audience (product teams), measurable outcome (faster), unique mechanism (keyboard-first) |
| **Supabase** | "Supabase is an open-source backend-as-a-service that helps developers build and scale apps by providing a Firebase alternative with Postgres" | 4/5 | Strong category-creation via comparison, clear mechanism |
| **Notion** | "Notion is a connected workspace that helps teams collaborate by combining docs, databases, and project management" | 4/5 | Good but "collaborate" is somewhat generic — what specifically? |
| **Generic SaaS** | "Acme is a platform that empowers organizations to drive innovation" | 1/5 | No real category, audience too broad, outcome unmeasurable, no mechanism |
| **Generic SaaS** | "Acme is a tool that helps businesses succeed" | 1/5 | Every word is generic — fails all quality tests |

### Common Positioning Failures

```
┌─────────────────────────────────────────────────────────────────┐
│                    POSITIONING FAILURE MODES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. CATEGORY INVENTION                                           │
│     "We created a new category: Intelligent Workflow Automation" │
│     Problem: Buyers can't search for invented categories         │
│     Fix: Use existing category + differentiating modifier        │
│     Better: "Workflow automation with deterministic execution"   │
│                                                                  │
│  2. AUDIENCE DIFFUSION                                           │
│     "For businesses of all sizes"                                │
│     Problem: When everyone is the audience, no one is            │
│     Fix: Name a specific role at a specific company stage        │
│     Better: "For engineering teams at Series A-C startups"       │
│                                                                  │
│  3. OUTCOME VAGUENESS                                            │
│     "Helps you work better"                                      │
│     Problem: Unmeasurable, unfalsifiable                         │
│     Fix: Use a verb + measurable noun                            │
│     Better: "Reduce deployment time by 80%"                      │
│                                                                  │
│  4. MECHANISM ABSENCE                                            │
│     "Helps teams collaborate more effectively"                   │
│     Problem: Doesn't explain HOW — could be any product          │
│     Fix: Name the specific architectural/product approach        │
│     Better: "...through real-time multiplayer editing"            │
│                                                                  │
│  5. BUZZWORD STACKING                                            │
│     "AI-powered, cloud-native, enterprise-grade platform"        │
│     Problem: Adjective pile-up with zero specificity             │
│     Fix: Pick ONE defining attribute, prove it                   │
│     Better: "Uses GPT-4 to auto-classify support tickets"        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Ideal Customer Profile (ICP)

### Formula

```
[Role] at [company type] who struggles with [pain point].
```

### Stakeholder Extraction Questions

Use these questions to identify the ICP. Ask the product team or founder:

1. **Who signs the check?** What is their title and department?
2. **What company stage?** (Seed, Series A-C, Growth, Enterprise)
3. **What team size?** (solo, 5-20, 20-100, 100+)
4. **What existing tool are they replacing?** (or what manual process?)
5. **What triggers the search?** What event makes them look for a solution?
6. **What is the cost of NOT solving this?** (time, money, risk)
7. **What industry verticals?** (horizontal or specific verticals?)
8. **What technical sophistication?** (technical buyer vs. business buyer)
9. **How do they evaluate?** (free trial, demo, RFP, peer recommendation)
10. **What's the deal-breaker?** What feature/quality causes instant rejection?

### ICP Template

```
PRIMARY ICP:
[Role] at [company type] who struggles with [pain point].

Context:
- Company stage: [stage]
- Team size: [range]
- Replacing: [current tool/process]
- Trigger event: [what makes them search]
- Cost of inaction: [quantified if possible]
- Technical level: [technical / semi-technical / business]
- Evaluation method: [free trial / demo / RFP]
- Deal-breaker: [instant rejection criterion]

SECONDARY ICP (if applicable):
[Role] at [company type] who struggles with [pain point].
```

### ICP Quality Tests

| Test | Question | Pass Criteria |
|------|---------|--------------|
| **Picturability** | Can you visualize this person at their desk? | Yes — specific role, not "stakeholders" |
| **Findability** | Could you build a list of 100 of these people? | Yes — role + company type is searchable |
| **Pain specificity** | Is the pain point falsifiable? | Yes — you could ask them and they'd confirm |
| **Trigger clarity** | Do you know what event starts the search? | Yes — not "when they need better tools" |
| **Exclusion power** | Does this ICP exclude most of the market? | Yes — if it includes everyone, it's useless |

---

## Anti-ICP

### Formula

```
We are explicitly not built for [persona] looking for [irrelevant feature/approach].
```

### Derivation Method

For each ICP attribute, define its opposite:

| ICP Attribute | Anti-ICP Opposite |
|--------------|-------------------|
| Technical buyer | Non-technical buyer who needs drag-and-drop |
| Startup velocity | Enterprise procurement with 6-month evaluation cycles |
| Code-first workflows | No-code/visual builder expectation |
| Self-serve onboarding | White-glove implementation requirement |
| Team collaboration | Individual/personal use |

### Anti-ICP Examples

| Product | Anti-ICP | Rationale |
|---------|----------|-----------|
| **Linear** | "Not built for non-technical teams managing general business projects" | Linear is for software teams, not PMOs |
| **Stripe** | "Not built for merchants who need a turnkey storefront with no code" | Stripe is API-first, not Shopify |
| **Supabase** | "Not built for enterprises requiring managed database administration" | Self-serve developer tool, not managed service |
| **Basecamp** | "Not built for teams that need Gantt charts and resource allocation" | Opinionated simplicity, not enterprise PM |

### Why Anti-ICP Matters

```
Without Anti-ICP:                  With Anti-ICP:
┌──────────────────────┐           ┌──────────────────────┐
│ Marketing speaks to  │           │ Marketing speaks to  │
│ everyone → resonates │           │ the right people →   │
│ with no one          │           │ deeply resonates     │
│                      │           │                      │
│ Support handles      │           │ Self-selecting       │
│ wrong-fit customers  │           │ customers, lower     │
│ who churn            │           │ churn, better NPS    │
│                      │           │                      │
│ Product roadmap      │           │ Product roadmap      │
│ pulled in all        │           │ stays focused on     │
│ directions           │           │ ICP needs            │
└──────────────────────┘           └──────────────────────┘
```

---

## Differentiators

### Formula

```
[Unique capability] validated by [proof].
```

### Proof Type Taxonomy

| Proof Type | Description | Strength | Example |
|-----------|-------------|----------|---------|
| **Metric** | Quantified performance claim | Highest | "99.99% uptime over 36 months" |
| **Architecture** | Technical approach that's hard to copy | High | "Built on SQLite at the edge, not Postgres in a datacenter" |
| **Benchmark** | Third-party or public comparison | High | "3x faster than [competitor] in independent tests" |
| **Customer testimony** | Named person confirming the claim | Medium-High | "'Reduced our deploy time by 80%' — CTO, Acme" |
| **Open source** | Code is inspectable and verifiable | Medium | "MIT licensed, 40K GitHub stars" |
| **Certification** | External audit/compliance | Medium | "SOC 2 Type II certified, audited annually" |
| **Negative proof** | What you explicitly don't do | Medium | "No vendor lock-in — export everything, any time" |

### Differentiator Quality Tests

A strong differentiator passes ALL of these:

| Test | Question | Fail Example |
|------|---------|-------------|
| **Unique** | Would a competitor say the exact same thing? | "We have great support" — everyone says this |
| **Provable** | Can you show evidence right now? | "Best-in-class" — by whose measure? |
| **Relevant** | Does the ICP care about this? | "Built with Rust" — only if audience values performance |
| **Concise** | Can you state it in under 15 words? | Long explanation = not a differentiator, it's a feature list |
| **Durable** | Will this still be true in 12 months? | "Only product with X" — competitors can copy features |

### Differentiator Template

```
DIFFERENTIATOR 1:
Capability: [unique capability in < 15 words]
Proof: [specific evidence]
Proof Type: [from taxonomy above]
ICP Relevance: [why the ICP cares]

DIFFERENTIATOR 2:
Capability: [unique capability in < 15 words]
Proof: [specific evidence]
Proof Type: [from taxonomy above]
ICP Relevance: [why the ICP cares]

DIFFERENTIATOR 3:
Capability: [unique capability in < 15 words]
Proof: [specific evidence]
Proof Type: [from taxonomy above]
ICP Relevance: [why the ICP cares]
```

---

## Instructions

When writing brand positioning:

1. **Start with competitive analysis output** from `analyzing-competitors` — understand the landscape before positioning
2. **Draft the positioning statement** using the formula. Test each slot against quality criteria
3. **Extract the ICP** using the 10 stakeholder questions. Be specific enough to picture one person
4. **Define the Anti-ICP** by inverting each ICP attribute. This sharpens positioning
5. **Identify 3-5 differentiators** using the formula. Each must have proof from the taxonomy
6. **Validate** using the quality checklist below
7. **Output all 4 fields** in the Brand Packet format:
   - Field 1: Positioning Statement
   - Field 2: ICP
   - Field 3: Anti-ICP
   - Field 4: Differentiators (3-5)

---

## Quality Checklist

- [ ] Positioning statement uses all 5 slots (product, category, audience, outcome, mechanism)
- [ ] Category is searchable (not invented)
- [ ] Audience is specific enough to picture one person
- [ ] Outcome is measurable or falsifiable
- [ ] Mechanism is unique to this product
- [ ] ICP includes role, company type, and pain point
- [ ] Anti-ICP excludes at least 2 specific personas
- [ ] Each differentiator has explicit proof (not just claims)
- [ ] No banned vocabulary in any field (seamless, robust, leverage, empower, etc.)

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **"Platform for everyone"** | No audience specificity | Name one role, one company stage |
| **Feature-as-positioning** | "We have real-time collaboration" | Translate feature into outcome for ICP |
| **Competitor-reactive** | "Better than [competitor]" | Position on your own axis, not theirs |
| **Aspirational positioning** | Claiming what you want to be, not what you are | Position from current truth, not future vision |
| **Category overload** | "AI-powered DevSecOps platform" | Pick one category, differentiate within it |

---

## Edge Cases

### Pre-Launch Products
- Positioning can be aspirational but must be grounded in what's being built
- ICP should be the design partner profile
- Differentiators may rely on architecture proof and founder credibility

### Horizontal Products
- Resist "for everyone" — pick the wedge ICP that will expand later
- Example: Notion started with "for engineers" before expanding

### Crowded Categories
- Lean harder on mechanism and Anti-ICP
- The more competitors, the more specific positioning must be
- Consider positioning on a different axis entirely (speed, simplicity, transparency)

### Category Creation
- Only attempt if the existing category actively misleads buyers
- Must still map to a recognizable adjacent category
- Example: "The open source Firebase alternative" — uses existing category as anchor

---

## Related Skills and Agents

- **`analyzing-competitors` skill**: Produces competitive intelligence that informs positioning decisions
- **`writing-brand-voice` skill**: Translates positioning into tonal guidelines (Fields 5-7)
- **`writing-trust-signals` skill**: Identifies proof points that validate differentiators
- **`validating-brand-packets` skill**: Validates completeness and quality of positioning fields
- **`copy_writer` agent**: Writes page copy grounded in positioning

**When to use which**: Use this skill to define the strategic foundation (Fields 1-4). Then use `writing-brand-voice` for tone, `writing-visual-identity` for design tokens, and `writing-trust-signals` for proof points.
