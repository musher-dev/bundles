---
name: writing-wireframe-copy
version: 1.0.0
user-invocable: false
description: Write exact raw copy for every content slot defined in an Information Architecture document, governed by Brand Packet voice constraints. Output is 90% final wireframe copy ready for Stage 3 editorial pass. Use when writing landing page copy, filling content slots, producing wireframe text, or generating section-by-section marketing copy. Triggered by: wireframe copy, landing page copy, content slots, section copy, marketing copy, copy writing, headline writing, subheadline writing, CTA copy, feature card copy, testimonial framing, wireframe text.
allowed-tools: Read, Glob, Grep, Write
---

# Wireframe Copy Writing

## Purpose

Take an Information Architecture document (`brand/information-architecture.md`) and a Brand Packet (`brand/brand-packet.md`), then write exact raw copy for every content slot defined in the IA. The output is a complete wireframe copy document — every headline, subheadline, body paragraph, CTA, feature card, metric label, and testimonial frame — ready for the Stage 3 editorial pass.

This skill writes copy **within the immutable IA structure**. It does NOT modify section sequencing, add sections, remove sections, change archetypes, or alter layout patterns. The IA is the structural contract. This skill fills it with words.

The output should be 90%+ final — the Stage 3 editorial pass (`reviewing-copy`) catches remaining generic language, banned vocabulary violations, and rhythm issues, but should not need to rewrite from scratch.

---

## Input Contract

This skill requires two inputs and derives copy governance from both:

### Primary Input: Information Architecture Document

Read from `brand/information-architecture.md`. Must contain:
- Section definitions with archetypes, layout patterns, and content slots
- Content slot types with constraints
- Trust mechanic assignments per section
- Differentiator links per section

### Secondary Input: Brand Packet

Read from `brand/brand-packet.md`. Specific fields govern copy decisions:

| Brand Packet Field | Copy Governance | How It's Used |
|--------------------|----------------|---------------|
| **Field 1: Positioning Statement** | Hero headline and subheadline derivation | 5-slot formula provides the raw material for the most important copy on the page |
| **Field 2: ICP** | Pain-point language, audience mirroring | Copy must use the ICP's vocabulary and address their specific context |
| **Field 3: Anti-ICP** | Objection handler copy, self-selection language | Copy in objection sections must address Anti-ICP concerns without alienation |
| **Field 4: Differentiators** | Feature card copy, mechanism descriptions | Each differentiator's capability + proof becomes card copy |
| **Field 5: Vibe** | Voice formula — all copy embodies all 3 adjectives simultaneously | Every sentence must pass the "does this sound [adj1], [adj2], and [adj3]?" test |
| **Field 6: Anti-Vibe** | Copy boundary enforcement | Every sentence must NOT sound like any of the 3 anti-vibes |
| **Field 7: Banned Vocabulary** | Pre-write extraction, post-write verification | Extract full ban list before writing; verify zero violations after writing |
| **Field 8: Trust Signals** | Metric copy, testimonial framing, proof language | Exact numbers, quotes, and badges from the catalog become copy |

---

## Copy Voice Rules

### Voice Formula

All copy must embody all 3 vibe adjectives (Field 5) simultaneously. Not one per section — all three in every sentence.

```
VOICE TEST — apply to every sentence:

"Does this sentence sound [adjective 1], [adjective 2], and [adjective 3]?"

If the sentence fails ANY adjective:
  → Rewrite until all 3 are present

If the sentence triggers ANY anti-vibe (Field 6):
  → Rewrite to stay within the boundary

Example for vibes "Precise, Direct, Understated":
  FAIL: "Our amazing platform helps you do incredible things" (not precise, not direct, not understated)
  PASS: "Deploy to 35 regions. One command." (precise: specific number, direct: imperative, understated: no hyperbole)
```

### Sentence-Level Rules

| Rule | Constraint | Enforcement |
|------|-----------|-------------|
| **Average sentence length** | Maximum 15 words average across the page | Count after writing; rewrite long sentences |
| **Paragraph maximum** | No paragraph exceeds 3 sentences | Split longer paragraphs into statement + proof pairs |
| **Active voice** | 90%+ of sentences use active voice | Scan for "is/was/been [verb]ed" patterns; rewrite to active |
| **Front-loaded value** | Key information in first 5 words of each sentence | Move qualifiers and preambles to the end or delete them |
| **Adjective ceiling** | Maximum 2 adjectives per sentence | If more than 2, replace adjectives with nouns or verbs |
| **Filler-free** | Zero filler phrases | No "in order to", "helps you to", "when it comes to", "it is important to note" |

### Headline Formulas

Use one of these 5 approved headline patterns. Every headline on the page must fit one pattern:

| Pattern | Formula | Example | Best For |
|---------|---------|---------|----------|
| **Outcome** | [Measurable result] + [constraint/speed] | "Deploy to 35 regions in 90 seconds." | Hero, CTA Block |
| **Contrast** | [Old way problem]. [New way solution]. | "3 tools. 3 logins. 3 dashboards. Or one." | Mechanism, Value Grid |
| **Technical Truth** | [Specific mechanism] + [why it matters] | "Git push to production. No CI config." | Mechanism, developer tools |
| **Mechanism** | [How it works] in [N words] | "One YAML file. Full pipeline." | Mechanism, technical products |
| **Metric** | [Number] + [what it means] | "12,000 teams ship with confidence." | Proof Block, Social Proof |

**Headline constraints**:
- 4-8 words (headlines are scanned, not read)
- Zero banned words (headlines are highest-scrutiny copy)
- Concrete benefit or technical mechanism (never abstract promise)
- No wordplay, puns, or clever phrasing (reduces clarity)

### Subheadline Formulas

Use one of these 4 approved subheadline patterns:

| Pattern | Formula | Example | Best For |
|---------|---------|---------|----------|
| **Hyper-specific** | [Exact number] + [exact capability] | "35 edge regions. Sub-second deploys. Zero config." | Hero, Value Grid |
| **Negation** | [What's NOT required] | "No credit card. No configuration. No vendor lock-in." | Hero, CTA Block |
| **Amplification** | [Expand on headline with proof] | "Used by 12,000 engineering teams including Stripe and Vercel." | Hero, Proof Block |
| **Mechanism detail** | [One-sentence explanation of how] | "Push to your repo. Builds deploy automatically to every region." | Mechanism |

**Subheadline constraints**:
- 1-2 sentences (amplifies headline, doesn't repeat it)
- Must add NEW information not in the headline
- Include numbers, names, or constraints when possible
- Proof-adjacent: directly supports the headline claim

---

## Section-Type Copy Patterns

For each IA archetype, here are the content slot deliverables, derivation methods, and concrete examples.

### Hero Copy

| Slot | Derivation Method | Example |
|------|------------------|---------|
| `headline` | Distill Field 1 outcome + mechanism into 4-8 words using Outcome or Technical Truth pattern | "From git push to production in 90 seconds." |
| `subheadline` | Amplify headline with Field 2 ICP context + Field 8 primary metric using Amplification or Negation pattern | "One pipeline for build, test, and deploy. Used by 12,000 engineering teams." |
| `cta-primary` | Use Field 1 mechanism verb + outcome. Never "Get Started" or "Learn More" | "Start building — free" |
| `cta-secondary` | Lower-commitment alternative. Reference docs, demo, or proof | "See it in 2 minutes" |

### Social Proof Bar Copy

| Slot | Derivation Method | Example |
|------|------------------|---------|
| `metric-value` | Largest scale metric from Field 8, hyper-specific | "12,847" |
| `metric-label` | Context that makes the number meaningful | "companies ship with confidence" |
| `logo-set` | No copy — logos from Field 8 logo catalog | [Visual element, not copy] |

### Mechanism Copy

| Slot | Derivation Method | Example |
|------|------------------|---------|
| `section-headline` | Name the mechanism from Field 1 using Mechanism or Technical Truth pattern | "One YAML file. Full pipeline." |
| `section-subheadline` | Explain what this means for the ICP using Mechanism detail pattern | "Define your build, test, and deploy stages in a single config. Push to git — it runs." |
| `body-paragraph` | 2-3 sentences expanding on the mechanism's unique architecture (Field 4 proof) | "No plugin marketplace. No third-party integrations. Your pipeline definition is code, versioned alongside your application." |
| `annotation-labels` | If using annotated-screenshot, label specific UI/code elements | "Config file", "Build log", "Deploy status" |

### Value Grid Copy

| Slot | Derivation Method | Example |
|------|------------------|---------|
| `section-headline` | Frame the differentiators collectively using Outcome pattern | "Everything your pipeline needs. Nothing it doesn't." |
| `feature-card-title` | Distill each Field 4 differentiator capability into 3-6 words | "Sub-second builds" |
| `feature-card-description` | 1-2 sentences: specific outcome + proof from Field 4 | "Incremental builds cache at the file level. Average build time: 1.2 seconds for a 200K LOC monorepo." |

### Proof Block Copy

| Slot | Derivation Method | Example |
|------|------------------|---------|
| `quote-text` | Direct from Field 8 quotes — do NOT modify or paraphrase | "'Cut our release cycle from 3 weeks to 2 days. Paid for itself in the first sprint.'" |
| `quote-attribution` | Name + Role + Company from Field 8 | "Sarah Chen, VP Engineering, Lattice" |
| `metric-value` | Performance or scale metric from Field 8 | "99.997%" |
| `metric-label` | Context label | "uptime over 36 months" |

### CTA Block Copy

| Slot | Derivation Method | Example |
|------|------------------|---------|
| `cta-headline` | Restate the primary outcome from Field 1 using Outcome pattern; different words than Hero | "Ship your first build in 90 seconds." |
| `cta-subheadline` | Friction reducers + social proof using Negation or Amplification pattern | "Free tier. No credit card. Join 12,000 teams." |
| `cta-primary` | Same as or variation of Hero CTA | "Start building — free" |
| `cta-secondary` | Alternative path for different buyer readiness | "Talk to sales" |
| `friction-reducer` | 2-3 risk-removal statements from Field 8 or product reality | "No credit card required", "Free tier included", "Cancel anytime" |

### Objection Handler Copy

| Slot | Derivation Method | Example |
|------|------------------|---------|
| `section-headline` | Frame as helpful, not defensive | "Common questions" |
| `objection-question` | Written as ICP would phrase it (Field 2 vocabulary) | "How is this different from GitHub Actions?" |
| `objection-answer` | 2-4 sentences: acknowledge concern, provide proof, link to evidence | "GitHub Actions is a general-purpose CI system. We're purpose-built for [category]. Customers report [metric] compared to their previous Actions setup." |

### Community Block Copy

| Slot | Derivation Method | Example |
|------|------------------|---------|
| `section-headline` | Celebrate the community, don't sell | "Built with the community" |
| `community-metric` | Exact numbers from Field 8 community signals | "40,847 GitHub stars" |
| `community-link` | Channel name + purpose | "Join 15K developers on Discord" |

---

## Placeholder Protocol

When real metrics, quotes, or data are unavailable, use bracketed placeholders instead of inventing data.

### Format

```
[PLACEHOLDER: description of needed data — suggested source]
```

### Examples

```
"Trusted by [PLACEHOLDER: exact customer count — pull from billing dashboard] companies."

"[PLACEHOLDER: named customer quote about deployment speed — request from customer success]"

"p99 latency under [PLACEHOLDER: actual p99 latency — pull from Datadog APM] ms."
```

### Rules

1. **Describe the needed data** — not just "[number]" but "[exact customer count]"
2. **Suggest the source** — where the team should look to find this data
3. **Target fewer than 5 placeholders per page** — if more than 5 data points are missing, the Brand Packet trust signals (Field 8) need more work
4. **Never fabricate** — a placeholder is always better than an invented metric
5. **Placeholders appear in the output's Placeholders Table** — collected in one place for easy resolution

---

## Banned Vocabulary Enforcement

### Pre-Write Extraction

Before writing ANY copy, extract the complete banned vocabulary list:

1. Read the 8 default banned words from Field 7: supercharge, unleash, seamless, revolutionize, empower, robust, unlock, leverage
2. Read any product-specific banned words from Field 7's extended list
3. Read the generic adjective ban list: powerful, innovative, comprehensive, intuitive, cutting-edge, next-generation, enterprise-grade, best-in-class, state-of-the-art, world-class, advanced, modern, smart, flexible, scalable
4. Hold this complete list in working memory during all copy writing

### Post-Write Verification

After writing ALL copy for ALL sections:

1. Scan every word of the complete output against the full banned list
2. For each violation found, replace the entire construction (not just the banned word)
3. Re-scan to verify zero violations remain

**Fix violations before output** — do NOT leave them for the Stage 3 editorial pass. Stage 3 should find zero banned vocabulary violations. Its job is to catch subtler issues like rhythm, specificity gaps, and voice drift.

---

## Typographic Rhythm Rules

### Statement/Proof Alternation

All body copy alternates between two modes:

```
MODE 1: SHORT STATEMENT (punch)     — 3-8 words
MODE 2: DESCRIPTIVE PROOF (support) — 10-20 words

Pattern: Statement → Proof → Statement → Proof

EXAMPLE:
"Ship code, not configs."                   ← statement (5 words)
Push to git. Builds deploy in 90            ← proof (14 words)
seconds to 35 edge regions.

"Scale without thinking."                   ← statement (3 words)
Auto-scaling handles 0 to 100K             ← proof (12 words)
req/s. Zero configuration required.
```

### Validation Checklist

After writing all copy, verify:

- [ ] No paragraph exceeds 3 sentences
- [ ] Statement/proof alternation is present in body copy sections
- [ ] Headlines are 4-8 words
- [ ] Subheadlines are 1-2 sentences
- [ ] Average sentence length is under 15 words
- [ ] 90%+ of sentences use active voice
- [ ] No sentence has more than 2 adjectives

---

## Output Format

Write the completed wireframe copy to `brand/wireframe-copy.md`. Use this template:

```markdown
# Wireframe Copy: [Product Name]

**Date**: [date]
**Brand Packet**: brand/brand-packet.md
**IA Document**: brand/information-architecture.md
**Voice**: [3 vibe adjectives from Field 5]

---

## Copy Summary Metrics

| Metric | Value |
|--------|-------|
| Total sections | [N] |
| Total content slots filled | [N] |
| Total word count | [N] |
| Average sentence length | [N] words |
| Active voice percentage | [N]% |
| Banned vocabulary violations | 0 |
| Placeholders remaining | [N] |
| Headline pattern distribution | [list patterns used] |

---

## Section Copy

### Section 1: [Name] ([Archetype])

**Layout Pattern**: [from IA]

| Slot | Type | Copy |
|------|------|------|
| headline | headline | [exact copy] |
| subheadline | subheadline | [exact copy] |
| cta-primary | cta-primary | [exact copy] |
| ... | ... | ... |

[Repeat for each section]

---

## Placeholders

| # | Section | Slot | Placeholder | Suggested Source |
|---|---------|------|-------------|-----------------|
| 1 | [section] | [slot] | [description] | [where to find data] |

---

## Voice Verification Checklist

- [ ] All 3 vibe adjectives observable in copy (tested per-section)
- [ ] Zero anti-vibe violations (no sentence crosses a boundary)
- [ ] Zero banned vocabulary from default list (8 words)
- [ ] Zero banned vocabulary from product-specific list
- [ ] Zero generic adjectives (powerful, innovative, comprehensive, etc.)
- [ ] Every claim has adjacent proof (number, name, or link)
- [ ] Average sentence length under 15 words
- [ ] No paragraph exceeds 3 sentences
- [ ] 90%+ active voice
- [ ] Statement/proof rhythm present in body sections
- [ ] Headlines match approved patterns (Outcome, Contrast, Technical Truth, Mechanism, Metric)
- [ ] Subheadlines match approved patterns (Hyper-specific, Negation, Amplification, Mechanism detail)
- [ ] CTA copy uses action verbs, not generic "Get Started"
- [ ] Copy passes the Grayscale Test (works without design)
```

---

## Instructions

When writing wireframe copy:

1. **Read both inputs** — read `brand/information-architecture.md` and `brand/brand-packet.md` completely. If either is missing, stop and report.
2. **Extract voice constraints** — pull the 3 vibe adjectives (Field 5), 3 anti-vibes (Field 6), and full banned vocabulary list (Field 7 + generic adjectives). Hold these in working memory.
3. **Extract trust signal catalog** — pull all metrics, quotes, logos, badges, and community signals from Field 8. These are the raw materials for proof copy.
4. **Write copy section-by-section** — follow the IA document's section order. For each section:
   a. Read the archetype, content slots, and constraints from the IA
   b. Select the appropriate headline and subheadline patterns
   c. Derive slot copy from the Brand Packet fields specified in Section-Type Copy Patterns
   d. Apply the voice test to every sentence (all 3 vibes, no anti-vibes)
   e. Check rhythm: statement/proof alternation in body copy
5. **Use placeholders** for any data not in the Brand Packet — follow the placeholder protocol exactly
6. **Run banned vocabulary scan** — check every word against the full ban list. Fix ALL violations before proceeding.
7. **Compute summary metrics** — count total slots, words, average sentence length, active voice percentage, placeholder count
8. **Write output** to `brand/wireframe-copy.md` using the template above

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Synonym substitution** | Replacing "seamless" with "smooth" or "effortless" | Replace the entire construction with a specific outcome, not a synonym |
| **Metric fabrication** | Inventing numbers not in the Brand Packet | Use `[PLACEHOLDER]` format — never fabricate metrics |
| **IA modification** | Adding sections, changing archetypes, or reordering during copy phase | The IA is immutable. If the structure seems wrong, flag it — don't modify |
| **Headline-as-sentence** | Writing headlines as complete sentences with periods and clauses | Headlines are 4-8 word fragments. "Deploy to 35 regions." not "You can deploy your code to 35 regions around the world." |
| **Abstract card descriptions** | Feature cards that describe categories instead of capabilities | "Security features" → "SOC 2 Type II. Encrypted at rest and in transit. Audit logs for every action." |
| **Testimonial invention** | Writing quotes that don't exist in Field 8 | Only use quotes from the Brand Packet trust catalog. Use `[PLACEHOLDER]` if needed |
| **Vibe abandonment** | Writing "normal" copy and ignoring the 3 adjective constraint | Test every sentence against all 3 vibe adjectives. Rewrite until all 3 are present |
| **Rhythm collapse** | Body copy is uniform-length sentences with no punch/proof alternation | Alternate: 3-8 word statements followed by 10-20 word proof sentences |
| **Orphan claims** | Making a claim with no adjacent proof within the same section | Every claim needs a number, name, or link within 1-2 sentences |
| **CTA vagueness** | "Get Started", "Learn More", "Sign Up" | Use action verb + outcome: "Start building — free", "See it in 2 minutes" |
| **Wrong-ICP copy** | Writing copy that speaks to the Anti-ICP instead of the ICP | Re-read Field 2 (ICP) and Field 3 (Anti-ICP). Verify language matches ICP vocabulary and concerns |

---

## Edge Cases

### Conditional IA Sections
- Some IA sections may be marked as conditional (e.g., "include if community signals are strong")
- Write copy for conditional sections if the Brand Packet has supporting data
- If data is insufficient, write placeholder copy and flag in the Placeholders Table

### Competing Differentiators
- When Field 4 has 5 differentiators but the Value Grid only has 3 cards, prioritize by ICP relevance
- Less relevant differentiators can be mentioned in body copy of other sections
- Note any deprioritized differentiators in the copy summary

### Sentiment-Only Quotes
- If Field 8 quotes express sentiment but lack metrics ("We love this product"), frame them alongside a metric from elsewhere in Field 8
- Pairing: metric from catalog + sentiment quote = stronger than either alone
- Do NOT rewrite the quote — use it verbatim but provide metric context around it

### Short Pages (4-5 Sections)
- Every word carries more weight on a short page
- Headlines may need to do double duty (outcome + mechanism in one)
- Subheadlines should maximize proof density (numbers, constraints, names)
- Omit section-subheadlines if they add length without value

### Highly Technical ICP
- Use technical vocabulary that the ICP uses daily — don't simplify
- Code examples in mechanism sections can serve as "copy" (annotated code blocks)
- Metrics should use technical units (p99 latency, req/s, TTFB) not business units
- Acronyms are acceptable if the ICP uses them (CI/CD, RBAC, SSO)

---

## Related Skills and Agents

- **`writing-information-architecture` skill**: Produces the IA document that this skill fills with copy
- **`reviewing-copy` skill**: Reviews the wireframe copy output for banned vocabulary, generic language, and rhythm violations (Stage 3 editorial pass)
- **`brand_packet_producer` agent**: Produces the Brand Packet that governs all copy decisions
- **`writing-brand-voice` skill**: Defines the voice constraints (Fields 5-7) this skill enforces
- **`writing-brand-positioning` skill**: Positioning (Field 1) provides the raw material for hero copy
- **`writing-trust-signals` skill**: Trust signals (Field 8) provide the proof data for copy

**Pipeline position**: This skill is **Stage 2** in the marketing production pipeline. It runs after the IA document is produced (Stage 1) and before the editorial review (Stage 3). The wireframe copy document is the deliverable that Stage 3 reviews and polishes.
