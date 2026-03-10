---
name: reviewing-copy
version: 1.0.0
user-invocable: false
description: Anti-generic editorial pass that detects and replaces banned vocabulary, generic adjectives, hyperbolic claims, and filler patterns in marketing copy. Use when reviewing draft copy, decontaminating generic language, enforcing voice constraints, or polishing marketing text. Triggered by: copy review, editorial pass, generic copy, banned words, copy decontamination, anti-generic, copy polish, headline review, filler detection, hyperbolic claims, copy specificity, marketing edit.
allowed-tools: Read, Glob, Grep, Edit
---

# Anti-Generic Editorial Pass

## Purpose

Detect and eliminate generic, hyperbolic, and filler language from marketing copy. This skill operates as a standalone editorial pass that can run on any marketing text — page copy, headlines, feature descriptions, testimonial framing, CTAs, and metadata.

Generic copy is the single biggest reason marketing pages underperform. It doesn't inform, doesn't differentiate, and doesn't convert. This skill provides systematic detection rules and replacement methodology to transform generic copy into "expensive" copy — the kind that feels premium because every word earns its place.

---

## 5 Detection Rules

### Rule 1: Banned Vocabulary Check

Scan for the default banned vocabulary and any product-specific additions from the Brand Packet (Field 7).

**Default Banned Words** (always flag):

| Word | Detection Pattern | Why It's Banned |
|------|------------------|-----------------|
| seamless | `seamless(ly)?` | Most overused SaaS adjective — carries zero information |
| robust | `robust(ness)?` | Filler adjective — what does robust actually mean? |
| leverage | `leverag(e\|es\|ed\|ing)` | Corporate jargon — no human speaks this way |
| empower | `empower(s\|ed\|ing\|ment)?` | Patronizing — implies user is powerless |
| supercharge | `supercharg(e\|es\|ed\|ing)` | Hyperbolic — meaningless metaphor |
| unleash | `unleash(es\|ed\|ing)?` | Hyperbolic — implies user is caged |
| revolutionize | `revolutioniz(e\|es\|ed\|ing)` | Almost nothing in SaaS is revolutionary |
| unlock | `unlock(s\|ed\|ing)?` | Vague metaphor — unlock what, specifically? |

**Replacement Strategy**: Don't replace with synonyms. Replace with specific outcomes:
- "Seamless integration" → "Connect in 3 clicks. No middleware required."
- "Robust analytics" → "Query 10B rows in under 2 seconds."
- "Leverage your data" → "Use your existing Postgres database directly."
- "Empower your team" → "Give every engineer deploy access in 5 minutes."

### Rule 2: Generic Adjective Identification

Flag adjectives that describe nothing specific. Generic adjectives are the verbal equivalent of stock photos — they fill space without adding information.

**Generic Adjective Detection Table**:

| Generic Adjective | Why It's Generic | Replacement Strategy |
|-------------------|-----------------|---------------------|
| powerful | Every product claims power | Name the specific capability |
| innovative | Self-awarded, unverifiable | Describe what's new specifically |
| comprehensive | Vague scope claim | List the specific things included |
| intuitive | Subjective, unverifiable | Describe the specific UX: "keyboard-first" |
| cutting-edge | Cliche tech buzzword | Name the specific technology |
| next-generation | Undefined future promise | Name what changed from previous generation |
| enterprise-grade | Meaningless modifier | Name the specific enterprise feature |
| best-in-class | Unverifiable superlative | Cite a specific benchmark or comparison |
| state-of-the-art | Academic cliche | Name the specific technical advance |
| world-class | Unverifiable, self-congratulatory | Name the specific achievement |
| advanced | Relative to what? | Name the specific capability |
| modern | Compared to what? | Name the specific architectural choice |
| smart | Anthropomorphizing software | Describe the specific automation |
| flexible | Everything claims flexibility | Describe the specific customization |
| scalable | Every SaaS claims this | Name the specific scale numbers |

**Replacement Methodology**:
```
Generic adjective → Concrete noun or verb

"A powerful analytics platform"
→ "Query 10B rows. Sub-second response. No pre-aggregation."

"An intuitive interface"
→ "Keyboard-first. 47 shortcuts. Zero training required."

"A comprehensive security solution"
→ "SAST, DAST, SCA, and secrets scanning in one pipeline."
```

### Rule 3: Hyperbolic Pattern Detection

Flag patterns that over-promise or use superlatives without evidence.

**Detection Patterns** (regex):

| Pattern | Regex | Example Match |
|---------|-------|---------------|
| Superlative without proof | `(best\|fastest\|most\|only\|first)\s+\w+` | "The fastest database" |
| "The" + category claim | `[Tt]he\s+(leading\|premier\|ultimate\|definitive)` | "The leading platform" |
| Absolute claims | `(never\|always\|every\|all)\s+(fail\|break\|need)` | "Never fails" |
| Percentage hyperbole | `\d{3,}%\s+(faster\|better\|more)` | "1000% faster" |
| Transformation claims | `(transform\|revolutionize\|disrupt)\s+your` | "Transform your workflow" |
| Silver bullet framing | `(everything\|anything)\s+you\s+need` | "Everything you need" |
| Infinite scope | `(unlimited\|infinite\|endless)\s+\w+` | "Unlimited possibilities" |

**Replacement Strategy**: Replace hyperbole with specific, provable claims:
```
"The fastest database on the market"
→ "p99 read latency under 5ms. Benchmark: [link]"

"Transform your workflow"
→ "Cut PR review time from 2 hours to 15 minutes."

"Everything you need in one platform"
→ "Issue tracking, code review, CI/CD, and deployment. One login."
```

### Rule 4: Repetitive Structure Detection

Flag structural repetition that creates monotonous reading rhythm.

**Detection Patterns**:

| Pattern | Detection Method | Example |
|---------|-----------------|---------|
| Same sentence opening | 3+ sentences starting with same word | "Our product... Our team... Our mission..." |
| Parallel adjective stacking | 3+ items with same structure | "Fast, reliable, and scalable" repeated |
| Bullet point monotony | All bullets same length and structure | Every bullet is "[Verb] your [noun]" |
| Section opening formula | Each section starts identically | "With [Product], you can..." × 5 |
| CTA repetition | Same CTA copy used 3+ times | "Get Started" appears 6 times unchanged |

**Fix**: Alternate between:
- Short declarative statement (5-8 words)
- Supporting evidence with specifics (12-20 words)
- This creates a **rhythm**: punch → proof → punch → proof

```
MONOTONOUS:
"Deploy faster. Scale easier. Monitor better. Debug quicker."

RHYTHMIC:
"Deploy in 90 seconds.                    ← punch
Zero config. Your git repo is enough.     ← proof
Scale to 100K requests without thinking.  ← punch
Auto-scaling, edge-cached, 35 regions.    ← proof"
```

### Rule 5: Filler Word Detection

Flag words and phrases that add length without adding meaning.

**Filler Patterns**:

| Filler | Detection | Replacement |
|--------|-----------|-------------|
| "In order to" | Always replaceable | "To" |
| "Helps you to" | Hedging construction | "[Direct verb]" |
| "Is able to" | Passive filler | "[Verb]s" |
| "That being said" | Transition filler | Delete entirely |
| "It is important to note that" | Throat-clearing | Delete entirely |
| "At the end of the day" | Cliche | Delete entirely |
| "In today's world" | Time filler | Delete entirely |
| "When it comes to" | Filler prepositional | Delete entirely |
| "The fact that" | Nominalizing filler | Delete, restructure sentence |
| "As a matter of fact" | Emphasis filler | Delete entirely |
| "Very" / "Really" / "Truly" | Intensifier crutch | Use a stronger base word |
| "Quite" / "Rather" / "Somewhat" | Hedging | Remove the hedge — commit to the claim |
| "Basically" / "Essentially" | Minimizing | Delete — say the thing directly |

---

## Typographic Rhythm Enforcement

### The Alternating Pattern

Marketing copy should alternate between two modes:

```
┌─────────────────────────────────────────────────────────────────┐
│                    TYPOGRAPHIC RHYTHM                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MODE 1: SHORT STATEMENT (punch)                                 │
│  • 3-8 words                                                     │
│  • Bold or large text                                            │
│  • Makes a claim or names a benefit                              │
│  • Creates visual anchor                                         │
│                                                                  │
│  MODE 2: DESCRIPTIVE PROOF (support)                             │
│  • 10-20 words                                                   │
│  • Regular weight, smaller or secondary text                     │
│  • Provides evidence, specifics, or context                      │
│  • Contains numbers, names, or concrete details                  │
│                                                                  │
│  Pattern: Statement → Proof → Statement → Proof                  │
│                                                                  │
│  EXAMPLE:                                                        │
│  "Ship code, not configs."              ← statement              │
│  Push to git. Builds deploy in 90       ← proof                  │
│  seconds to 35 edge regions.                                     │
│                                                                  │
│  "Scale without thinking."              ← statement              │
│  Auto-scaling handles 0 to 100K         ← proof                  │
│  req/s. Zero configuration required.                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 3-Sentence Maximum

No paragraph in marketing copy should exceed 3 sentences. If it does:

1. Split into statement + proof pattern
2. Move supporting details to a sub-section or tooltip
3. Ask: "Is all of this necessary above the fold?"

---

## Headline Rules

### Requirements

| Rule | Description | Example |
|------|-------------|---------|
| **Concrete benefit** | Headlines state a specific outcome, not a generic promise | "Deploy to 35 regions in 90 seconds" not "Deploy anywhere" |
| **Technical mechanism** | Alternative: name the specific how | "Git push to production" not "Simplified deployment" |
| **No wordplay** | Puns and clever phrasing reduce clarity | "Fast builds" not "Building the future of speed" |
| **4-8 words** | Headlines are scanned, not read | "Query 10B rows instantly" (5 words) |
| **No banned words** | Headlines are highest-scrutiny copy | Zero tolerance for banned vocabulary |

### Headline Before/After

```
BEFORE: "Empower Your Team with Seamless Collaboration"
AFTER:  "Ship 3x faster. One shared workspace."

BEFORE: "The Next-Generation Platform for Modern Teams"
AFTER:  "Issue tracking that keeps up with your keyboard."

BEFORE: "Unlock the Power of Your Data"
AFTER:  "Query 10B rows. Sub-second. No pre-aggregation."

BEFORE: "Revolutionizing the Way You Build Software"
AFTER:  "From git push to production in 90 seconds."
```

---

## Subheadline Rules

### Requirements

| Rule | Description | Example |
|------|-------------|---------|
| **Exact numbers** | Always prefer specific numbers over vague claims | "35 edge regions" not "global infrastructure" |
| **Constraints** | Name what's NOT required | "No credit card. No configuration." |
| **Timeframes** | Include temporal specificity | "Start in 90 seconds" not "Quick setup" |
| **1-2 sentences** | Subheadlines amplify, not repeat | Adds new information to headline |
| **Proof-adjacent** | Directly supports the headline claim | If headline claims speed, subhead gives the number |

---

## "Expensive Copy" Checklist

Use this 10-point checklist on any piece of marketing copy:

- [ ] Zero banned words from default + product-specific list
- [ ] Zero generic adjectives (powerful, innovative, comprehensive, etc.)
- [ ] Every claim has adjacent proof (number, name, or link)
- [ ] Average sentence length < 15 words
- [ ] No paragraph exceeds 3 sentences
- [ ] Headlines use concrete benefits or technical mechanisms
- [ ] Subheadlines include exact numbers, constraints, or timeframes
- [ ] Alternating rhythm: statement → proof → statement → proof
- [ ] No filler phrases (in order to, helps you to, when it comes to)
- [ ] Copy passes the Grayscale Test (works without design to carry it)

---

## Before/After Decontamination Examples

### Example 1: Hero Section

```
BEFORE:
──────────────────────────────────────────────────────────
The Most Powerful Platform for Modern Development Teams

Seamlessly integrate your tools and workflows to unlock
unprecedented productivity. Our comprehensive, enterprise-grade
solution empowers teams of all sizes to build, deploy, and
scale with confidence. Leverage cutting-edge AI to supercharge
your development pipeline and revolutionize how you ship code.

[Get Started Free]  [Book a Demo]
──────────────────────────────────────────────────────────

DETECTED ISSUES:
• Banned words: seamlessly, unlock, empowers, leverage,
  supercharge, revolutionize (6 violations)
• Generic adjectives: powerful, comprehensive, enterprise-grade,
  cutting-edge, unprecedented (5 violations)
• Hyperbole: "most powerful", "unprecedented" (2 violations)
• Filler: none
• Sentence length: 22 words average (exceeds 15)
• No metrics or proof

AFTER:
──────────────────────────────────────────────────────────
From git push to production in 90 seconds.

One pipeline for build, test, and deploy. 35 edge regions.
Zero configuration. Used by 12,000+ engineering teams
including Stripe, Vercel, and Lattice.

[Start building — free]  [See it in 2 minutes]
──────────────────────────────────────────────────────────
```

### Example 2: Feature Section

```
BEFORE:
──────────────────────────────────────────────────────────
Advanced Monitoring & Analytics

Our robust monitoring suite provides comprehensive visibility
into your infrastructure. With powerful dashboards and intuitive
visualizations, you can easily identify issues and optimize
performance. Our cutting-edge alerting system ensures you never
miss a critical event, helping you maintain seamless operations
across your entire stack.
──────────────────────────────────────────────────────────

DETECTED ISSUES:
• Banned words: robust, seamless (2 violations)
• Generic adjectives: advanced, comprehensive, powerful,
  intuitive, cutting-edge (5 violations)
• Filler: "helping you maintain" (1 violation)
• Sentence length: 18 words average
• No metrics or proof

AFTER:
──────────────────────────────────────────────────────────
See every request. Sub-millisecond resolution.

Trace requests across 200+ services in one view.
Alerts fire in under 3 seconds. 99.99% of anomalies
caught before customers notice.

Custom dashboards in 60 seconds. No query language required.
──────────────────────────────────────────────────────────
```

### Example 3: Testimonial Section

```
BEFORE:
──────────────────────────────────────────────────────────
What Our Customers Say

"This is an amazing, game-changing tool that has truly
transformed how our team works. The seamless experience
and powerful features have revolutionized our entire
workflow. Highly recommended!"
— Happy Customer
──────────────────────────────────────────────────────────

DETECTED ISSUES:
• Banned words: seamless, revolutionized (2 violations)
• Generic adjectives: amazing, game-changing, powerful (3)
• Hyperbole: "truly transformed", "entire workflow" (2)
• No name, no role, no company, no metric
• No specifics about what changed

AFTER:
──────────────────────────────────────────────────────────
"Cut our release cycle from 3 weeks to 2 days.
Paid for itself in the first sprint."

— Sarah Chen, VP Engineering, Lattice
  220 engineers across 4 time zones
──────────────────────────────────────────────────────────
```

---

## Review Output Format

When performing an editorial review, produce this structured output:

```markdown
## Copy Review: [Page/Section Name]

### Detection Summary

| Rule | Violations | Severity |
|------|-----------|----------|
| 1. Banned Vocabulary | [count] | Critical |
| 2. Generic Adjectives | [count] | Major |
| 3. Hyperbolic Patterns | [count] | Major |
| 4. Repetitive Structure | [count] | Minor |
| 5. Filler Words | [count] | Minor |

**Total Violations**: [sum]

### "Expensive Copy" Checklist

- [x/✗] Zero banned words
- [x/✗] Zero generic adjectives
- [x/✗] Every claim has adjacent proof
- [x/✗] Average sentence length < 15 words
- [x/✗] No paragraph exceeds 3 sentences
- [x/✗] Headlines use concrete benefits
- [x/✗] Subheadlines include exact numbers
- [x/✗] Alternating statement/proof rhythm
- [x/✗] No filler phrases
- [x/✗] Passes Grayscale Test

**Score**: [count passing]/10

### Changes Made

| Location | Original | Revised | Rule Applied |
|----------|----------|---------|-------------|
| [section] | "[original text]" | "[revised text]" | [rule #] |

### Revised Copy

[Full revised copy with all changes applied]

### Metrics Needed

[List of bracketed placeholders that need real data, e.g.,
"[X customers]" — source from product analytics]
```

---

## Instructions

When performing an editorial review:

1. **Read the complete copy** before making any changes
2. **Run all 5 detection rules** systematically — don't fix while detecting
3. **Catalog all violations** by rule and severity
4. **Replace banned vocabulary** with specific outcomes (not synonyms)
5. **Replace generic adjectives** with concrete nouns and verbs
6. **Replace hyperbolic claims** with provable specifics or `[bracketed placeholders]`
7. **Fix structural repetition** by introducing statement/proof alternation
8. **Remove all filler** words and phrases
9. **Apply the "Expensive Copy" checklist** to the revised version
10. **If a Brand Packet exists**, validate against voice constraints (Fields 5-7)
11. **Produce the Review Output** using the template above

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Synonym substitution** | Replacing "seamless" with "smooth" | Replace the entire construction, not just the word |
| **Metric fabrication** | Inventing specific numbers | Use `[bracketed placeholders]` for unknown metrics |
| **Over-editing voice** | Stripping all personality during review | Preserve the 3 vibe adjectives while removing violations |
| **Selective scanning** | Only checking headlines, skipping body copy | Run detection rules on ALL text including CTAs and metadata |
| **Copy lengthening** | Adding proof makes copy longer than original | If adding proof, cut filler elsewhere to compensate |

---

## Edge Cases

### Technical Documentation
- Relax generic adjective rules for genuinely technical terms ("advanced configuration")
- Filler rules still apply
- Banned vocabulary still applies
- Metrics may be less relevant (accuracy matters more)

### Copy with Intentional Voice Breaks
- Some brands use banned words ironically or in quotes
- Flag but don't auto-replace — note for human review
- Example: "They said it was 'seamless.' It wasn't. Here's what actually happened."

### Short-Form Copy (CTAs, Buttons, Tags)
- 2-4 word copy can't follow all rules
- Prioritize: no banned words > specific verb > proof
- "Start building" > "Get Started" > "Try It" > "Unlock Now"

### Copy in Non-English Languages
- Banned vocabulary applies conceptually, not literally
- Each language has its own filler words — identify equivalents
- Rhythm rules may differ (German sentences are longer by nature)

---

## Related Skills and Agents

- **`writing-brand-voice` skill**: Defines the voice constraints (Fields 5-7) this skill enforces
- **`validating-brand-packets` skill**: Validates Brand Packet completeness; this skill validates copy quality
- **`writing-brand-positioning` skill**: Positioning provides the claims that need proof
- **`writing-trust-signals` skill**: Trust signals provide the proof that replaces generic claims
- **`copy_writer` agent**: Generates draft copy; this skill reviews and polishes it
- **`auditing-trust-engineering` skill**: Trust messaging must survive this editorial pass

**When to use which**: Use this skill as a post-writing editorial pass on any marketing copy. Use `writing-brand-voice` to define the rules. Use `copy_writer` to generate the initial draft. This skill is the quality gate between draft copy and published copy.
