---
name: validating-brand-packets
version: 1.0.0
user-invocable: false
description: Validate Brand Packets for completeness and quality using an 11-field checklist and 6-dimension Polish Evaluation Rubric. Use when reviewing completed Brand Packets, running quality gates before page production, or scoring marketing readiness. Triggered by: brand packet validation, quality gate, polish rubric, completeness check, brand packet review, marketing readiness, visual hierarchy score, trust density score, copy specificity, validation report.
allowed-tools: Read, Glob, Grep
---

# Brand Packet Validation

## Purpose

Quality gate for completed Brand Packets. This skill validates that all 11 fields are present, internally consistent, and meet minimum quality standards. It also applies the 6-dimension Polish Evaluation Rubric to score the packet's readiness for downstream page production.

No Brand Packet should enter the page production pipeline without passing this validation. A weak Brand Packet produces weak pages — garbage in, garbage out.

---

## 11-Field Completeness Checklist

Every Brand Packet must contain all 11 fields. Each field has a source skill that produced it:

```
┌─────────────────────────────────────────────────────────────────┐
│                    BRAND PACKET FIELDS                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STRATEGIC FOUNDATION (from brand-positioning):                  │
│  ┌─────────────────────────────────────────────────────┐        │
│  │ Field 1:  Positioning Statement                      │        │
│  │ Field 2:  Ideal Customer Profile (ICP)               │        │
│  │ Field 3:  Anti-ICP                                   │        │
│  │ Field 4:  Differentiators (3-5 with proof)           │        │
│  └─────────────────────────────────────────────────────┘        │
│                                                                  │
│  VOICE & TONE (from brand-voice):                                │
│  ┌─────────────────────────────────────────────────────┐        │
│  │ Field 5:  Vibe (3 adjectives)                        │        │
│  │ Field 6:  Anti-Vibe (3 failure modes)                │        │
│  │ Field 7:  Banned Vocabulary                          │        │
│  └─────────────────────────────────────────────────────┘        │
│                                                                  │
│  TRUST (from trust-signals):                                     │
│  ┌─────────────────────────────────────────────────────┐        │
│  │ Field 8:  Trust Signals (catalog + placement)        │        │
│  └─────────────────────────────────────────────────────┘        │
│                                                                  │
│  VISUAL SYSTEM (from visual-identity):                           │
│  ┌─────────────────────────────────────────────────────┐        │
│  │ Field 9:  Color Tokens (6 tokens)                    │        │
│  │ Field 10: Typography System (fonts + scale)          │        │
│  │ Field 11: Spatial Rules (8-point scale + macro)      │        │
│  └─────────────────────────────────────────────────────┘        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Field-Level Validation

| Field | Source Skill | Present? | Minimum Criteria |
|-------|-------------|----------|-----------------|
| **1. Positioning** | `writing-brand-positioning` | [ ] | All 5 slots filled (product, category, audience, outcome, mechanism) |
| **2. ICP** | `writing-brand-positioning` | [ ] | Role + company type + pain point specified |
| **3. Anti-ICP** | `writing-brand-positioning` | [ ] | At least 2 excluded personas with reasons |
| **4. Differentiators** | `writing-brand-positioning` | [ ] | 3-5 differentiators, each with proof from taxonomy |
| **5. Vibe** | `writing-brand-voice` | [ ] | Exactly 3 adjectives, each specific/observable/differentiating |
| **6. Anti-Vibe** | `writing-brand-voice` | [ ] | 3 failure modes, one per vibe adjective |
| **7. Banned Vocabulary** | `writing-brand-voice` | [ ] | 8 default banned words + product-specific additions |
| **8. Trust Signals** | `writing-trust-signals` | [ ] | 3+ metrics, 6+ logos, 2+ named quotes, 1+ badge |
| **9. Color Tokens** | `writing-visual-identity` | [ ] | 6 tokens with hex + OKLCH, contrast check passes |
| **10. Typography** | `writing-visual-identity` | [ ] | Font pair + scale + weights + tracking + line heights |
| **11. Spatial Rules** | `writing-visual-identity` | [ ] | 8-point scale values + macro-whitespace rules |

### Completeness Scoring

```
COMPLETENESS CALCULATION:

Fields present and meeting minimum criteria: [count] / 11

11/11 = COMPLETE    — Ready for Polish Evaluation
9-10/11 = PARTIAL   — Identify and fill gaps before proceeding
< 9/11  = INCOMPLETE — Return to source skills for missing fields
```

---

## 6-Dimension Polish Evaluation Rubric

After completeness is confirmed, score the Brand Packet across 6 dimensions. Each dimension is scored 1-5.

### Dimension 1: Visual Hierarchy

Does the visual system create clear, scannable hierarchy without relying on color alone?

| Score | Criteria |
|-------|----------|
| **5** | Passes grayscale test completely. 3+ distinct heading levels via weight + size. Macro-whitespace ≥ 96px between sections. Weight contrast delta ≥ 300. CTA visually dominant without color |
| **4** | Passes grayscale test with minor issues. 2-3 heading levels clear. Macro-whitespace present. Weight delta ≥ 200. CTA distinguishable |
| **3** | Partial grayscale pass — hierarchy mostly readable. Heading levels exist but inconsistent. Some macro-whitespace. Weight delta = 200 |
| **2** | Grayscale test fails — hierarchy depends on color. Heading levels indistinct. Minimal whitespace. Weight delta < 200 |
| **1** | No discernible hierarchy in grayscale. All text appears same weight/size. No macro-whitespace. Visually flat |

### Dimension 2: Typography System

Is the type system tight, consistent, and premium-feeling?

| Score | Criteria |
|-------|----------|
| **5** | Display tracking ≤ -0.02em. All line heights on 4px grid. Variable font or well-chosen pair. 5+ scale levels defined. Mono font for code contexts. Weight hierarchy: 400 body / 600-800 display |
| **4** | Display tracking tightened. Most line heights on grid. Good font pair. 4+ scale levels. Weight hierarchy present |
| **3** | Some tracking tightening. Line heights mostly consistent. Acceptable font choice. 3+ scale levels. Weight contrast adequate |
| **2** | Default tracking throughout. Inconsistent line heights. Generic or mismatched fonts. Few scale levels. Weak weight contrast |
| **1** | No tracking consideration. Random line heights. System defaults or poor choices. No defined scale. No weight hierarchy |

### Dimension 3: Spacing & Rhythm

Does the spatial system create consistent rhythm and breathing room?

| Score | Criteria |
|-------|----------|
| **5** | All spacing from 8-point scale. Section gaps ≥ 96px. Grid with defined max-width + gaps. Internal padding consistent. Responsive spacing rules defined. Zero arbitrary values |
| **4** | Mostly 8-point values. Section gaps ≥ 80px. Grid defined. Padding mostly consistent. Some responsive rules |
| **3** | Mix of 8-point and arbitrary values. Section gaps ≥ 64px. Grid partially defined. Some inconsistency in padding |
| **2** | Mostly arbitrary spacing. Section gaps < 64px. No grid definition. Inconsistent padding. Cramped feel |
| **1** | No spacing system. No whitespace strategy. No grid. Ad-hoc spacing throughout |

### Dimension 4: Mobile Responsiveness

Does the visual system accommodate mobile viewports?

| Score | Criteria |
|-------|----------|
| **5** | Fluid typography with clamp(). Touch targets ≥ 44px. Responsive spacing scale. Breakpoints defined (mobile/tablet/desktop). Single-column mobile layout specified. Hero adapts gracefully |
| **4** | Some fluid typography. Touch targets addressed. Responsive considerations noted. Breakpoints defined. Mobile layout considered |
| **3** | Fixed type scale with mobile overrides. Touch targets mostly adequate. Some responsive notes. Basic breakpoints |
| **2** | Desktop-only type scale. Touch targets not considered. Minimal responsive guidance. Vague breakpoints |
| **1** | No mobile consideration. No responsive rules. No touch target guidance. Desktop-only specification |

### Dimension 5: Copy Specificity

Does the positioning and voice produce specific, metric-backed, ban-free copy?

| Score | Criteria |
|-------|----------|
| **5** | Zero banned words. All claims metric-backed. Positioning uses all 5 slots with specific terms. ICP is pictureable. Every differentiator has typed proof. Voice adjectives are specific + observable |
| **4** | Zero banned words. Most claims have proof. Positioning complete. ICP specific. Most differentiators proven. Voice adjectives clear |
| **3** | 1-2 borderline words. Some claims lack proof. Positioning has weak slots. ICP somewhat vague. Some differentiators unproven |
| **2** | Multiple banned words. Claims mostly unproven. Positioning has 2+ weak slots. ICP too broad. Differentiators are feature lists |
| **1** | Banned vocabulary throughout. No metric backing. Positioning generic. ICP is "everyone". No real differentiators |

### Dimension 6: Trust Density

Are trust signals sufficient, specific, and strategically placed?

| Score | Criteria |
|-------|----------|
| **5** | 3+ metrics (hyper-specific), 8+ recognizable logos, 2+ named quotes with metrics, compliance badges, community signals, placement strategy for all 5 zones. All pass Provability Test |
| **4** | 2+ metrics, 6+ logos, named quotes, 1+ badge. Placement strategy defined. Signals pass Provability Test |
| **3** | 1 metric, 4+ logos, quotes present, no badges. Partial placement. Some signals need verification |
| **2** | Logo bar only. No quantified metrics. Anonymous quotes. No placement strategy |
| **1** | No trust signals defined. No metrics, no logos, no quotes |

---

## Validation Report Template

```markdown
# Brand Packet Validation Report

**Product**: [name]
**Date**: [date]
**Validator**: [skill/agent name]

## Completeness Check

| Field | Status | Notes |
|-------|--------|-------|
| 1. Positioning Statement | ✓/✗ | [detail] |
| 2. ICP | ✓/✗ | [detail] |
| 3. Anti-ICP | ✓/✗ | [detail] |
| 4. Differentiators | ✓/✗ | [detail] |
| 5. Vibe | ✓/✗ | [detail] |
| 6. Anti-Vibe | ✓/✗ | [detail] |
| 7. Banned Vocabulary | ✓/✗ | [detail] |
| 8. Trust Signals | ✓/✗ | [detail] |
| 9. Color Tokens | ✓/✗ | [detail] |
| 10. Typography | ✓/✗ | [detail] |
| 11. Spatial Rules | ✓/✗ | [detail] |

**Completeness Score**: [count]/11

## Polish Evaluation Rubric

| Dimension | Score | Key Finding |
|-----------|-------|-------------|
| 1. Visual Hierarchy | /5 | [one-line finding] |
| 2. Typography System | /5 | [one-line finding] |
| 3. Spacing & Rhythm | /5 | [one-line finding] |
| 4. Mobile Responsiveness | /5 | [one-line finding] |
| 5. Copy Specificity | /5 | [one-line finding] |
| 6. Trust Density | /5 | [one-line finding] |

**Total Score**: [sum]/30
**Minimum Passing**: 24/30 (4 on ALL dimensions)

## Verdict

[ ] **PASS** — Brand Packet ready for page production
[ ] **CONDITIONAL PASS** — Minor issues to fix (listed below)
[ ] **FAIL** — Significant issues require rework

## Issues Found

| Issue | Dimension | Severity | Fix |
|-------|-----------|----------|-----|
| [description] | [1-6] | Critical/Major/Minor | [recommendation] |

## Recommendations

1. [Priority 1]
2. [Priority 2]
3. [Priority 3]
```

---

## Minimum Passing Criteria

```
┌─────────────────────────────────────────────────────────────────┐
│                    PASSING CRITERIA                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  COMPLETENESS: 11/11 fields present                              │
│                                                                  │
│  POLISH RUBRIC: Score 4 or higher on ALL 6 dimensions            │
│                                                                  │
│  ┌─────────────────────────────────────────┐                    │
│  │  Dimension          │ Min Score │ Notes  │                    │
│  │  ──────────────────┼───────────┼────────│                    │
│  │  Visual Hierarchy   │    4      │        │                    │
│  │  Typography System  │    4      │        │                    │
│  │  Spacing & Rhythm   │    4      │        │                    │
│  │  Mobile Response    │    4      │        │                    │
│  │  Copy Specificity   │    4      │        │                    │
│  │  Trust Density      │    4      │        │                    │
│  │  ──────────────────┼───────────┼────────│                    │
│  │  TOTAL MINIMUM      │   24/30   │        │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
│  A single dimension at 3 or below = FAIL                         │
│  Even if total is ≥ 24, any dimension below 4 fails.             │
│                                                                  │
│  CONDITIONAL PASS:                                               │
│  • All dimensions ≥ 4 but 1-2 minor issues flagged              │
│  • Issues must be fixable without reworking source skills        │
│  • List specific fixes required                                  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Common Failure Patterns

### Pattern 1: "Pretty But Empty"

```
Affected Dimensions: Copy Specificity (2), Trust Density (2)
Passing Dimensions:  Visual Hierarchy (5), Typography (5), Spacing (4)

Symptom: Beautiful visual system with no substance behind it.
         Color tokens are perfect. Typography is refined.
         But positioning is generic and trust signals are absent.

Fix: Return to writing-brand-positioning and
     writing-trust-signals. Visual system is fine —
     the strategic foundation needs work.
```

### Pattern 2: "Strong Words, Weak Design"

```
Affected Dimensions: Visual Hierarchy (2), Typography (2), Spacing (2)
Passing Dimensions:  Copy Specificity (5), Trust Density (4)

Symptom: Excellent positioning, strong trust signals, but the
         visual system is an afterthought. Default fonts, no
         spacing system, no hierarchy beyond text size.

Fix: Return to writing-visual-identity. The strategy
     is solid — the visual system needs to match its quality.
```

### Pattern 3: "Desktop-Only"

```
Affected Dimensions: Mobile Responsiveness (2)
Passing Dimensions:  All others at 4+

Symptom: Everything works on desktop but no mobile rules defined.
         No fluid typography, no touch targets, no responsive spacing.

Fix: Add clamp() rules to typography, define responsive breakpoints,
     specify touch target minimums. Usually fixable without rework.
```

### Pattern 4: "Generic Voice"

```
Affected Dimensions: Copy Specificity (2-3)
Passing Dimensions:  Most others at 4+

Symptom: Vibe adjectives are "professional, reliable, modern."
         Banned vocabulary exists but positioning statement still
         contains weasel words. Anti-ICP is vague.

Fix: Return to writing-brand-voice and
     writing-brand-positioning. Apply the differentiation
     test: would a competitor claim the same adjectives?
```

### Pattern 5: "Trust Vacuum"

```
Affected Dimensions: Trust Density (1-2)
Passing Dimensions:  Most others at 4+

Symptom: No trust signals sourced. "We'll add logos later."
         Metrics are round numbers or absent. No named quotes.

Fix: This is a business problem, not a writing problem.
     Create signal acquisition plan with timeline.
     Use placeholders in Brand Packet but flag as INCOMPLETE.
```

---

## Instructions

When validating a Brand Packet:

1. **Run the 11-field completeness check** — mark each field as present/absent with notes
2. **If incomplete**, return the packet to the source skill for the missing field(s)
3. **If complete**, score each of the 6 Polish Evaluation dimensions (1-5)
4. **Check for cross-field consistency**:
   - Do differentiators (Field 4) have matching trust signals (Field 8)?
   - Does voice (Fields 5-7) align with the ICP (Field 2)?
   - Does the visual system (Fields 9-11) match the voice archetype?
5. **Produce the Validation Report** using the template above
6. **Issue verdict**: PASS, CONDITIONAL PASS, or FAIL
7. **For FAIL or CONDITIONAL PASS**, list specific issues and which upstream skill should fix them

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Rubber-stamping** | Passing a packet without scoring each dimension | Always produce the full rubric with scores |
| **Compensating scores** | "Trust is 2 but visual is 5, so it averages out" | Each dimension must independently reach 4 |
| **Ignoring consistency** | Fields are individually fine but contradict each other | Check cross-field alignment explicitly |
| **Deferring trust** | "We'll add trust signals after launch" | Trust is not optional — flag as FAIL, not CONDITIONAL |
| **Subjective scoring** | "Feels like a 4" without referencing criteria | Always cite specific criteria from the scoring table |

---

## Edge Cases

### Pre-Launch Packets
- Trust Density minimum drops to 2 (expected — limited signals available)
- Flag trust gaps with acquisition timeline
- All other dimensions maintain 4 minimum
- Packet receives CONDITIONAL PASS with trust acquisition plan

### Incremental Validation
- Validate after each writing skill completes (not only at the end)
- Run completeness check after each field is added
- Run Polish Evaluation only when all 11 fields are present

### Multiple ICPs
- Validate primary ICP meets all criteria
- Secondary ICPs are acceptable but must not dilute the primary
- Anti-ICP must still clearly exclude non-target personas

---

## Related Skills and Agents

- **`writing-brand-positioning` skill**: Source for Fields 1-4
- **`writing-brand-voice` skill**: Source for Fields 5-7
- **`writing-trust-signals` skill**: Source for Field 8
- **`writing-visual-identity` skill**: Source for Fields 9-11
- **`analyzing-competitors` skill**: Provides baseline for competitive scoring
- **`reviewing-copy` skill**: Post-validation editorial pass on generated copy

**When to use which**: Use this skill as the quality gate between Brand Packet production and page production. Run after all writing skills have produced their fields. If validation fails, return to the specific upstream skill indicated in the failure report.
