---
name: writing-brand-voice
version: 1.0.0
user-invocable: false
description: Define tonal boundaries for marketing copy including vibe adjectives, anti-vibe constraints, and banned vocabulary. Use when establishing brand voice, defining tone guidelines, creating copy style guides, or setting editorial standards. Triggered by: brand voice, tone of voice, copy tone, vibe, anti-vibe, banned words, banned vocabulary, editorial style, writing guidelines, copy style guide, voice consistency, tonal boundaries.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Brand Voice Definition

## Purpose

Define the tonal boundaries that govern all marketing copy: 3 vibe adjectives, their anti-vibe opposites, and a banned vocabulary list. These form Fields 5-7 of the Brand Packet.

Voice is not what you say — it's how you say it. A consistent voice builds trust. An inconsistent voice signals an immature brand. The goal is to create guardrails tight enough to produce consistent copy across any writer (human or AI) while leaving room for creative expression.

---

## The 3-Adjective Constraint

### Methodology

Every brand voice is defined by exactly 3 adjectives. Not 2 (too vague), not 5 (too diluted). Three forces prioritization.

```
┌─────────────────────────────────────────────────────────────────┐
│                    3-ADJECTIVE SELECTION CRITERIA                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Each adjective MUST be:                                         │
│                                                                  │
│  1. SPECIFIC — Not "good" or "professional"                      │
│     ✗ "Professional" (means nothing, every brand claims this)    │
│     ✓ "Precise" (observable in word choice and sentence length)  │
│                                                                  │
│  2. OBSERVABLE — You can point to it in actual copy               │
│     ✗ "Innovative" (how does copy sound innovative?)             │
│     ✓ "Terse" (short sentences, no filler, economical)           │
│                                                                  │
│  3. DIFFERENTIATING — Not every competitor would claim it         │
│     ✗ "Trustworthy" (every SaaS claims trustworthiness)          │
│     ✓ "Irreverent" (only brands willing to break conventions)    │
│                                                                  │
│  The 3 adjectives should create TENSION:                         │
│  "Technical + Warm + Concise" has productive tension              │
│  "Good + Great + Excellent" has zero tension (and zero meaning)  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Vibe Spectrum Archetypes

Use these archetypes as starting points, then customize:

| Archetype | 3 Adjectives | Typical Product | Example Brand |
|-----------|-------------|-----------------|---------------|
| **Technical Founder** | Precise, Direct, Understated | Dev tools, infrastructure | Stripe, Linear |
| **Enterprise Authority** | Confident, Measured, Authoritative | Enterprise SaaS, compliance | Salesforce, Palo Alto |
| **Developer Casual** | Playful, Technical, Honest | OSS tools, developer communities | PostHog, Supabase |
| **Design Minimalist** | Refined, Quiet, Intentional | Design tools, premium products | Arc, Raycast |
| **Warm Educator** | Clear, Encouraging, Patient | EdTech, documentation-heavy | Notion, Stripe Docs |
| **Challenger Brand** | Bold, Opinionated, Concise | Disruptors, category challengers | Basecamp, Hey |
| **Data-Driven** | Analytical, Specific, Neutral | Analytics, BI, data platforms | Amplitude, Segment |
| **Community Builder** | Inclusive, Enthusiastic, Transparent | OSS, community platforms | Hugging Face, Discord |
| **Security-First** | Measured, Factual, Calm | Security, compliance, privacy | 1Password, Tailscale |
| **Startup Energy** | Ambitious, Urgent, Conversational | Early-stage, growth-phase | Fast-growing startups |

### Archetype Selection Process

1. Review positioning statement and ICP from `writing-brand-positioning`
2. Identify which 2-3 archetypes feel closest to the product's personality
3. Select 1 adjective from each archetype (or modify to fit)
4. Test the combination: does it create productive tension?
5. Validate: could a writer use these 3 words to make a style decision?

---

## Anti-Vibe Derivation

### Method

For each vibe adjective, define its failure mode — not the literal opposite, but the version of that quality taken too far or executed poorly.

```
┌─────────────────────────────────────────────────────────────────┐
│                    ANTI-VIBE DERIVATION                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Vibe Adjective ──→ Anti-Vibe (failure mode)                     │
│                                                                  │
│  "Precise"      ──→ "Robotic"     (precision without humanity)   │
│  "Direct"       ──→ "Blunt"       (directness without empathy)   │
│  "Playful"      ──→ "Flippant"    (playfulness without respect)  │
│  "Confident"    ──→ "Arrogant"    (confidence without humility)  │
│  "Technical"    ──→ "Jargon-heavy" (technical without clarity)   │
│  "Warm"         ──→ "Saccharine"  (warmth without substance)     │
│  "Bold"         ──→ "Aggressive"  (boldness without restraint)   │
│  "Understated"  ──→ "Invisible"   (understatement to obscurity)  │
│  "Concise"      ──→ "Cryptic"     (brevity without clarity)      │
│  "Honest"       ──→ "Self-deprecating" (honesty without pride)   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Anti-Vibe Template

```
VIBE 1: [adjective]
Anti-Vibe: [failure mode]
Boundary: "We are [vibe], but never [anti-vibe]."
Example: "We explain technical concepts clearly (precise) but never sound
         like a manual written by a committee (robotic)."

VIBE 2: [adjective]
Anti-Vibe: [failure mode]
Boundary: "We are [vibe], but never [anti-vibe]."

VIBE 3: [adjective]
Anti-Vibe: [failure mode]
Boundary: "We are [vibe], but never [anti-vibe]."
```

---

## Banned Vocabulary

### Default Banned Words

These 8 words are banned by default in all Brand Packets. They are generic, overused, and carry zero information:

| Banned Word | Why It's Banned | Replacement Strategy |
|------------|----------------|---------------------|
| **Supercharge** | Hyperbolic, meaningless — what does 10x electricity mean for software? | Name the specific improvement: "reduce build times by 60%" |
| **Unleash** | Implies the user is caged — patronizing and overused | Name what becomes possible: "ship to production in minutes" |
| **Seamless** | The #1 most overused SaaS adjective — means nothing | Describe the actual experience: "one-click deployment" or "zero-config setup" |
| **Revolutionize** | Hyperbolic — almost nothing in SaaS is revolutionary | Name the specific change: "replaces 3 tools with one" |
| **Empower** | Vague and patronizing — implies the user is powerless | Name the capability: "gives engineering leads full visibility" |
| **Robust** | Generic filler adjective — what does robust mean? | Name the quality: "handles 10K concurrent connections" |
| **Unlock** | Metaphor without specificity — unlock what, exactly? | Name the outcome: "access real-time analytics" |
| **Leverage** | Corporate jargon — no human says "leverage" conversationally | Use "use", "apply", or name the specific action |

### Extended Banned Vocabulary

These additional words should be evaluated for banning on a per-brand basis:

| Word | Risk | When to Ban | When to Allow |
|------|------|-------------|---------------|
| **Cutting-edge** | Generic technology claim | When no proof of novelty | If genuinely first-to-market with proof |
| **Best-in-class** | Unverifiable superlative | When no benchmark exists | If you have third-party comparison data |
| **Next-generation** | Vague future promise | When current product ships | If replacing a known legacy architecture |
| **Scalable** | Every SaaS claims this | When no scale metrics exist | If you can cite specific numbers |
| **Innovative** | Self-congratulatory | Always (show, don't tell) | Almost never — let features speak |
| **Streamline** | Vague process improvement | When the improvement isn't quantified | If paired with specific time savings |
| **Holistic** | Meaningless in tech context | Always | Never |
| **Synergy** | Corporate cliche | Always | Never |
| **End-to-end** | Overused scope claim | When not literally true | If the product genuinely covers full lifecycle |
| **Game-changing** | Hyperbolic | Always (show, don't tell) | Almost never |
| **State-of-the-art** | Unverifiable | When no benchmark | If citing specific technical advance |
| **Turnkey** | Often misleading about effort | When setup actually required | If genuinely zero-config |
| **Frictionless** | Same as seamless — overused | Same as seamless | Almost never |
| **World-class** | Unverifiable superlative | Always | Never |
| **Bleeding-edge** | Implies instability | Always for marketing | Acceptable in technical blog posts |
| **Disruptive** | Self-awarded, meaningless | When market hasn't validated | If industry analysts use the term |
| **Mission-critical** | Borrowed gravitas | When product isn't infrastructure | If genuinely in the critical path |
| **Effortless** | Dismisses real complexity | When onboarding exists | Almost never — effort is honest |
| **Paradigm** | Academic jargon | Always in marketing | Acceptable in whitepapers |
| **Transform** | Overused, vague | When not describing measurable change | If paired with before/after metrics |

---

## "Expensive" vs "Generic" Copy

### Comparison Table

| Dimension | Generic Copy | Expensive Copy |
|-----------|-------------|---------------|
| **Adjectives** | Seamless, robust, powerful | Zero adjectives — nouns and verbs only |
| **Claims** | "Best-in-class performance" | "p99 latency under 50ms" |
| **Sentences** | 20+ words, compound, hedging | 8-12 words, simple, declarative |
| **Structure** | Wall of text paragraphs | Short statement → proof point → short statement |
| **Specificity** | "Helps teams collaborate" | "Sync 500 contributors across 12 time zones" |
| **Evidence** | Adjectives as evidence | Numbers, names, screenshots as evidence |
| **Tone** | Trying to impress | Confident enough to be plain |

### The Grayscale Test

Premium copy works without visual design. Test your copy by stripping all styling:

```
GRAYSCALE TEST PROCEDURE:

1. Remove all colors, images, icons, and styling
2. Set all text to black on white, single font, single size
3. Read the copy in this stripped state
4. Ask: "Does this still feel authoritative and specific?"

If YES → Copy carries its own weight
If NO  → Copy depends on design to mask weakness

Examples that PASS the Grayscale Test:
- "Deploy to 35 edge regions in under 10 seconds."
- "Used by 6 of the Fortune 10 to process $2B+ daily."

Examples that FAIL the Grayscale Test:
- "A powerful platform for modern teams."
- "Seamlessly integrate with your existing workflow."
```

---

## Before/After Transformations

### Transformation 1: Hero Section

```
BEFORE (generic):
"Empower your team with our powerful, seamless platform
that revolutionizes how you work."

AFTER (expensive):
"Ship audit-ready code in half the time.
12,000 engineering teams. Zero compliance failures."
```

### Transformation 2: Feature Description

```
BEFORE (generic):
"Our robust analytics engine provides cutting-edge insights
that help you make better decisions and drive growth."

AFTER (expensive):
"Query 10B rows in under 2 seconds.
No pre-aggregation. No data warehouse required."
```

### Transformation 3: CTA Section

```
BEFORE (generic):
"Ready to unlock the full potential of your team?
Get started with our innovative platform today!"

AFTER (expensive):
"Start building in 90 seconds.
Free tier. No credit card."
```

### Transformation 4: Testimonial Framing

```
BEFORE (generic):
"A game-changing tool that transformed our business."
- Happy Customer

AFTER (expensive):
"Cut our release cycle from 3 weeks to 2 days.
Paid for itself in the first sprint."
- Sarah Chen, VP Engineering, Lattice (220 engineers)
```

---

## Voice Consistency Checklist

Use this checklist to validate that copy adheres to the defined voice:

- [ ] All 3 vibe adjectives are observable in the copy
- [ ] No anti-vibe violations detected (copy doesn't cross boundaries)
- [ ] Zero banned words from the default list appear in copy
- [ ] Zero banned words from the product-specific extended list appear
- [ ] Sentences average under 15 words
- [ ] Active voice used in 90%+ of sentences
- [ ] No more than 2 adjectives per sentence
- [ ] Every claim has adjacent proof (number, name, or example)
- [ ] Copy passes the Grayscale Test (works without design)
- [ ] Tone is consistent across all sections (no jarring shifts)

---

## Instructions

When defining brand voice:

1. **Review positioning and ICP** from `writing-brand-positioning` — voice must serve the audience
2. **Select the closest archetype(s)** from the Vibe Spectrum table
3. **Choose exactly 3 vibe adjectives** — test each against the specific/observable/differentiating criteria
4. **Derive 3 anti-vibes** using the failure mode method
5. **Confirm the default banned vocabulary** (8 words) and select additional bans from the extended list
6. **Write boundary statements** for each vibe/anti-vibe pair
7. **Produce a before/after example** that demonstrates the voice in action
8. **Output Fields 5-7** of the Brand Packet:
   - Field 5: Vibe (3 adjectives with definitions)
   - Field 6: Anti-Vibe (3 failure modes with boundaries)
   - Field 7: Banned Vocabulary (default + product-specific additions)

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Adjective inflation** | Picking 5+ vibe words dilutes each one | Force-rank to 3. If you can't cut, they're too similar |
| **Generic adjectives** | "Professional, reliable, modern" — every brand claims these | Apply the differentiation test: would a competitor say the same? |
| **No anti-vibes** | Without boundaries, vibes drift into failure modes | Always define the failure mode for each vibe |
| **Aspirational voice** | Defining voice you want, not voice you can sustain | Test against existing copy — can current team write this way? |
| **Voice as personality** | "Our brand is like a friendly expert" — too abstract | Voice = specific word choices, not personality traits |
| **Inconsistent application** | Voice defined but not enforced in actual copy | Use the checklist on every piece of copy |

---

## Edge Cases

### Multi-Product Brands
- Define a master voice with per-product modulation
- Example: Stripe has consistent voice but Stripe Docs is warmer than Stripe.com

### Technical vs. Marketing Copy
- Same 3 adjectives apply, but weight shifts
- Marketing: lean into the most accessible adjective
- Docs: lean into the most precise adjective
- Blog: lean into the most human adjective

### International/Localized Copy
- Voice adjectives translate to intent, not literal words
- "Concise" in German may mean longer sentences than in English
- Ban list applies concept-by-concept, not word-by-word

### Brand Evolution
- Voice should be re-evaluated annually or at major pivots
- Adjectives may shift as audience matures (startup → enterprise)
- Banned vocabulary only grows — never remove a ban

---

## Related Skills and Agents

- **`writing-brand-positioning` skill**: Provides positioning, ICP, and differentiators that voice must serve
- **`reviewing-copy` skill**: Enforces voice constraints during editorial review
- **`validating-brand-packets` skill**: Validates voice fields for completeness and quality
- **`copy_writer` agent**: Writes copy using the defined voice constraints
- **`auditing-trust-engineering` skill**: Trust messaging must align with voice tone

**When to use which**: Use this skill to define voice guidelines (Fields 5-7). Use `reviewing-copy` to enforce them on actual copy. Use `copy_writer` to generate new copy within voice constraints.
