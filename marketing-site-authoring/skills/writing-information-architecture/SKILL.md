---
name: writing-information-architecture
version: 1.0.0
user-invocable: false
description: Produce a section-by-section Information Architecture document from a validated Brand Packet, defining narrative flow, layout patterns, trust mechanics, and conversion strategy for a landing page. Does NOT write copy. Use when designing page structure, defining section sequencing, selecting layout patterns, or mapping conviction narrative arc. Triggered by: information architecture, IA document, page structure, section sequencing, narrative arc, layout patterns, conviction model, section archetype, page flow, landing page IA, marketing IA, content strategy.
allowed-tools: Read, Glob, Grep, Write
---

# Information Architecture Strategy

## Purpose

Take a validated Brand Packet (`brand/brand-packet.md`) and produce a section-by-section Information Architecture document that defines what each section of a landing page does, why it's positioned where it is, and how it moves the visitor through a psychological conviction arc toward conversion.

This skill produces **strategy only** — section names, archetypes, layout patterns, content slot definitions, trust mechanic assignments, and transition logic. It does NOT write copy. Copy is produced downstream by `writing-wireframe-copy`.

The IA document is the structural blueprint. Every downstream decision — copy, components, implementation — traces back to it.

---

## Input Contract

The IA document derives all structural decisions from the 11-field Brand Packet. Each field informs specific IA decisions:

| Brand Packet Field | IA Decision It Informs | How It's Used |
|--------------------|----------------------|---------------|
| **Field 1: Positioning Statement** | Hero section strategic goal, headline content slot definition | The 5-slot formula (product, category, audience, outcome, mechanism) maps directly to hero content slots |
| **Field 2: ICP** | Section sequencing priority, pain-point section placement | ICP pain point determines which sections appear early; technical level determines layout complexity |
| **Field 3: Anti-ICP** | Objection handler placement, self-selection mechanics | Anti-ICP personas inform which objections to address and where to place disqualification signals |
| **Field 4: Differentiators** | Value Grid content, Mechanism section structure | Each differentiator becomes a content slot in Value Grid or Mechanism sections; proof type determines layout |
| **Field 5: Vibe** | Layout pattern selection, visual density decisions | "Precise" vibes favor minimal layouts; "playful" vibes favor richer visual patterns |
| **Field 6: Anti-Vibe** | Layout constraint boundaries | Anti-vibes prevent layout choices that would violate tone (e.g., "not flippant" prevents overly casual patterns) |
| **Field 8: Trust Signals** | Trust mechanic assignment per section, Social Proof Bar content | Trust signal catalog maps directly to section-level trust mechanic slots |
| **Field 11: Spatial Rules** | Section spacing, layout grid constraints | Macro-whitespace values govern section separation; grid system constrains layout pattern options |
| **Field 7: Banned Vocabulary** | *Passes through to Stage 2* | Not used in IA — enforced during copy writing |
| **Field 9: Color Tokens** | *Passes through to implementation* | Not used in IA — applied during component build |
| **Field 10: Typography System** | *Passes through to implementation* | Not used in IA — applied during component build |

**Required input**: `brand/brand-packet.md` must exist and pass validation (all 11 fields present).

---

## Conviction Narrative Model

Every landing page moves visitors through a psychological arc from stranger to buyer. The IA document maps each section to a stage in this arc.

```
+-------------+     +---------------+     +----------+     +----------+     +------------+
| RECOGNITION |---->| UNDERSTANDING |---->|  BELIEF  |---->|  DESIRE  |---->| CONFIDENCE |
+-------------+     +---------------+     +----------+     +----------+     +------------+
  "This is        "I see how it       "The evidence    "I want this     "I'm ready
   for me."        works."              checks out."     for my team."    to act."
```

### 5 Conviction States

| State | Visitor Psychology | Strategic Goal | Typical Sections |
|-------|-------------------|---------------|-----------------|
| **Recognition** | "Is this relevant to me? Do they understand my problem?" | Match ICP pain point, establish category, signal tribe membership | Hero, Social Proof Bar |
| **Understanding** | "How does this actually work? What's the mechanism?" | Explain the unique mechanism from Field 1, show the product in action | Mechanism, Value Grid |
| **Belief** | "Can I trust these claims? Is there evidence?" | Provide third-party validation, metrics, named testimonials | Proof Block, Social Proof Bar |
| **Desire** | "What would this look like for my team/workflow?" | Paint the after-state, connect differentiators to ICP outcomes | Value Grid, Objection Handler |
| **Confidence** | "What's the risk? How do I start?" | Remove friction, address objections, clarify next step | CTA Block, Objection Handler, Community Block |

### Sequencing Principle

Sections MUST appear in conviction-arc order. A visitor cannot believe claims they don't understand, and cannot desire outcomes they don't believe are real. Skipping states creates a trust gap.

---

## Section Archetype Catalog

Every section on the page uses one of these 8 archetypes. Each archetype has a strategic purpose, candidate layout patterns, content slots, trust mechanics, and cognitive load rating.

### Archetype 1: Hero

| Property | Value |
|----------|-------|
| **Strategic Goal** | Establish category, match ICP pain point, trigger Recognition state |
| **Conviction State** | Recognition |
| **Layout Pattern Options** | `centered-statement`, `split-hero`, `hero-with-demo` |
| **Content Slots** | headline, subheadline, cta-primary, cta-secondary, hero-visual (optional) |
| **Trust Mechanic** | Social proof bar immediately below (logo bar or primary metric) |
| **Cognitive Load** | Low — 5-second scan must convey category + audience + outcome |
| **Mandatory** | Yes — every landing page has exactly one Hero |

**Pattern Selection Decision Tree**:
- Is there a compelling product screenshot or demo? → `hero-with-demo` or `split-hero`
- Is the product abstract or infrastructure? → `centered-statement`
- Does the vibe favor minimalism? → `centered-statement`
- Does the vibe favor energy/action? → `split-hero`

### Archetype 2: Social Proof Bar

| Property | Value |
|----------|-------|
| **Strategic Goal** | Provide instant borrowed credibility, validate category claim |
| **Conviction State** | Recognition → Belief bridge |
| **Layout Pattern Options** | `logo-ticker`, `logo-grid`, `metric-bar`, `metric-plus-logos` |
| **Content Slots** | logo-set (6-8 logos), metric-label (optional), metric-value (optional) |
| **Trust Mechanic** | IS the trust mechanic — logos and/or metrics are the content |
| **Cognitive Load** | Minimal — pure recognition, no reading required |
| **Mandatory** | Yes — must appear within 1 viewport of Hero |

**Pattern Selection Decision Tree**:
- 6+ recognizable logos available? → `logo-ticker` or `logo-grid`
- Strong primary metric (e.g., "500K+ developers")? → `metric-bar` or `metric-plus-logos`
- Fewer than 6 logos? → `metric-bar` with available logos as secondary

### Archetype 3: Mechanism

| Property | Value |
|----------|-------|
| **Strategic Goal** | Explain the unique mechanism from the positioning statement (Field 1) |
| **Conviction State** | Understanding |
| **Layout Pattern Options** | `split-column`, `code-then-output`, `annotated-screenshot`, `step-sequence` |
| **Content Slots** | section-headline, section-subheadline, mechanism-visual, body-paragraph, annotation-labels (optional) |
| **Trust Mechanic** | Architecture proof (Field 4 proof type), demo/screenshot |
| **Cognitive Load** | Medium — requires attention but should feel inevitable, not complex |
| **Mandatory** | Yes — mechanism is the "how" that differentiates |

**Pattern Selection Decision Tree**:
- Is the mechanism code-centric? → `code-then-output`
- Is the mechanism visual/workflow? → `annotated-screenshot` or `step-sequence`
- Is the mechanism conceptual/architectural? → `split-column` with diagram

### Archetype 4: Value Grid

| Property | Value |
|----------|-------|
| **Strategic Goal** | Present differentiators as concrete capabilities the ICP values |
| **Conviction State** | Understanding → Desire bridge |
| **Layout Pattern Options** | `bento-grid`, `three-column-cards`, `icon-grid`, `feature-list` |
| **Content Slots** | section-headline, feature-card-title (per card), feature-card-description (per card), feature-card-icon (optional per card) |
| **Trust Mechanic** | Each card links to a differentiator (Field 4); proof embedded in card description |
| **Cognitive Load** | Medium-High — limit to 3-6 cards to prevent overload |
| **Mandatory** | Yes — differentiators must be surfaced |

**Pattern Selection Decision Tree**:
- 3 differentiators? → `three-column-cards`
- 4-6 differentiators? → `bento-grid` (allows visual hierarchy among cards)
- Differentiators are highly technical? → `feature-list` with code examples
- Differentiators are outcome-focused? → `icon-grid`

### Archetype 5: Proof Block

| Property | Value |
|----------|-------|
| **Strategic Goal** | Provide irrefutable evidence that claims are real |
| **Conviction State** | Belief |
| **Layout Pattern Options** | `testimonial-card`, `case-study-preview`, `metric-showcase`, `before-after` |
| **Content Slots** | quote-text, quote-attribution (name, role, company), metric-value, metric-label, case-study-link (optional) |
| **Trust Mechanic** | Named testimonials (Field 8 quotes), verified metrics (Field 8 metrics) |
| **Cognitive Load** | Low — proof should feel effortless to absorb |
| **Mandatory** | Yes — at least one Proof Block required |

**Pattern Selection Decision Tree**:
- Named quotes with metrics available? → `testimonial-card` with metric callout
- Strong before/after story? → `before-after`
- Multiple proof points? → `metric-showcase`
- Full case study available? → `case-study-preview`

### Archetype 6: CTA Block

| Property | Value |
|----------|-------|
| **Strategic Goal** | Convert conviction into action |
| **Conviction State** | Confidence |
| **Layout Pattern Options** | `centered-statement`, `split-column`, `card-cta` |
| **Content Slots** | cta-headline, cta-subheadline, cta-primary, cta-secondary (optional), friction-reducer (optional) |
| **Trust Mechanic** | Friction reducers — "No credit card", "Free tier", "Cancel anytime" |
| **Cognitive Load** | Minimal — zero ambiguity about next step |
| **Mandatory** | Yes — every page needs a final CTA |

**Pattern Selection Decision Tree**:
- Primary CTA is free trial / sign-up? → `centered-statement` with friction reducers
- Primary CTA is demo / sales? → `split-column` with form or calendar embed
- Multiple CTAs (free + enterprise)? → `card-cta` with two paths

### Archetype 7: Objection Handler

| Property | Value |
|----------|-------|
| **Strategic Goal** | Address the top 2-3 objections the ICP has before converting |
| **Conviction State** | Confidence |
| **Layout Pattern Options** | `faq-accordion`, `comparison-table`, `split-column`, `objection-cards` |
| **Content Slots** | section-headline, objection-question (per item), objection-answer (per item), supporting-proof (optional per item) |
| **Trust Mechanic** | Anti-ICP contrast (Field 3), compliance badges (Field 8 badges) |
| **Cognitive Load** | Medium — visitors in this state are actively evaluating |
| **Mandatory** | No — include when ICP has predictable objections |

**Pattern Selection Decision Tree**:
- Objections are "how is this different from X?" → `comparison-table`
- Objections are "what about Y?" → `faq-accordion`
- Objections are "is this safe/compliant?" → `split-column` with badges
- Objections are varied/miscellaneous? → `objection-cards`

### Archetype 8: Community Block

| Property | Value |
|----------|-------|
| **Strategic Goal** | Signal that a community of peers already trusts and uses the product |
| **Conviction State** | Desire → Confidence bridge |
| **Layout Pattern Options** | `community-stats`, `contributor-grid`, `social-wall`, `centered-statement` |
| **Content Slots** | section-headline, community-metric (per stat), community-link (per channel), contributor-avatars (optional) |
| **Trust Mechanic** | Community signals from Field 8 (GitHub stars, Discord members, contributors) |
| **Cognitive Load** | Low — social validation is absorbed, not analyzed |
| **Mandatory** | No — include when community signals are strong (open source, developer tool) |

**Pattern Selection Decision Tree**:
- Open source product with strong GitHub metrics? → `community-stats` or `contributor-grid`
- Active social/community channels? → `social-wall`
- Community exists but metrics are modest? → `centered-statement` with link

---

## Section Sequencing Rules

### Mandatory Ordering Constraints

These rules are inviolable regardless of flow template:

1. **Hero is always Section 1** — no exceptions
2. **Social Proof Bar is always Section 2** — must appear within 1 viewport of Hero
3. **CTA Block is always the final section** — the page ends with a conversion opportunity
4. **Mechanism must precede Value Grid** — visitors must understand the "how" before evaluating capabilities
5. **At least one Proof Block must precede the final CTA** — never ask for conversion without proof
6. **Objection Handler, if present, precedes the final CTA** — handle objections before asking for action

### Flow Templates

Select the template that best matches the product type. Templates are starting points — adjust based on Brand Packet specifics.

#### Template A: Developer Tool (6-7 sections)

```
1. Hero                     — Recognition
2. Social Proof Bar         — Recognition → Belief
3. Mechanism                — Understanding
4. Value Grid               — Understanding → Desire
5. Proof Block              — Belief
6. Community Block          — Desire → Confidence
7. CTA Block                — Confidence
```

Best for: OSS tools, CLI tools, APIs, developer infrastructure. The Mechanism section is heavy (code examples, architecture diagrams). Community Block is prominent.

#### Template B: Enterprise SaaS (7-8 sections)

```
1. Hero                     — Recognition
2. Social Proof Bar         — Recognition → Belief
3. Value Grid               — Understanding → Desire
4. Mechanism                — Understanding
5. Proof Block              — Belief
6. Proof Block (2nd)        — Belief (deeper)
7. Objection Handler        — Confidence
8. CTA Block                — Confidence
```

Best for: B2B SaaS, enterprise products, compliance-heavy tools. Value Grid comes before Mechanism because enterprise buyers evaluate outcomes before mechanisms. Two Proof Blocks provide extra trust density.

#### Template C: Open Source / Community-Led (6-8 sections)

```
1. Hero                     — Recognition
2. Social Proof Bar         — Recognition → Belief
3. Mechanism                — Understanding
4. Community Block          — Desire → Confidence
5. Value Grid               — Understanding → Desire
6. Proof Block              — Belief
7. Objection Handler        — Confidence (optional)
8. CTA Block                — Confidence
```

Best for: OSS projects, community-driven tools, freemium. Community Block is elevated because peer validation is the primary trust mechanism. Objection Handler addresses "is this production-ready?" concerns.

---

## Layout Pattern Reference

Layout patterns are stack-agnostic descriptions of visual arrangement. They describe spatial relationships, not implementation details. Drawn from analysis of a 40-site corpus of high-performing SaaS marketing pages.

| Pattern Name | Description | Best For | Frequency in Corpus |
|-------------|-------------|----------|-------------------|
| `centered-statement` | Single column, centered text, maximum whitespace | Hero headlines, CTA blocks, minimal vibes | 38/40 sites |
| `split-column` | Two columns, content + visual, balanced weight | Mechanism sections, explanations with diagrams | 35/40 sites |
| `split-hero` | Asymmetric two-column hero, text left + visual right | Hero with product screenshot or demo | 28/40 sites |
| `hero-with-demo` | Hero text above embedded product demo or video | Hero for product-led growth | 12/40 sites |
| `bento-grid` | Asymmetric grid with varied card sizes, visual hierarchy | Value grids with 4-6 features | 22/40 sites |
| `three-column-cards` | Three equal-width cards in a row | Value grids with exactly 3 features | 30/40 sites |
| `icon-grid` | Grid of icon + title + description units | Feature overviews, capability lists | 25/40 sites |
| `feature-list` | Vertical list with alternating left/right visuals | Detailed feature walkthroughs | 18/40 sites |
| `code-then-output` | Code block on left/top, rendered output on right/bottom | Developer tools showing usage + result | 15/40 sites |
| `annotated-screenshot` | Product screenshot with numbered callout labels | Product tours, UI-heavy mechanisms | 14/40 sites |
| `step-sequence` | Numbered horizontal or vertical steps | Onboarding flows, setup processes | 20/40 sites |
| `testimonial-card` | Quote + attribution in a styled card | Social proof, customer stories | 32/40 sites |
| `case-study-preview` | Summary card linking to full case study | Enterprise trust building | 16/40 sites |
| `metric-showcase` | Large numbers with labels, grid or horizontal | Scale/performance proof | 24/40 sites |
| `before-after` | Two-state comparison (old way vs. new way) | Transformation stories | 10/40 sites |
| `logo-ticker` | Horizontally scrolling logo bar | Social proof, customer logos | 28/40 sites |
| `logo-grid` | Static grid of customer logos | Social proof, fewer logos | 20/40 sites |
| `metric-bar` | Horizontal bar with 3-4 key metrics | Compact social proof | 18/40 sites |
| `metric-plus-logos` | Metric bar combined with logo row | Combined proof density | 12/40 sites |
| `faq-accordion` | Expandable question/answer pairs | Objection handling, FAQs | 26/40 sites |
| `comparison-table` | Side-by-side feature comparison grid | Competitor differentiation | 14/40 sites |
| `community-stats` | Grid of community metrics (stars, members, etc.) | Open source social proof | 10/40 sites |
| `contributor-grid` | Avatar grid of top contributors | Open source community | 8/40 sites |
| `social-wall` | Grid of social media mentions or testimonials | Broad social proof | 6/40 sites |
| `card-cta` | Multiple CTA paths as distinct cards | Multiple conversion paths (free/enterprise) | 12/40 sites |
| `objection-cards` | Individual cards addressing specific concerns | Mixed objection handling | 8/40 sites |

---

## Trust Mechanic Integration

Each section is assigned a trust mechanic from the Brand Packet's trust signal catalog (Field 8). This table maps section positions to appropriate trust signal types:

| Section Position | Recommended Trust Signal Types | Assignment Rule |
|-----------------|-------------------------------|----------------|
| **Hero** | Primary metric, logo bar | Strongest single metric + most recognizable logos |
| **Social Proof Bar** | Logo set, metric bar | 6-8 logos the ICP would recognize; optional aggregate metric |
| **Mechanism** | Architecture proof, demo/screenshot | Proof type from Field 4 differentiators that match the mechanism |
| **Value Grid** | Per-card micro-proof | Each feature card references its differentiator's proof (Field 4) |
| **Proof Block** | Named quotes, verified metrics, case study links | Highest-quality trust signals from Field 8 catalog |
| **Objection Handler** | Compliance badges, comparison data, negative proof | Badges and certifications that address specific objections |
| **Community Block** | GitHub stars, contributor count, Discord members | Community signals from Field 8 catalog |
| **CTA Block** | Friction reducers, compliance badges | "No credit card", "Free tier", SOC 2 badge near conversion |

### Trust Distribution Validation

After assigning trust mechanics, verify:
- Every section has at least one trust mechanic (even if subtle)
- No trust signal type is used in more than 3 sections (prevent monotony)
- The strongest signals are concentrated near Hero and CTA (conversion proximity)
- Proof Block uses the highest-provability signals from the catalog

---

## IA Entry Schema

Each section in the IA document uses this specification format:

```
### Section [N]: [Section Name]

**Archetype**: [from catalog]
**Layout Pattern**: [from reference]
**Conviction State**: [entering] → [leaving]
**Strategic Goal**: [one sentence — what this section accomplishes]

**Content Slots**:
| Slot | Type | Constraints |
|------|------|-------------|
| [slot-name] | [slot-type] | [max length, derivation source, etc.] |

**Trust Mechanic**: [assigned trust signal type + specific signal from Field 8]

**Differentiator Link**: [which Field 4 differentiator this section supports, if any]

**Transition Logic**: [how this section hands off to the next — what psychological state
the visitor should be in when they scroll past this section]
```

---

## Content Slot Types

Standard vocabulary for content slots used across all archetypes. Each slot type has constraints that govern the copy written in Stage 2.

| Slot Type | Description | Typical Length | Constraints |
|-----------|-------------|---------------|-------------|
| `headline` | Primary section heading | 4-8 words | Concrete benefit or technical mechanism; no banned words |
| `subheadline` | Supporting text below headline | 1-2 sentences | Amplifies headline with proof, numbers, or constraints |
| `body-paragraph` | Explanatory text block | 2-3 sentences max | No paragraph exceeds 3 sentences; active voice |
| `cta-primary` | Primary call-to-action button text | 2-4 words | Action verb + outcome; not "Get Started" or "Learn More" |
| `cta-secondary` | Secondary/alternative action text | 2-4 words | Lower commitment than primary; "See docs", "Watch demo" |
| `feature-card-title` | Card heading in Value Grid | 3-6 words | Names the capability, not the category |
| `feature-card-description` | Card body in Value Grid | 1-2 sentences | Specific outcome + proof; no generic adjectives |
| `metric-value` | Numeric proof point | 1-5 characters | Must be from Field 8 metrics; hyper-specific |
| `metric-label` | Label explaining the metric | 3-8 words | Context that makes the number meaningful |
| `quote-text` | Testimonial content | 1-3 sentences | From Field 8 quotes; metric-backed preferred |
| `quote-attribution` | Name, role, company | Structured | Must include all 3; optional company detail |
| `logo-set` | Collection of customer logos | 6-8 logos | From Field 8 logos; ICP-recognizable |
| `objection-question` | FAQ or objection heading | 5-15 words | Written as the ICP would actually phrase it |
| `objection-answer` | Response to objection | 2-4 sentences | Proof-backed, not dismissive |
| `community-metric` | Community stat | Number + label | From Field 8 community signals |
| `community-link` | Channel link | URL + label | GitHub, Discord, forum, etc. |
| `friction-reducer` | Risk-removal statement | 3-8 words | "No credit card", "Free tier", "Cancel anytime" |
| `section-headline` | Heading for non-hero sections | 4-10 words | Names the section's strategic goal, not generic |
| `section-subheadline` | Supporting text for section heading | 1-2 sentences | Provides context for the section content |
| `annotation-label` | Callout label on screenshot/diagram | 2-5 words | Identifies specific UI element or feature |

---

## Output Format

Write the completed IA document to `brand/information-architecture.md`. Use this template:

```markdown
# Information Architecture: [Product Name]

**Date**: [date]
**Brand Packet**: brand/brand-packet.md
**Flow Template**: [A: Developer Tool | B: Enterprise SaaS | C: Open Source / Community-Led]
**Section Count**: [N]

---

## Narrative Summary

[2-3 sentences describing the page's narrative arc: what psychological journey
the visitor takes from landing to conversion. Reference the ICP and primary
differentiator.]

---

## Conviction Arc Map

| State | Section(s) | Purpose |
|-------|-----------|---------|
| Recognition | [section numbers] | [what visitor recognizes] |
| Understanding | [section numbers] | [what visitor understands] |
| Belief | [section numbers] | [what visitor believes] |
| Desire | [section numbers] | [what visitor desires] |
| Confidence | [section numbers] | [what gives visitor confidence] |

---

## Section Definitions

### Section 1: [Name]

**Archetype**: [archetype]
**Layout Pattern**: [pattern]
**Conviction State**: [entering] → [leaving]
**Strategic Goal**: [one sentence]

**Content Slots**:
| Slot | Type | Constraints |
|------|------|-------------|
| ... | ... | ... |

**Trust Mechanic**: [type + specific signal]

**Differentiator Link**: [Field 4 reference or "N/A"]

**Transition Logic**: [handoff to next section]

[Repeat for each section]

---

## Trust Distribution Summary

| Section | Trust Signal Type | Specific Signal |
|---------|------------------|----------------|
| [N] | [type] | [from Field 8] |

---

## Validation Checklist

- [ ] All mandatory archetypes present (Hero, Social Proof Bar, Mechanism, Value Grid, Proof Block, CTA Block)
- [ ] Sections follow conviction arc order (Recognition → Understanding → Belief → Desire → Confidence)
- [ ] Hero is Section 1; CTA Block is final section
- [ ] Social Proof Bar is within 1 viewport of Hero
- [ ] Mechanism precedes Value Grid
- [ ] At least one Proof Block precedes final CTA
- [ ] Every section has an assigned trust mechanic
- [ ] Every differentiator (Field 4) is linked to at least one section
- [ ] No layout pattern repeated more than twice consecutively
- [ ] Section count is 6-8 (not under-structured, not bloated)
- [ ] Content slot types use standard vocabulary
- [ ] Transition logic is defined for every section boundary
```

---

## Instructions

When producing an Information Architecture document:

1. **Read the Brand Packet** at `brand/brand-packet.md`. Verify all 11 fields are present. If the file doesn't exist or is incomplete, stop and report the gap.
2. **Select the flow template** (A: Developer Tool, B: Enterprise SaaS, C: Open Source / Community-Led) based on product type, ICP (Field 2), and positioning (Field 1).
3. **Map the conviction arc** — assign each Brand Packet insight to the conviction state it best supports. Determine which states need extra reinforcement based on ICP skepticism level and product complexity.
4. **Define sections** — choose archetypes for each section. Start with the flow template, then customize: add sections where conviction gaps exist, remove optional sections that lack supporting data.
5. **Select layout patterns** — for each section, use the pattern selection decision tree. Cross-reference with vibe (Field 5) and spatial rules (Field 11) to ensure pattern choices match brand personality.
6. **Define content slots** — for each section, list all content slots with types and constraints. These become the exact deliverables for the copy stage.
7. **Assign trust mechanics** — map Field 8 trust signals to sections using the integration table. Verify distribution covers all sections and strongest signals are near conversion.
8. **Link differentiators** — verify every differentiator from Field 4 is explicitly linked to at least one section. If a differentiator has no home, add a section or expand an existing one.
9. **Write transition logic** — for each section boundary, describe the psychological handoff. What state is the visitor in leaving section N? Does section N+1 meet them there?
10. **Run the validation checklist** — verify all constraints are met. If any fail, adjust sections before writing output.

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Feature laundry list** | Value Grid has 10+ cards, overwhelming the visitor | Limit to 3-6 cards; group secondary features into a link |
| **Trust vacuum** | 3+ consecutive sections with no trust mechanic | Assign at least one trust signal per section |
| **Copy prescription** | IA document contains actual copy or headline drafts | Remove all copy; IA defines slot types and constraints, not words |
| **Layout monotony** | Same layout pattern used 3+ times consecutively | Alternate between patterns; never repeat more than twice |
| **Kitchen sink** | 10+ sections trying to cover everything | Limit to 6-8 sections; ruthlessly cut sections that don't advance the conviction arc |
| **Arc skipping** | Jumping from Recognition to Desire without Understanding | Insert Mechanism or Value Grid section to bridge the gap |
| **Orphan differentiator** | A differentiator from Field 4 has no section link | Add a section or expand a content slot to accommodate it |
| **Proof-last** | All trust signals placed after the fold or near footer | Move strongest signals to Hero-adjacent Social Proof Bar |
| **Generic section names** | "Features", "About", "Why Us" | Name sections by strategic goal: "How it works", "Trusted by 12,000 teams" |
| **Missing transition logic** | Sections feel disconnected, no narrative thread | Define explicit psychological handoff at every section boundary |

---

## Edge Cases

### Pre-Launch Products
- Community Block may be premature — omit or use waitlist count as social proof
- Mechanism section is critical — it's the primary proof when metrics are unavailable
- Use founder credibility and investor logos in place of customer trust signals
- Proof Block may use beta user feedback or design partner quotes

### Limited Trust Signals
- If fewer than 6 logos are available, use `metric-bar` pattern instead of `logo-ticker`
- If no named quotes exist, use `metric-showcase` for Proof Block instead of `testimonial-card`
- Flag gaps in the validation checklist and note acquisition timeline
- Prioritize Field 4 differentiator proofs as the primary trust source

### Highly Technical Products
- Mechanism section may need two passes: conceptual overview + technical deep-dive
- Consider `code-then-output` or `annotated-screenshot` patterns more heavily
- Value Grid cards can include technical specifications rather than outcomes
- Objection Handler should address integration/migration concerns

### Multiple ICPs
- Primary ICP governs section sequencing and hero content
- Secondary ICP can be served by adding an Objection Handler or secondary Value Grid
- Do NOT create separate page tracks within one IA — design for the primary ICP
- Note the secondary ICP in transition logic where relevant

### Redesigns of Existing Pages
- Audit the existing page structure before designing the new IA
- Map existing sections to archetypes to identify what's working
- Preserve sections that already perform well (check analytics if available)
- Note "migration" considerations in transition logic

---

## Related Skills and Agents

- **`brand_packet_producer` agent**: Produces the Brand Packet that is the sole input to this skill
- **`writing-wireframe-copy` skill**: Consumes the IA document to write exact copy for every content slot
- **`reviewing-copy` skill**: Reviews the copy produced from this IA for voice compliance and generic language
- **`writing-brand-positioning` skill**: Positioning (Fields 1-4) governs section archetypes and sequencing
- **`writing-trust-signals` skill**: Trust catalog (Field 8) maps to section-level trust mechanics
- **`writing-visual-identity` skill**: Spatial rules (Field 11) constrain layout pattern selection

**Pipeline position**: This skill is **Stage 1** in the marketing production pipeline. It runs after the Brand Packet is validated (Stage 0) and before wireframe copy is written (Stage 2). The IA document is the structural contract that both copy (Stage 2) and editorial review (Stage 3) operate within.
