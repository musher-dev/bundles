---
name: analyzing-competitors
version: 1.0.0
user-invocable: false
description: Systematically deconstruct competitor and aspirational SaaS marketing sites across 5 dimensions. Use when researching competitor positioning, analyzing hero patterns, extracting design systems, or building competitive intelligence. Triggered by: competitor analysis, competitive research, hero pattern, copy strategy, trust signals, design system extraction, tech stack detection, competitive positioning, market research, aspirational sites, SaaS marketing analysis.
allowed-tools: Read, Glob, Grep, WebFetch, WebSearch
---

# Competitor & Aspirational Site Analysis

## Purpose

Provide a systematic framework for deconstructing competitor and aspirational SaaS marketing sites. The output feeds directly into Brand Packet production — positioning, visual identity, trust signals, and copy strategy all derive from understanding what the market does well and where gaps exist.

This skill extracts structured intelligence from live marketing sites across 5 dimensions, producing a competitive position scorecard that informs all downstream brand decisions.

---

## 5-Dimension Extraction Model

```
┌─────────────────────────────────────────────────────────────────┐
│               COMPETITOR EXTRACTION FRAMEWORK                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  For each site, extract across 5 dimensions:                     │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  1. HERO    │  │  2. COPY    │  │  3. TRUST   │              │
│  │  LAYOUT    │  │  STRATEGY   │  │  SIGNALS    │              │
│  │  PATTERN   │  │             │  │             │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐                                │
│  │  4. DESIGN  │  │  5. TECH    │                                │
│  │  SYSTEM    │  │  STACK      │                                │
│  │  INFERENCE │  │  DETECTION  │                                │
│  └─────────────┘  └─────────────┘                                │
│                                                                  │
│  Each dimension uses specific heuristics detailed below.         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Dimension 1: Hero Layout Patterns

The hero section is the highest-signal element on any marketing page. Classify each site into one of these canonical patterns:

### Hero Pattern Taxonomy

| Pattern | Description | Typical User | Example Sites |
|---------|-------------|--------------|---------------|
| **Centered Statement** | Single bold headline centered, minimal subtext, prominent CTA | Enterprise, category leaders | Stripe, Linear |
| **Split-Column** | Left: copy + CTA. Right: product screenshot or illustration | Product-led growth | Notion, Airtable |
| **Animated Terminal** | Live code/terminal animation as hero visual | Developer tools | Vercel, Railway |
| **Metric-Led** | Large number or statistic as the dominant visual element | Data/analytics products | Datadog, Amplitude |
| **Video Background** | Full-bleed video loop behind hero copy | Experience-driven brands | Loom, Mux |
| **Interactive Demo** | Embedded interactive product preview | Product-led, freemium | Figma, CodeSandbox |
| **Logo Wall + Claim** | Bold claim backed immediately by recognizable logo bar | B2B trust-dependent | Salesforce, Lattice |
| **Problem Statement** | Leads with the pain point, not the product | Challenger brands | Basecamp, Hey |
| **Category Creation** | Defines a new category in the headline itself | First movers | Notion (early), Linear |
| **Gradient Wash** | Bold typography over gradient background, minimal imagery | Design-forward tools | Arc, Raycast |

### Extraction Checklist

- [ ] Identify primary hero pattern from taxonomy above
- [ ] Note secondary pattern if hybrid (e.g., "Centered Statement + Logo Wall")
- [ ] Record headline word count (optimal: 4-8 words)
- [ ] Record subheadline word count (optimal: 12-25 words)
- [ ] Count CTAs visible in hero (optimal: 1 primary, 0-1 secondary)
- [ ] Note CTA copy verbatim (e.g., "Get Started Free", "Book a Demo")
- [ ] Check for social proof in hero zone (logo bar, metric, testimonial)
- [ ] Measure visual weight ratio: text vs. image/media

---

## Dimension 2: Copy Strategy Analysis

### Copy Strategy Taxonomy

| Strategy | Lead Element | Signal Phrase Pattern | Example |
|----------|-------------|----------------------|---------|
| **Benefit-First** | What users gain | "Ship faster", "Build better" | Vercel: "Develop. Preview. Ship." |
| **Problem-First** | Pain point | "Tired of...", "Stop wasting..." | Basecamp: "Project management has become group therapy" |
| **Metric-Backed** | Specific numbers | "10x faster", "99.99% uptime" | Datadog: "See inside any stack, any app, at any scale" |
| **Identity-Based** | Who the user becomes | "For teams that...", "Built for..." | Linear: "Built for the way you work" |
| **Category-Creation** | New category name | "The [new-category] for..." | Notion: "The connected workspace" |
| **Technical Authority** | Architecture/capability | "Powered by...", "Built on..." | Supabase: "The open source Firebase alternative" |
| **Social Proof-Led** | Others' validation | "Trusted by...", "Used by..." | Stripe: "Millions of companies of all sizes..." |

### Copy Heuristics

When analyzing copy, evaluate these dimensions:

| Heuristic | What to Look For | Score 1-5 |
|-----------|------------------|-----------|
| **Specificity** | Concrete nouns/verbs vs. abstract adjectives | |
| **Metric density** | Numbers, percentages, timeframes in claims | |
| **Banned vocabulary** | Generic words: seamless, robust, leverage, empower | |
| **Sentence economy** | Average words per sentence (target: < 15) | |
| **Active voice** | Percentage of active vs. passive constructions | |
| **Proof proximity** | Distance between claim and supporting evidence | |

### Copy Analysis Template

```
Site: [name]
Primary Strategy: [taxonomy entry]
Secondary Strategy: [taxonomy entry, if applicable]

Headline: "[exact text]"
Subheadline: "[exact text]"
CTA Primary: "[exact text]"
CTA Secondary: "[exact text]"

Specificity Score: [1-5]
Metric Density: [count of numbers on homepage]
Banned Words Found: [list]
Avg Sentence Length: [word count]
```

---

## Dimension 3: Trust Signal Strategy

### Trust Signal Catalog

| Type | Description | Strength | Examples |
|------|-------------|----------|---------|
| **Metric** | Specific, verifiable numbers | Highest | "500K+ developers", "99.99% uptime", "2B API calls/day" |
| **Logo** | Recognizable customer/partner logos | High | Fortune 500 logos, tech company logos |
| **Badge** | Compliance/certification marks | High | SOC 2, GDPR, ISO 27001, G2 Leader |
| **Quote** | Named person + role + company | Medium-High | "Jane Smith, CTO at Acme Corp" |
| **Community** | Open source metrics, Discord/Slack size | Medium | "40K+ GitHub stars", "15K member community" |
| **Media** | Press mentions, analyst recognition | Medium | "Featured in TechCrunch", Gartner Quadrant |

### Trust Signal Extraction Template

```
Site: [name]

Metrics Found:
- [metric 1 with exact number]
- [metric 2 with exact number]

Logo Bar:
- Position: [hero-adjacent / below fold / footer]
- Count: [number of logos]
- Notable: [recognizable names]

Badges:
- [badge 1]
- [badge 2]

Testimonials:
- Named: [count]
- Anonymous: [count]
- With photo: [count]
- With company logo: [count]

Community Signals:
- [signal 1]
- [signal 2]

Media Mentions:
- [mention 1]
- [mention 2]

Trust Density Score: [1-5, see scoring below]
```

### Trust Density Scoring

| Score | Criteria |
|-------|----------|
| **5** | 3+ metrics, 8+ logos, named quotes with photos, compliance badges, community signals |
| **4** | 2+ metrics, 6+ logos, named quotes, 1+ badge |
| **3** | 1 metric, 4+ logos, quotes (may be anonymous), no badges |
| **2** | Logo bar only, no metrics, anonymous or no quotes |
| **1** | No visible trust signals above the fold |

---

## Dimension 4: Design System Inference

Reverse-engineer the visual system from what's visible on the page.

### Design System Extraction Checklist

**Colors**:
- [ ] Primary brand color (hex/HSL if detectable via DevTools)
- [ ] Background color (dark mode / light mode / both)
- [ ] Accent color(s) used for CTAs and highlights
- [ ] Text color hierarchy (primary, secondary, muted)
- [ ] Border/divider color
- [ ] Gradient usage (directions, stops)

**Typography**:
- [ ] Heading font family (inspect or infer from visual characteristics)
- [ ] Body font family
- [ ] Monospace font (if developer-focused)
- [ ] Heading weight (bold, black, medium)
- [ ] Letter-spacing (tight display text = premium signal)
- [ ] Font size range (largest heading to body text)

**Spacing**:
- [ ] Section spacing (large gaps = premium feel)
- [ ] Grid system (max-width, columns, gaps)
- [ ] Internal card/component padding
- [ ] Consistent spacing scale or ad-hoc

**Motion**:
- [ ] Page load animations (fade-in, slide-up)
- [ ] Scroll-triggered animations
- [ ] Micro-interactions (hover, click)
- [ ] Animated illustrations or backgrounds
- [ ] Motion style (subtle/refined vs. playful/bouncy)

**Component Patterns**:
- [ ] Card design (border, shadow, radius)
- [ ] Button styles (solid, outline, ghost)
- [ ] Navigation pattern (sticky, transparent, minimal)
- [ ] Footer complexity (minimal vs. mega-footer)

### Design Maturity Scoring

| Score | Indicators |
|-------|------------|
| **5** | Custom design system, tight spacing, consistent tokens, refined motion, dark mode |
| **4** | Mostly consistent, minor inconsistencies, good typography, some motion |
| **3** | Template-based but well-customized, adequate spacing, standard components |
| **2** | Generic template with minimal customization, inconsistent spacing |
| **1** | Uncustomized template, default fonts, no visual hierarchy |

---

## Dimension 5: Tech Stack Detection

### Framework Detection Heuristics

| Signal | Indicates | Detection Method |
|--------|-----------|-----------------|
| `__next` in DOM, `_next/static` in URLs | Next.js | Inspect page source, network tab |
| `__sveltekit` in DOM, `_app/` paths | SvelteKit | Inspect page source |
| `__gatsby` in DOM | Gatsby | Inspect page source |
| `data-reactroot` attribute | React (generic) | Inspect DOM |
| `ng-version` attribute | Angular | Inspect DOM |
| `data-astro-cid-*` attributes | Astro | Inspect DOM |
| `wp-content/` in URLs | WordPress | Inspect network |
| `/_nuxt/` in URLs | Nuxt | Inspect network |

### Font Detection Heuristics

| Font | Detection Signal | Common Pairing |
|------|-----------------|----------------|
| **Inter** | Wide x-height, straight-sided characters | Monospace companion |
| **Geist** | Tight metrics, Vercel ecosystem signal | Geist Mono |
| **SF Pro** | Apple ecosystem, system font stack | SF Mono |
| **Plus Jakarta Sans** | Rounded, geometric, modern SaaS feel | JetBrains Mono |
| **DM Sans** | Geometric, low contrast | DM Mono |
| **Satoshi** | Modern geometric, variable weight | IBM Plex Mono |

### CSS Framework Signals

| Signal | Indicates |
|--------|-----------|
| `class="flex items-center"` pattern | Tailwind CSS |
| `class="MuiButton-root"` pattern | Material UI |
| `class="chakra-button"` pattern | Chakra UI |
| `class="ant-btn"` pattern | Ant Design |
| CSS custom properties with `--radix-*` | Radix UI |
| `class="cl-*"` pattern | Clerk (auth) |

### Stack Detection Template

```
Site: [name]

Framework: [detected or "unknown"]
CSS: [Tailwind / custom / framework]
Fonts: [detected families]
Hosting: [Vercel / Netlify / AWS / Cloudflare]
CMS: [if detectable]
Analytics: [visible scripts]
```

---

## 40-Site Reference Corpus

This lookup table provides pre-analyzed data from 40 elite SaaS marketing sites. Use as a starting point for competitive analysis — verify current state via live site inspection.

### Developer Tools & Infrastructure

| Site | Industry | Hero Pattern | Copy Strategy | Notable Copy | Trust Signal | Inferred Stack |
|------|----------|-------------|---------------|-------------|-------------|---------------|
| **Stripe** | Payments | Centered Statement | Benefit-First | "Financial infrastructure for the internet" | Metric (millions of companies) + Logo Wall | Next.js, Custom CSS |
| **Vercel** | Deployment | Animated Terminal | Technical Authority | "Develop. Preview. Ship." | Community (GitHub stars) + Logo Wall | Next.js, Geist |
| **Supabase** | BaaS | Split-Column | Category-Creation | "The open source Firebase alternative" | Community (48K+ GitHub stars) | Next.js, Tailwind |
| **Railway** | PaaS | Animated Terminal | Benefit-First | "Bring your code, we'll handle the rest" | Metric + Logo | Next.js, Custom |
| **PlanetScale** | Database | Centered Statement | Technical Authority | "The database for developers" | Logo Wall + Metric | Next.js, Tailwind |
| **Neon** | Database | Split-Column | Technical Authority | "Serverless Postgres" | Community + Logo | Next.js, Tailwind |
| **Turso** | Database | Centered Statement | Technical Authority | "SQLite for production" | Community (GitHub stars) | Astro, Tailwind |
| **Resend** | Email API | Gradient Wash | Benefit-First | "Email for developers" | Logo + Community | Next.js, Tailwind |
| **Inngest** | Workflows | Split-Column | Benefit-First | "Ship reliable functions, zero infrastructure" | Logo Wall + Metric | Next.js, Tailwind |
| **Temporal** | Workflows | Centered Statement | Technical Authority | "Durable execution for developers" | Logo Wall (enterprise) | Next.js, Custom |

### Productivity & Collaboration

| Site | Industry | Hero Pattern | Copy Strategy | Notable Copy | Trust Signal | Inferred Stack |
|------|----------|-------------|---------------|-------------|-------------|---------------|
| **Linear** | Project Mgmt | Centered Statement | Identity-Based | "Linear is a purpose-built tool for planning and building products" | Logo Wall + Community | Next.js, Custom CSS |
| **Notion** | Workspace | Split-Column | Category-Creation | "The connected workspace" | Metric (millions of teams) + Logo | Next.js, Custom |
| **Figma** | Design | Interactive Demo | Benefit-First | "The collaborative interface design tool" | Logo Wall + Community | Custom, WebGL |
| **Loom** | Video | Video Background | Problem-First | "Video messaging for async work" | Metric (25M+) + Logo | Next.js, Tailwind |
| **Slack** | Messaging | Split-Column | Benefit-First | "Where work happens" | Metric + Logo Wall | Custom, React |
| **Miro** | Whiteboard | Split-Column | Benefit-First | "The visual workspace for innovation" | Metric (60M+) + Logo | Next.js, Custom |
| **Coda** | Docs | Split-Column | Category-Creation | "The doc that brings it all together" | Logo Wall + Testimonials | Next.js, Custom |
| **Airtable** | Database | Split-Column | Benefit-First | "The platform to build next-gen apps" | Logo Wall (enterprise) | Next.js, Custom |

### Security & Compliance

| Site | Industry | Hero Pattern | Copy Strategy | Notable Copy | Trust Signal | Inferred Stack |
|------|----------|-------------|---------------|-------------|-------------|---------------|
| **1Password** | Passwords | Split-Column | Problem-First | "The world's most-loved password manager" | Metric + Badge + Quote | Next.js, Custom |
| **Tailscale** | Networking | Centered Statement | Technical Authority | "Private networks made easy" | Logo Wall + Community | Hugo, Custom |
| **Snyk** | Security | Split-Column | Problem-First | "Developer-first security" | Logo Wall + Badge + Metric | Next.js, Tailwind |
| **Vanta** | Compliance | Logo Wall + Claim | Benefit-First | "Automate compliance. Simplify security." | Logo Wall + Badge (SOC 2) | Next.js, Custom |
| **Drata** | Compliance | Logo Wall + Claim | Benefit-First | "Put security compliance on autopilot" | Logo Wall + Badge + Metric | Next.js, Tailwind |

### Analytics & Data

| Site | Industry | Hero Pattern | Copy Strategy | Notable Copy | Trust Signal | Inferred Stack |
|------|----------|-------------|---------------|-------------|-------------|---------------|
| **Datadog** | Monitoring | Metric-Led | Technical Authority | "See inside any stack, any app, at any scale" | Logo Wall (enterprise) | Custom, React |
| **Amplitude** | Analytics | Split-Column | Benefit-First | "Build better products" | Logo Wall + Metric | Next.js, Custom |
| **PostHog** | Analytics | Gradient Wash | Category-Creation | "The single platform for engineers" | Community (GitHub) + Metric | Next.js, Tailwind |
| **Mixpanel** | Analytics | Split-Column | Benefit-First | "Powerful, self-serve product analytics" | Logo Wall + Metric | Next.js, Custom |
| **Segment** | CDP | Centered Statement | Technical Authority | "The leading customer data platform" | Logo Wall (enterprise) | Next.js, Custom |

### HR & People

| Site | Industry | Hero Pattern | Copy Strategy | Notable Copy | Trust Signal | Inferred Stack |
|------|----------|-------------|---------------|-------------|-------------|---------------|
| **Lattice** | HR Platform | Logo Wall + Claim | Benefit-First | "People management that drives performance" | Logo Wall (enterprise) + Metric | Next.js, Custom |
| **Rippling** | HR + IT | Split-Column | Benefit-First | "The workforce platform that starts with HR and IT" | Logo Wall + Metric | Next.js, Custom |
| **Deel** | Global HR | Split-Column | Metric-Backed | "Payroll and compliance for global teams" | Metric (35K+ companies) + Logo | Next.js, Tailwind |
| **Gusto** | Payroll | Split-Column | Benefit-First | "The people platform" | Metric (300K+ businesses) + Logo | Next.js, Custom |

### AI & ML

| Site | Industry | Hero Pattern | Copy Strategy | Notable Copy | Trust Signal | Inferred Stack |
|------|----------|-------------|---------------|-------------|-------------|---------------|
| **Anthropic** | AI Safety | Centered Statement | Identity-Based | "AI safety research company" | Media + Research | Next.js, Tailwind |
| **OpenAI** | AI Platform | Centered Statement | Category-Creation | "Creating safe AGI" | Media + Metric | Next.js, Custom |
| **Hugging Face** | ML Platform | Gradient Wash | Community-Led | "The AI community building the future" | Community (models, datasets) | Next.js, Custom |
| **Weights & Biases** | ML Ops | Split-Column | Benefit-First | "The AI developer platform" | Logo Wall + Community | Next.js, Tailwind |
| **Pinecone** | Vector DB | Centered Statement | Technical Authority | "The vector database for AI" | Logo Wall + Metric | Next.js, Tailwind |
| **Distributional** | AI Testing | Centered Statement | Problem-First | "Test AI systems before they fail" | Badge + Technical Authority | Next.js, Tailwind |

### Fintech & Commerce

| Site | Industry | Hero Pattern | Copy Strategy | Notable Copy | Trust Signal | Inferred Stack |
|------|----------|-------------|---------------|-------------|-------------|---------------|
| **Plaid** | Banking API | Split-Column | Technical Authority | "The easiest way to connect with financial data" | Logo Wall + Metric | Next.js, Custom |
| **Mercury** | Banking | Centered Statement | Identity-Based | "The financial stack for startups" | Logo Wall + Testimonials | Next.js, Tailwind |
| **Shopify** | Commerce | Split-Column | Benefit-First | "Making commerce better for everyone" | Metric (millions of businesses) | Custom, React |
| **Paddle** | Billing | Split-Column | Benefit-First | "The complete payments infrastructure for SaaS" | Logo Wall + Metric | Next.js, Tailwind |

---

## Competitive Position Scoring

After extracting data across all 5 dimensions, produce a comparative scorecard.

### Scoring Template

Rate each competitor (and your product) on a 1-5 scale across all 5 dimensions:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                    COMPETITIVE POSITION SCORECARD                        │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Dimension            │ Our Product │ Comp A │ Comp B │ Comp C │ Aspire │
│  ─────────────────────┼─────────────┼────────┼────────┼────────┼────────│
│  1. Hero Impact       │     /5      │   /5   │   /5   │   /5   │   /5   │
│  2. Copy Quality      │     /5      │   /5   │   /5   │   /5   │   /5   │
│  3. Trust Density     │     /5      │   /5   │   /5   │   /5   │   /5   │
│  4. Design Maturity   │     /5      │   /5   │   /5   │   /5   │   /5   │
│  5. Tech Modernity    │     /5      │   /5   │   /5   │   /5   │   /5   │
│  ─────────────────────┼─────────────┼────────┼────────┼────────┼────────│
│  TOTAL                │     /25     │   /25  │   /25  │   /25  │   /25  │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Scoring Criteria per Dimension

| Score | Hero Impact | Copy Quality | Trust Density | Design Maturity | Tech Modernity |
|-------|-------------|-------------|---------------|-----------------|----------------|
| **5** | Instantly memorable, unique pattern, perfect CTA placement | Zero generic words, metric-dense, concise | 3+ metrics, 8+ logos, named quotes, badges | Custom system, consistent tokens, refined motion | Latest framework, edge-rendered, < 1s LCP |
| **4** | Strong pattern execution, clear hierarchy | Mostly specific, 1-2 generic words, good proof | 2+ metrics, 6+ logos, quotes with names | Mostly consistent, good typography | Modern framework, decent performance |
| **3** | Competent but unremarkable, follows conventions | Mix of specific and generic, some proof | 1 metric, logos, some quotes | Template-based but customized | Standard framework, acceptable speed |
| **2** | Cluttered or unclear hierarchy | Mostly generic adjectives, vague claims | Logo bar only, no metrics | Generic template, inconsistent | Older framework, slow load |
| **1** | Confusing, no clear visual hierarchy | All generic, no proof, banned vocabulary | No visible trust signals | Uncustomized, broken spacing | Legacy tech, poor performance |

---

## Instructions

When performing a competitive analysis:

1. **Identify 3-5 direct competitors** (same category, similar audience) and **2-3 aspirational sites** (best-in-class marketing regardless of category)
2. **For each site**, extract data across all 5 dimensions using the checklists and templates above
3. **Reference the 40-site corpus** for pattern comparison — note which established patterns each competitor follows
4. **Score each site** using the Competitive Position Scoring template
5. **Identify gaps and opportunities**:
   - Where do ALL competitors score low? (Market opportunity)
   - Where does the aspirational set score high? (Best practice to adopt)
   - Which hero pattern is overused in the category? (Differentiation chance)
   - What trust signals does the category lack? (Trust gap to fill)
6. **Produce a summary** with:
   - Pattern frequency table (which hero patterns dominate the category)
   - Copy strategy distribution (what approach is most common)
   - Trust signal baseline (minimum expected by category)
   - Design system benchmark (what quality level is table stakes)
   - Recommended positioning angle (based on gaps found)

---

## Quality Checklist

- [ ] Minimum 3 direct competitors and 2 aspirational sites analyzed
- [ ] All 5 dimensions extracted for each site
- [ ] Hero pattern classified using taxonomy (not ad-hoc descriptions)
- [ ] Copy strategy classified using taxonomy (not ad-hoc descriptions)
- [ ] Trust signals cataloged with specific counts and types
- [ ] Design system inference includes colors, typography, spacing, motion
- [ ] Tech stack detection attempted for each site
- [ ] Competitive Position Scorecard filled with 1-5 scores
- [ ] Gap analysis identifies at least 2 positioning opportunities
- [ ] Output references the 40-site corpus for pattern comparison

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Shallow scan** | Skimming homepages without systematic extraction | Use the 5-dimension checklist for every site |
| **Copy-paste analysis** | Noting what competitors do without scoring | Always produce the comparative scorecard |
| **Aspiration blind spot** | Only analyzing direct competitors | Include 2-3 best-in-class sites from any category |
| **Recency bias** | Assuming current state is intentional | Check Wayback Machine for recent changes |
| **Feature comparison** | Comparing product features, not marketing execution | Focus on how they market, not what they sell |
| **Single dimension focus** | Deep analysis of copy but ignoring design/trust | Extract all 5 dimensions for balanced view |

---

## Edge Cases

### Pre-Launch Products
- Focus on landing page patterns and waitlist conversion
- Trust signals shift to founder credentials, investor logos, beta user count
- Design maturity expectations lower, but typography still matters

### Open Source Projects
- Community signals (GitHub stars, contributors) outweigh enterprise logos
- Documentation quality substitutes for traditional marketing copy
- Hero patterns often favor Animated Terminal or Interactive Demo

### Enterprise-Only Products
- Marketing sites may be thin (gated behind sales)
- Trust signals heavily weighted toward badges, analyst reports
- Analyze what's public + any ungated resources (whitepapers, case studies)

### Category Creators
- No direct competitors to analyze — focus on adjacent categories
- Aspirational set becomes primary reference
- Copy strategy analysis focuses on category-naming effectiveness

---

## Related Skills and Agents

- **`writing-brand-positioning` skill**: Consumes competitive analysis to define positioning, ICP, and differentiators
- **`writing-brand-voice` skill**: Uses competitor copy analysis to define tonal differentiation
- **`writing-visual-identity` skill**: Uses design system inference to establish visual direction
- **`writing-trust-signals` skill**: Uses trust signal catalog to set trust strategy
- **`auditing-trust-engineering` skill**: For deeper privacy and compliance trust analysis
- **`auditing-page-velocity` skill**: For performance benchmarking against competitors
- **`copy_writer` agent**: For writing copy informed by competitive analysis

**When to use which**: Use this skill as the entry point for Brand Packet production. Run this first, then feed findings into the writing skills for positioning, voice, visual identity, and trust signals.
