---
name: writing-visual-identity
version: 1.0.0
user-invocable: false
description: Specify color tokens, typography system, and spatial rules for marketing pages. Use when defining visual identity, selecting color palettes, choosing typography pairings, establishing spacing systems, or creating design token specifications. Triggered by: visual identity, color tokens, typography system, font pairing, spacing system, design tokens, color palette, brand colors, type scale, spatial rules, 8-point grid, OKLCH, macro-whitespace, baseline grid.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Visual Identity Specification

## Purpose

Specify the visual system that implements the brand on marketing pages: color tokens, typography system, and spatial rules. These form Fields 9-11 of the Brand Packet.

Visual identity is not decoration — it's information architecture. Color creates hierarchy. Typography creates rhythm. Spacing creates breathing room. Together, they determine whether a page feels premium or generic before a single word is read.

This skill references `brand/tokens/primitives/` as a pattern for token structure, but outputs are independent — they define a brand's visual system from scratch.

---

## Color System

### 6 Required Color Tokens

Every marketing page needs exactly 6 foundational color tokens. Additional tokens can be derived, but these 6 must be explicitly defined:

```
┌─────────────────────────────────────────────────────────────────┐
│                    6 REQUIRED COLOR TOKENS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐        │
│  │  base-dark    │  │ accent-primary│  │surface-second.│        │
│  │               │  │               │  │               │        │
│  │  Page bg in   │  │  CTA buttons, │  │  Cards, code  │        │
│  │  dark mode,   │  │  links, key   │  │  blocks, alt  │        │
│  │  nav bars     │  │  highlights   │  │  backgrounds  │        │
│  │               │  │               │  │               │        │
│  │  e.g. #0A0A0B │  │  e.g. #6366F1 │  │  e.g. #18181B │        │
│  └───────────────┘  └───────────────┘  └───────────────┘        │
│                                                                  │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────┐        │
│  │ text-primary  │  │text-secondary │  │ border-subtle │        │
│  │               │  │               │  │               │        │
│  │  Headlines,   │  │  Body text,   │  │  Dividers,    │        │
│  │  key content  │  │  descriptions │  │  card borders │        │
│  │               │  │               │  │               │        │
│  │  e.g. #FAFAFA │  │  e.g. #A1A1AA │  │  e.g. #27272A │        │
│  └───────────────┘  └───────────────┘  └───────────────┘        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Token Definitions

| Token | Role | Selection Criteria |
|-------|------|--------------------|
| **base-dark** | Primary page background (dark mode), navigation backgrounds | Near-black with slight color cast. Avoid pure #000 (too harsh) |
| **accent-primary** | CTAs, links, interactive highlights, key UI elements | Must pass WCAG AA contrast on base-dark. Should be recognizable as "the brand color" |
| **surface-secondary** | Cards, code blocks, secondary backgrounds, hover states | 5-15% lighter than base-dark. Subtle distinction, not jarring |
| **text-primary** | Headlines, important labels, primary content | Near-white. Avoid pure #FFF (too harsh). Slight warmth optional |
| **text-secondary** | Body text, descriptions, meta information | 40-60% opacity of text-primary, or explicit mid-gray |
| **border-subtle** | Card borders, dividers, input outlines | Low-contrast against surface-secondary. Visible but not dominant |

### Deriving Colors from a Single Brand Color

```
┌─────────────────────────────────────────────────────────────────┐
│                    COLOR DERIVATION METHOD                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Start: Brand accent color (e.g., #6366F1 — indigo)             │
│                                                                  │
│  Step 1: Set accent-primary = brand color                        │
│                                                                  │
│  Step 2: Derive base-dark                                        │
│     Take accent hue, set lightness to 3-5%, chroma to 0.01-0.02 │
│     Result: Near-black with subtle color temperature              │
│                                                                  │
│  Step 3: Derive surface-secondary                                │
│     Take base-dark, increase lightness by 5-8%                   │
│     Result: Distinguishable card/code background                 │
│                                                                  │
│  Step 4: Set text-primary                                        │
│     Lightness 95-98%, minimal chroma                             │
│     Slightly warm (0.005 chroma) avoids clinical feel            │
│                                                                  │
│  Step 5: Set text-secondary                                      │
│     Lightness 55-65%, match hue family of text-primary           │
│     Must be readable on both base-dark and surface-secondary     │
│                                                                  │
│  Step 6: Set border-subtle                                       │
│     Lightness 15-20%, low chroma                                 │
│     Barely visible on base-dark, clear on surface-secondary      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### OKLCH Perceptual Uniformity

Prefer OKLCH color space for token definitions. OKLCH provides perceptually uniform lightness, meaning a lightness value of 0.5 looks equally bright across all hues.

```
OKLCH Format: oklch(lightness chroma hue)

Benefits:
- Lightness comparisons are meaningful (0.95 is always near-white)
- Chroma (saturation) is independent of hue
- Hue rotation produces equally vivid colors
- No unexpected muddy or neon tones

Example Token Set in OKLCH:
  base-dark:         oklch(0.13 0.02 270)   /* very dark, slight blue */
  accent-primary:    oklch(0.55 0.25 270)   /* vivid indigo */
  surface-secondary: oklch(0.18 0.02 270)   /* dark card background */
  text-primary:      oklch(0.97 0.005 90)   /* near-white, warm */
  text-secondary:    oklch(0.60 0.01 270)   /* mid-gray */
  border-subtle:     oklch(0.25 0.01 270)   /* subtle divider */
```

### Color Output Template

```
COLOR SYSTEM:

accent-primary:    [hex] / [oklch]  — CTA, links, highlights
base-dark:         [hex] / [oklch]  — page background
surface-secondary: [hex] / [oklch]  — cards, code blocks
text-primary:      [hex] / [oklch]  — headlines, key text
text-secondary:    [hex] / [oklch]  — body, descriptions
border-subtle:     [hex] / [oklch]  — dividers, borders

Derivation: [describe how tokens were derived from brand color]
Contrast Check: accent-primary on base-dark = [ratio]:1 (WCAG AA: 4.5:1)
```

---

## Typography System

### Font Selection Decision Tree

```
┌─────────────────────────────────────────────────────────────────┐
│                    FONT SELECTION DECISION TREE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Q1: Is this a developer-focused product?                        │
│  ├── YES → Strong monospace presence required                    │
│  │   Q2: Is the product premium/minimal?                         │
│  │   ├── YES → Geist + Geist Mono                                │
│  │   └── NO  → Inter + JetBrains Mono                            │
│  │                                                               │
│  └── NO → Monospace optional                                     │
│      Q3: Is the product enterprise/authority?                    │
│      ├── YES → Inter or system fonts (safe, universal)           │
│      └── NO                                                      │
│          Q4: Is the product design-forward/creative?             │
│          ├── YES → Plus Jakarta Sans or Satoshi                  │
│          └── NO → Inter (safe default for any SaaS)              │
│                                                                  │
│  UNIVERSAL FALLBACK: Inter is always acceptable.                 │
│  Never wrong, but also never distinctive.                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Recommended Font Pairings

| Pairing | Heading / Body | Mono | Personality | Used By |
|---------|---------------|------|-------------|---------|
| **Geist / Geist** | Geist Sans (display) / Geist Sans (body) | Geist Mono | Premium, minimal, modern | Vercel, Resend |
| **Inter / Inter** | Inter (tight tracking) / Inter (normal) | JetBrains Mono | Clean, universal, professional | Supabase, countless SaaS |
| **Plus Jakarta / Inter** | Plus Jakarta Sans (display) / Inter (body) | Fira Code | Friendly, modern, approachable | Newer SaaS products |
| **Satoshi / Inter** | Satoshi (display) / Inter (body) | IBM Plex Mono | Geometric, contemporary | Design-forward tools |
| **SF Pro / SF Pro** | SF Pro Display / SF Pro Text | SF Mono | Apple-adjacent, system-native | Apple ecosystem products |

### Weight Contrast Rules

```
┌─────────────────────────────────────────────────────────────────┐
│                    WEIGHT CONTRAST RULES                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Rule: Weight delta between heading and body ≥ 200               │
│                                                                  │
│  GOOD:                                                           │
│  Heading: 700 (Bold)     Body: 400 (Regular)    Delta: 300 ✓    │
│  Heading: 600 (Semibold) Body: 400 (Regular)    Delta: 200 ✓    │
│  Heading: 800 (Black)    Body: 400 (Regular)    Delta: 400 ✓    │
│                                                                  │
│  BAD:                                                            │
│  Heading: 500 (Medium)   Body: 400 (Regular)    Delta: 100 ✗    │
│  Heading: 600 (Semibold) Body: 500 (Medium)     Delta: 100 ✗    │
│                                                                  │
│  Weight hierarchy for marketing pages:                           │
│  ├── Display/Hero:   700-800 (Bold to Black)                    │
│  ├── Section title:  600-700 (Semibold to Bold)                 │
│  ├── Subheading:     500-600 (Medium to Semibold)               │
│  ├── Body:           400 (Regular)                               │
│  └── Caption/meta:   400 (Regular, smaller size + color change) │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Tracking (Letter-Spacing) Rules

| Text Type | Tracking | Rationale |
|-----------|---------|-----------|
| **Display** (48px+) | -0.02em to -0.03em | Large text needs tightening — loose tracking looks amateur |
| **Title** (24-48px) | -0.01em to -0.02em | Moderate tightening for visual density |
| **Body** (14-18px) | 0 (normal) | Default tracking optimized for readability |
| **Caption** (12-14px) | 0 to +0.01em | Small text may benefit from slight loosening |
| **Mono/Code** (14-16px) | 0 (normal) | Monospace fonts have built-in spacing |
| **All-caps labels** | +0.05em to +0.1em | All-caps requires significant loosening |

### Baseline Grid

```
4-POINT BASELINE GRID FOR LINE HEIGHTS:

All line heights must snap to multiples of 4px.

Text Size → Line Height (nearest 4px multiple)
─────────────────────────────────────────────
12px      → 16px  (1.33)
14px      → 20px  (1.43)
16px      → 24px  (1.50)
18px      → 28px  (1.56)
20px      → 28px  (1.40)
24px      → 32px  (1.33)
32px      → 40px  (1.25)
40px      → 48px  (1.20)
48px      → 56px  (1.17)
64px      → 72px  (1.13)

Pattern: As text size increases, line-height ratio decreases.
Display text: 1.1-1.2 | Body text: 1.4-1.6 | Small text: 1.3-1.5
```

### Typography Output Template

```
TYPOGRAPHY SYSTEM:

Display Font:  [family], [weight], [tracking]
Body Font:     [family], [weight], [tracking]
Mono Font:     [family], [weight] (if applicable)

Type Scale:
  Hero:        [size]px / [line-height]px, [weight], tracking [value]
  Section:     [size]px / [line-height]px, [weight], tracking [value]
  Subheading:  [size]px / [line-height]px, [weight], tracking [value]
  Body:        [size]px / [line-height]px, [weight], tracking [value]
  Caption:     [size]px / [line-height]px, [weight], tracking [value]

Baseline Grid: 4px
Weight Contrast: Heading [weight] / Body [weight] = delta [value]
```

---

## Spatial Rules

### 8-Point Linear Scale

```
┌─────────────────────────────────────────────────────────────────┐
│                    8-POINT SPACING SCALE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  4px   ■           micro (icon-label gaps)                       │
│  8px   ■■          tight (label-input, inline spacing)           │
│  12px  ■■■         compact (related element gaps)                │
│  16px  ■■■■        base (standard component padding)             │
│  20px  ■■■■■       comfortable (grid gaps, card padding)         │
│  24px  ■■■■■■      relaxed (form field spacing)                  │
│  32px  ■■■■■■■■    generous (section sub-spacing)                │
│  40px  ■■■■■■■■■■  wide (card internal padding)                  │
│  48px  ■■■■■■■■■■■■ spacious (component group spacing)          │
│  64px  ────────────  section (minor section dividers)            │
│  96px  ════════════  macro (between major sections)              │
│  128px ████████████  heroic (hero to first section)              │
│                                                                  │
│  Rule: Use ONLY values from this scale.                          │
│  Arbitrary values (e.g., 37px, 55px) signal ad-hoc design.      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Macro-Whitespace Rules

Macro-whitespace is the defining characteristic of premium marketing pages. Generic pages are dense; premium pages breathe.

| Zone | Spacing | Purpose |
|------|---------|---------|
| **Hero → First Section** | 96-128px | Hero needs dramatic separation from content |
| **Between Major Sections** | 96px (minimum) | Each section should feel like a distinct "room" |
| **Between Sub-Sections** | 48-64px | Related content groups within a section |
| **Within Components** | 32-48px | Internal padding of cards, panels |
| **Navigation Height** | 64-80px | Enough for brand mark + primary nav |
| **Footer Top Margin** | 96-128px | Footer separation matches hero separation |

### Grid System

```
GRID SPECIFICATION:

Max Content Width: 1200-1280px (standard), 1440px (wide)
Column Count:      12 (marketing pages)
Column Gap:        20-32px
Side Padding:      24px (mobile), 40-64px (tablet), 80px+ (desktop)

Responsive Breakpoints:
  Mobile:   < 640px   → 1 column, 24px padding
  Tablet:   640-1024px → 2 columns, 32px gap
  Desktop:  1024px+    → 12 columns, 24-32px gap
  Wide:     1440px+    → max-width container centers
```

### Micro-Spacing Rules

| Context | Spacing | Example |
|---------|---------|---------|
| **Icon ↔ Label** | 8px | Button icon to button text |
| **Label ↔ Input** | 4-8px | Form label to form field |
| **Button Padding** | 12px 24px (default), 16px 32px (large) | Horizontal: 2x vertical |
| **Badge/Tag Padding** | 4px 12px | Compact, pill-shaped |
| **Card Internal** | 24-40px | Consistent all sides |
| **List Item Gap** | 8-12px | Between consecutive list items |
| **Paragraph Gap** | 16-24px | Between consecutive paragraphs |
| **Heading ↔ Body** | 12-16px | Heading to its body text |
| **Section Title ↔ Content** | 24-32px | Section heading to first content |

---

## Grayscale-First Validation

### Test Procedure

```
┌─────────────────────────────────────────────────────────────────┐
│                    GRAYSCALE-FIRST VALIDATION                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 1: Remove all color                                        │
│     Convert the entire page to grayscale                         │
│     (CSS: filter: grayscale(1))                                  │
│                                                                  │
│  Step 2: Check visual hierarchy                                  │
│     Can you still identify:                                      │
│     - [ ] Primary heading vs secondary heading?                  │
│     - [ ] CTA button vs secondary button?                        │
│     - [ ] Primary content vs supporting content?                 │
│     - [ ] Card boundaries and groupings?                         │
│     - [ ] Navigation structure?                                  │
│                                                                  │
│  Step 3: Check spacing rhythm                                    │
│     Does the page still feel:                                    │
│     - [ ] Organized (not chaotic)?                               │
│     - [ ] Premium (macro-whitespace present)?                    │
│     - [ ] Scannable (clear sections)?                            │
│                                                                  │
│  Step 4: Verdict                                                 │
│     If hierarchy and rhythm survive grayscale →                  │
│     The visual system is structurally sound.                     │
│     Color enhances but doesn't create the hierarchy.             │
│                                                                  │
│     If hierarchy collapses in grayscale →                        │
│     Fix weight contrast, spacing, and sizing FIRST.              │
│     Then re-add color.                                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Instructions

When specifying visual identity:

1. **Identify the brand's primary color** — this may come from an existing logo, product UI, or positioning decision
2. **Derive the 6 color tokens** using the derivation method above, preferring OKLCH for precision
3. **Select font pairing** using the decision tree, considering product type and audience
4. **Define the type scale** with sizes, weights, tracking, and line heights snapped to the 4px baseline grid
5. **Specify the spacing system** using the 8-point scale, macro-whitespace rules, and micro-spacing guidelines
6. **Run the Grayscale-First Validation** to verify structural integrity
7. **Output Fields 9-11** of the Brand Packet:
   - Field 9: Color Tokens (6 tokens with hex and OKLCH values)
   - Field 10: Typography System (fonts, scale, weights, tracking, line heights)
   - Field 11: Spatial Rules (8-point scale, macro-whitespace, grid, micro-spacing)

---

## Quality Checklist

- [ ] All 6 color tokens defined with both hex and OKLCH values
- [ ] accent-primary passes WCAG AA contrast (4.5:1) on base-dark
- [ ] text-primary passes WCAG AA contrast on base-dark
- [ ] text-secondary passes WCAG AA contrast on base-dark (4.5:1 for body text)
- [ ] Font pairing specified with weight and tracking values
- [ ] Type scale includes 5+ levels (hero, section, subhead, body, caption)
- [ ] All line heights snap to 4px baseline grid
- [ ] Weight contrast delta ≥ 200 between heading and body
- [ ] Spacing uses only 8-point scale values (no arbitrary numbers)
- [ ] Grayscale-first validation passes

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Pure black background** | #000000 is too harsh, causes eye strain | Use near-black with slight color cast (lightness 0.08-0.15) |
| **Accent on accent** | Using accent-primary for large surfaces | Accent is for small highlights, not backgrounds |
| **Weight uniformity** | All text at same weight | Enforce delta ≥ 200 between heading and body |
| **Loose display tracking** | Large headlines with default letter-spacing | Tighten display text to -0.02em minimum |
| **Arbitrary spacing** | Padding values like 37px, 55px, 100px | Use only 8-point scale values |
| **Missing macro-whitespace** | Sections crammed together | Minimum 96px between major sections |
| **Too many fonts** | 4+ font families loaded | Maximum 2 families + 1 monospace |
| **Color-dependent hierarchy** | Hierarchy collapses in grayscale | Fix weight, size, and spacing first — color enhances |

---

## Edge Cases

### Existing Brand Guidelines
- Extract existing colors and map to the 6-token system
- If the brand has more tokens, map to the 6 required + note extensions
- If the brand has fewer, derive missing tokens from what exists

### Light Mode Marketing Pages
- Invert the token roles: base becomes near-white, text becomes near-black
- accent-primary must still pass contrast on the light background
- Many premium SaaS sites use dark mode for marketing, light for docs/app

### Variable Fonts
- Prefer variable fonts when available (smaller file, more control)
- Define weight axis range (e.g., 400-700) rather than loading discrete weights
- Enables smooth weight transitions for hover states

### Responsive Typography
- Use CSS `clamp()` for fluid type scaling between breakpoints
- Example: `font-size: clamp(2rem, 5vw, 4rem)` for hero text
- Maintain baseline grid alignment at all viewport sizes

---

## Related Skills and Agents

- **`analyzing-competitors` skill**: Design system inference from competitors informs visual direction
- **`writing-brand-positioning` skill**: Positioning informs whether visual identity should feel premium, playful, authoritative, etc.
- **`writing-brand-voice` skill**: Voice archetype influences typography and color choices (e.g., "Technical Founder" → tight tracking, dark mode)
- **`validating-brand-packets` skill**: Validates visual identity tokens for completeness and quality
- **`auditing-page-velocity` skill**: Font choices impact page performance (web font loading)
- **`design-engineer` agent**: For implementing visual identity in actual components

**When to use which**: Use this skill to define the visual system specification (Fields 9-11). Use `design-engineer` to implement it in components. Use `auditing-page-velocity` to validate that font choices don't degrade performance.
