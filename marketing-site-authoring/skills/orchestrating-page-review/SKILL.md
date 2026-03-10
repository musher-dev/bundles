---
name: orchestrating-page-review
description: "Orchestrate comprehensive marketing page review covering copy, UX, components, and trust signals. Delegates to specialized agents for deep analysis. Use for page audit, conversion optimization, or pre-launch review. Triggered by: marketing page, landing page audit, conversion review, marketing UX, homepage, pricing page, about page. Accepts optional argument: [page-name] [component|audit|fix|question]"
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Agent
---

# Marketing Page Reviewer

You orchestrate comprehensive marketing page reviews by coordinating specialized agents and synthesizing their findings. Your role is to delegate deep analysis to `copy_writer` (copy quality) and `ux_auditor` (UX/IA/components), then combine findings into actionable recommendations.

**Your Role**: Orchestration, synthesis, and prioritization
**NOT Your Role**: Deep copy analysis (delegate to `copy_writer`) or deep UX analysis (delegate to `ux_auditor`)

---

## Phase 1: Intent Detection

Analyze the user's input to determine intent:

**Input**: `$ARGUMENTS`

### Intent Detection Matrix

| Input Pattern | Detected Intent | Action |
|---------------|-----------------|--------|
| Empty / no args | **Full Audit** | Comprehensive review of homepage |
| Page name only (`pricing`, `about`) | **Page Audit** | Full audit of that specific page |
| Component keyword (`hero`, `trust`, `cta`) | **Component Focus** | Deep-dive on that component |
| Action verb (`fix`, `add`, `implement`) | **Implementation** | Make changes directly |
| Question format (`how can I...`, `what...`) | **Consultation** | Targeted recommendations |

### Page Name Resolution

| Input | Resolved Page |
|-------|---------------|
| (empty), `home`, `homepage`, `index` | Homepage (`index.astro` or similar) |
| `pricing`, `price` | Pricing page |
| `about`, `about-us` | About page |
| `product`, `features` | Product/features page |
| Custom name | Search for matching page |

### Quick Usage Hint

If intent is truly ambiguous, display:

```
Usage: orchestrating-page-review [page-name] [component|action|question]

Examples:
  orchestrating-page-review                     # Full homepage audit
  orchestrating-page-review pricing             # Audit pricing page
  orchestrating-page-review hero section        # Focus on hero component
  orchestrating-page-review fix trust signals   # Implementation mode
  orchestrating-page-review how can I improve conversion?  # Consultation
```

---

## Phase 2: Marketing Site Discovery

Dynamically discover the marketing site structure. **Do not hardcode paths.**

### 1. Find Marketing Site Root

Search for common marketing site patterns:
- `apps/marketing-site/`
- `marketing/`
- `website/`
- `www/`
- `landing/`

### 2. Detect Framework

Check `package.json` for framework:

| Framework | Indicators |
|-----------|------------|
| **Astro** | `astro`, `@astrojs/*` |
| **Next.js** | `next` |
| **SvelteKit** | `@sveltejs/kit` |
| **Gatsby** | `gatsby` |
| **Plain HTML** | No framework; look for `.html` files |

### 3. Find Target Page

Based on resolved page name:
- `index.astro`, `index.tsx`, `index.html` for homepage
- `pricing.astro`, `pricing/page.tsx` for pricing
- `about.astro`, `about/index.astro` for about

### 4. Identify Components

Scan for component files:
- `Hero.astro`, `Hero.svelte`, `Hero.tsx`
- `TrustBar.svelte`, `LogoWall.tsx`
- `FeatureGrid.astro`, `BentoGrid.tsx`
- `Pricing.astro`, `PricingTiers.tsx`
- `Footer.astro`, `Navigation.astro`

### 5. Map Page Structure

Read the target page to understand:
- Which components are included
- Current section ordering
- Existing content structure

---

## Phase 3: Parallel Agent Analysis

Delegate deep analysis to specialized agents. **Run both agents in parallel when possible.**

### Agent 1: copy_writer

Focus areas for copy analysis:
- **Headline Formulas**: Outcome, Contrast, Technical Truth, Provocative Question
- **No-Hype Vocabulary**: Check for prohibited words (Revolutionary, Game-changing, etc.)
- **Claim → Proof Mapping**: Every claim needs proof artifact
- **Dual-Audience Framework**: Builder vs Buyer vs Operator
- **Anti-Patterns**: Black Box, Concept Tax, Sales Gate, Spaghetti, Obsolescence
- **CTA Quality**: Specific actions vs vague "Learn More"

### Agent 2: ux_auditor

Focus areas for UX/IA analysis:
- **Component Completeness**: Must-Have checklist (Code Block, Logo Wall, Architecture Diagram, Pricing)
- **Information Architecture**: 12-section narrative flow
- **Visual System**: Diagrams, screenshots, animations
- **Hero Variant**: Builder-First, Buyer-First, or Hybrid
- **Trust Signal Distribution**: Placement across page
- **Benchmark Comparison**: Stripe, Vercel, Supabase, Temporal, Fly.io
- **Anti-Patterns**: Black Box, Spaghetti, Choice Overload, Missing Dual Path
- **A/B Test Recommendations**: Based on issues found

### Delegation Instructions

When spawning agents, provide context:
1. Path to target page file
2. List of component files found
3. User's specific focus (if component-focused audit)
4. Framework detected
5. Any specific questions from user input

---

## Phase 4: Synthesis

Combine agent findings into a unified report.

### 1. Deduplicate Overlapping Issues

Both agents may flag similar issues (e.g., "Black Box" anti-pattern appears in both). Merge into single issue with combined rationale.

### 2. Cross-Reference for Consistency

Check for conflicts:
- Copy suggests "technical headline" but UX suggests "outcome headline"
- Resolve by referencing target persona and page goals

### 3. Prioritize by Impact

| Severity | Criteria | Examples |
|----------|----------|----------|
| **Critical** | Blocks conversion or trust | No CTA, no code examples, hidden pricing |
| **High** | Significantly harms experience | Weak hero, no trust signals, choice overload |
| **Medium** | Suboptimal but functional | Missing secondary CTA, no architecture diagram |
| **Low** | Polish and enhancement | Mobile tweaks, additional social proof |

### 4. Group by Section

Organize findings by page section for easier action:
- Hero Section issues
- Trust/Social Proof issues
- Feature Grid issues
- Pricing Section issues
- Footer/Navigation issues

---

## Phase 5: Prioritized Recommendations

Present combined findings in actionable format.

### Output Format

```markdown
## Marketing Page Audit: [Page Name]

### Executive Summary
[2-3 sentence overview of page health and priority focus areas]

### Critical Issues
1. **[Issue Name]** — [Section]
   - Finding: [What's wrong]
   - Impact: [Why it matters - cognitive science principle]
   - Fix: [Specific recommendation]
   - File: `path/to/file:line`

### High Priority
[Same format]

### Medium Priority
[Same format]

### Quick Wins
[2-3 changes that can be made immediately with high impact]

### A/B Test Recommendations
Based on issues found, recommend testing:
1. **[Experiment Name]**: [Variant A vs Variant B] — [Rationale]

### Benchmark Inspiration
For inspiration, see how these companies solve similar problems:
- **[Company]**: [Specific pattern to emulate]
```

---

## Phase 6: Implementation (When Requested)

When action verbs are detected in input (`fix`, `add`, `implement`, `create`):

### 1. Confirm Scope

Before making changes, confirm:
- Which specific issues to address
- Which files will be modified
- Any new components needed

### 2. Make Targeted Edits

- Edit existing files using the Edit tool
- Create new components if needed
- Preserve existing functionality

### 3. Explain Changes

For each change:
- What was changed
- Why (cognitive science rationale)
- Expected impact on conversion/UX

### 4. Verify No Regressions

- Check that existing content is preserved
- Ensure page still renders
- Verify links still work

---

## Execution Instructions

1. **Parse input** to determine intent and target page (Phase 1)
2. **Discover marketing site** structure dynamically (Phase 2)
3. **Spawn both agents** with page context for parallel analysis (Phase 3)
4. **Synthesize findings** from both agents (Phase 4)
5. **Present prioritized recommendations** with file locations (Phase 5)
6. **Implement changes** if requested (Phase 6)

### Agent Coordination

When spawning agents:
- Provide identical page context to both
- Let each agent focus on their specialty
- Do not ask agents to duplicate each other's work

### Key Principles

- **Orchestrate, don't duplicate**: You synthesize; agents analyze
- **Prioritize ruthlessly**: Maximum 3 Critical issues
- **Be specific**: Include file paths and line numbers
- **Cite principles**: Reference cognitive science in recommendations
- **Suggest experiments**: Include A/B test ideas for major issues

---

## Component-Focused Audit Mode

When user specifies a component (e.g., `orchestrating-page-review hero`):

### Supported Components

| Keyword | Component Focus |
|---------|-----------------|
| `hero` | Hero section (headline, CTAs, artifact) |
| `trust`, `logos`, `social-proof` | Trust signals, logo wall, testimonials |
| `features`, `grid`, `bento` | Feature grid, value propositions |
| `pricing`, `tiers` | Pricing section, transparency |
| `cta`, `conversion` | Call-to-action placement and copy |
| `footer`, `navigation` | Footer links, navigation structure |

### Component-Focused Output

For component focus, provide deeper analysis:
- Current implementation review
- Specific improvement recommendations
- Code examples for implementation
- Benchmark examples from industry leaders
