---
name: writing-trust-signals
version: 1.0.0
user-invocable: false
description: Identify, catalog, and validate trust signals for marketing pages including metrics, logos, badges, quotes, community signals, and media mentions. Use when sourcing trust signals, defining trust strategy, validating claim provability, or establishing minimum trust requirements. Triggered by: trust signals, social proof, customer logos, testimonials, compliance badges, metrics, community signals, trust density, trust strategy, proof points, media mentions, case studies, provability.
allowed-tools: Read, Glob, Grep, WebFetch, WebSearch
---

# Trust Signal Strategy

## Purpose

Identify, catalog, and validate trust signals that prove brand claims on marketing pages. Trust signals form Field 8 of the Brand Packet and directly support the differentiators defined in Field 4.

Every claim on a marketing page is a debt until it's backed by evidence. Trust signals are that evidence. This skill provides the methodology for finding, scoring, and placing trust signals that convert skeptics into believers.

---

## Trust Signal Taxonomy

### 6 Types in Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRUST SIGNAL HIERARCHY                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STRONGEST ───────────────────────────────── WEAKEST             │
│                                                                  │
│  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐   │
│  │METRIC│  │ LOGO │  │BADGE │  │QUOTE │  │COMMUN│  │MEDIA │   │
│  │      │  │      │  │      │  │      │  │      │  │      │   │
│  │Verif.│  │Recog.│  │Third │  │Named │  │Peer  │  │Press │   │
│  │data  │  │brands│  │party │  │person│  │valid.│  │cover.│   │
│  │      │  │      │  │audit │  │      │  │      │  │      │   │
│  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘  └──────┘   │
│                                                                  │
│  Hierarchy rationale:                                            │
│  • Metrics: hardest to fake, most falsifiable                    │
│  • Logos: instant recognition, borrowed credibility              │
│  • Badges: third-party validation, independently verifiable      │
│  • Quotes: personal testimony, specific but harder to verify     │
│  • Community: peer validation, impressive but gameable           │
│  • Media: external coverage, valuable but often paid/earned mix  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Type Definitions

| Type | Definition | Minimum Requirements | Examples |
|------|-----------|---------------------|---------|
| **Metric-Based** | Specific, verifiable numbers that prove scale or performance | Must be current (< 12 months), sourced, and falsifiable | "500K+ developers", "99.99% uptime", "2B API calls/day", "deployed in 150 countries" |
| **Logo-Based** | Recognizable customer or partner logos displayed prominently | Must have permission, represent real customers, be current | Fortune 500 logos, tech unicorn logos, industry leaders |
| **Badge-Based** | Compliance certifications, review platform awards, analyst recognition | Must be current, link to verification, display certification date | SOC 2 Type II, GDPR, G2 Leader badge, Gartner recognition |
| **Quote-Based** | Named testimonials with person, role, company, and ideally photo | Must be attributed (name + role + company), metric-backed preferred | "'Reduced deploy time 80%' — Jane Smith, CTO, Acme Corp" |
| **Community-Based** | Open source metrics, community size, contributor count | Must be verifiable (link to repo, Discord, etc.) | "40K GitHub stars", "15K Discord members", "500 contributors" |
| **Media-Based** | Press coverage, analyst reports, industry awards | Must link to source, be from recognized publications | "Featured in TechCrunch", "Forrester Wave Leader" |

---

## Hyper-Specificity Requirements

### Generic vs. Specific Comparison

Trust signals MUST be hyper-specific. Generic signals are noise — they don't differentiate and they don't convince.

| Generic (BANNED) | Specific (REQUIRED) | Why Specific Wins |
|------------------|--------------------|--------------------|
| "Trusted by thousands of companies" | "Trusted by 12,847 companies across 42 countries" | Odd numbers feel real, round numbers feel invented |
| "Industry-leading performance" | "p99 latency under 50ms at 10K req/s" | Technical audience respects measurable claims |
| "Top-rated on G2" | "G2 Leader, Winter 2026 — 4.8/5 from 847 reviews" | Specific season + score + count is verifiable |
| "Used by Fortune 500" | "Used by 6 of the Fortune 10, including [Company]" | Naming companies (with permission) adds weight |
| "99.9% uptime" | "99.997% uptime over the last 36 months (public status page)" | Extra decimal + timeframe + link = credible |
| "Backed by top investors" | "Series B: $45M led by Sequoia and a16z" | Names + amounts are verifiable |
| "Award-winning product" | "2025 InfoWorld Technology of the Year — Edge Computing" | Specific award + category + year |
| "Fast-growing community" | "From 0 to 40K GitHub stars in 18 months" | Growth trajectory is more compelling than static number |

### Specificity Rules

1. **Use odd or precise numbers** — "12,847" is more believable than "13,000"
2. **Include timeframes** — "in the last 12 months" or "since 2023"
3. **Name names** — companies, people, publications (with permission)
4. **Link to sources** — status pages, G2 profiles, press articles
5. **Add context** — "2B API calls/day" is more impressive with "that's 23K per second"
6. **Date the claim** — "as of Q4 2025" or "verified January 2026"

---

## Provability Test

Every trust signal must pass the 3-question provability test:

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROVABILITY TEST                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Question 1: SOURCE DATA                                         │
│  "Can you point to the source data right now?"                   │
│                                                                  │
│  ✓ Internal dashboard showing the metric                         │
│  ✓ G2 profile page showing rating + review count                │
│  ✓ Customer email granting logo permission                      │
│  ✗ "I think we have about 500K users"                           │
│  ✗ "Our CEO mentioned this in a talk"                           │
│                                                                  │
│  Question 2: JOURNALIST TEST                                     │
│  "If a journalist fact-checked this claim, would it hold?"       │
│                                                                  │
│  ✓ Published case study with named customer                     │
│  ✓ Public status page showing uptime                            │
│  ✓ SEC filing or press release with numbers                     │
│  ✗ "Best-in-class" with no third-party benchmark                │
│  ✗ Testimonial from unnamed "enterprise customer"               │
│                                                                  │
│  Question 3: CURRENCY                                            │
│  "Is this still true today? When was it last verified?"          │
│                                                                  │
│  ✓ Auto-updating metric from live data                          │
│  ✓ Reviewed and confirmed within last 6 months                  │
│  ✓ Certification with visible expiry/renewal date               │
│  ✗ Metric from 2 years ago, never re-checked                    │
│  ✗ Testimonial from a customer who has since churned            │
│                                                                  │
│  VERDICT: Signal must pass ALL 3 questions.                      │
│  Failing ANY question = signal is not ready for the page.        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Trust Signal Sourcing

Where to find trust signals when building a Brand Packet:

### Metric Sources

| Source | Signal Type | How to Access |
|--------|-----------|---------------|
| Product analytics | User count, API calls, uptime | Internal dashboards (Amplitude, PostHog, Datadog) |
| Billing system | Revenue metrics, customer count | Stripe dashboard, CRM |
| Public status page | Uptime percentage, incident count | status.example.com |
| GitHub | Stars, forks, contributors, issues closed | Public API or profile page |
| App stores | Rating, review count, download count | iOS App Store, Google Play |
| Performance benchmarks | Latency, throughput, response time | Internal benchmarks or third-party tests |

### Logo Sources

| Source | How to Obtain Permission |
|--------|-------------------------|
| Sales CRM | Check logo permission clause in customer contracts |
| Customer success | Request logo rights during QBR/renewal |
| Case study participants | Logo rights often included in case study agreement |
| Partner agreements | Partner logos typically include co-marketing rights |
| Public customer pages | If customer publicly lists you as a vendor, reciprocal use is common |

### Quote Sources

| Source | Quality Level |
|--------|-------------|
| Case study interviews | Highest — structured, approved, quotable |
| G2/Capterra reviews | High — public, attributable, third-party hosted |
| Customer success check-ins | Medium — need explicit approval to quote |
| Social media mentions | Medium — public, but verify context and permission |
| NPS survey responses | Low — often anonymous, need permission to attribute |

### Badge Sources

| Badge Type | Source |
|-----------|--------|
| SOC 2 Type II | Your audit firm (Drata, Vanta, Secureframe) |
| GDPR compliance | Legal/compliance team documentation |
| G2 badges | G2 seller dashboard (Leader, High Performer, etc.) |
| Industry awards | Award organization website |
| Analyst recognition | Gartner, Forrester, IDC reports |

---

## Trust Density Scoring

| Score | Criteria | Example |
|-------|----------|---------|
| **5** | 3+ specific metrics, 8+ recognizable logos, 2+ named quotes with photos, compliance badges, community signals | Stripe, Datadog |
| **4** | 2+ metrics, 6+ logos, named quotes (may lack photos), 1+ badge | Linear, Supabase |
| **3** | 1 metric, 4+ logos, quotes present (may be anonymous), no badges | Mid-stage SaaS |
| **2** | Logo bar only, no quantified metrics, anonymous or absent quotes | Early growth SaaS |
| **1** | No visible trust signals above the fold | Pre-launch / stealth |

---

## Placement Strategy

### By Page Zone

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRUST SIGNAL PLACEMENT MAP                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────┐                    │
│  │           HERO ZONE                      │                    │
│  │  • Logo bar (6-8 logos, immediately      │                    │
│  │    below hero headline)                  │                    │
│  │  • Primary metric ("500K+ developers")   │                    │
│  │  • Compliance badges near CTA            │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
│  ┌─────────────────────────────────────────┐                    │
│  │           POST-CLAIM ZONE                │                    │
│  │  • Testimonial quote after each major    │                    │
│  │    feature claim (claim → proof cycle)   │                    │
│  │  • Metric that validates the claim       │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
│  ┌─────────────────────────────────────────┐                    │
│  │           FEATURE SECTIONS               │                    │
│  │  • Feature-specific testimonial          │                    │
│  │  • Feature-specific metric               │                    │
│  │  • Case study link for that use case     │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
│  ┌─────────────────────────────────────────┐                    │
│  │           PRICING ZONE                   │                    │
│  │  • Enterprise logos (reduce buyer fear)  │                    │
│  │  • ROI metric or customer quote          │                    │
│  │  • Compliance badges (for enterprise)    │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
│  ┌─────────────────────────────────────────┐                    │
│  │           FOOTER ZONE                    │                    │
│  │  • Extended logo bar (all customers)     │                    │
│  │  • Trust Center link                     │                    │
│  │  • Compliance badge summary              │                    │
│  │  • Community links (GitHub, Discord)     │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Placement Rules

1. **Hero-adjacent is mandatory** — minimum: logo bar within 100px of hero CTA
2. **Claim → Proof cycle** — every major claim should be followed by a trust signal within 1 scroll viewport
3. **Match signal to context** — feature sections use feature-relevant quotes, not generic testimonials
4. **Repeat strategically** — logos can appear multiple times (hero, pricing, footer) without feeling redundant
5. **Proximity to conversion** — trust signals intensify near CTAs and pricing

---

## Minimum Requirements

Every Brand Packet trust strategy must meet these minimums:

| Signal Type | Minimum | Notes |
|-------------|---------|-------|
| **Metrics** | 3 specific, provable numbers | At least 1 scale metric + 1 performance metric |
| **Logos** | 6 recognizable customer logos | Must be from companies the ICP would recognize |
| **Testimonials** | 2 named quotes (person + role + company) | Metric-backed quotes preferred |
| **Badges** | 1 compliance or review badge | SOC 2, G2, or industry-specific |
| **Community** | 1 community signal (if applicable) | GitHub stars, community size, contributor count |

### If Minimums Can't Be Met

```
Priority Order for Signal Acquisition:

1. Logos — easiest to obtain (customer success outreach)
2. Metrics — often already exist in dashboards
3. Quotes — request during next customer check-in
4. Badges — initiate compliance audit (Vanta/Drata accelerate SOC 2)
5. Community — grow organically, don't fabricate

If truly unable to meet minimums:
- Use placeholders: "[Metric: target Q2]" in Brand Packet
- Flag gaps in the validation report
- Prioritize acquisition in marketing roadmap
```

---

## Instructions

When identifying and cataloging trust signals:

1. **Review differentiators** from `writing-brand-positioning` — each differentiator needs proof
2. **Audit existing signals** — search the product, website, CRM, and public profiles for existing trust data
3. **Catalog by type** — organize all found signals into the 6-type taxonomy
4. **Apply the Provability Test** — every signal must pass all 3 questions
5. **Score specificity** — replace any generic signals with hyper-specific versions
6. **Score trust density** — rate the overall collection on the 1-5 scale
7. **Define placement strategy** — assign signals to page zones
8. **Check minimums** — verify all minimum requirements are met, flag gaps
9. **Output Field 8** of the Brand Packet:
   - Trust Signal Catalog (organized by type)
   - Trust Density Score
   - Placement Strategy
   - Gap Analysis (signals needed but not yet available)

---

## Quality Checklist

- [ ] All 6 trust signal types evaluated (metric, logo, badge, quote, community, media)
- [ ] Every signal passes the 3-question Provability Test
- [ ] No generic signals remain (all are hyper-specific)
- [ ] Minimum requirements met (3 metrics, 6 logos, 2 named quotes, 1 badge)
- [ ] Placement strategy assigns signals to all 5 page zones
- [ ] Trust density score calculated (target: 4+)
- [ ] Each differentiator from Field 4 has at least 1 supporting trust signal
- [ ] Gaps identified with acquisition timeline

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| **Round number syndrome** | "10,000+ customers" feels invented | Use precise numbers: "12,847 companies" |
| **Anonymous testimonials** | "A Fortune 500 CTO" has no credibility | Name the person, role, and company |
| **Stale metrics** | "As of 2023" on a 2026 page | Add auto-refresh or review cadence |
| **Logo graveyard** | 30+ tiny logos nobody can read | Curate to 6-8 recognizable logos per placement |
| **Trust signal desert** | All signals in footer, nothing near hero | Place strongest signals hero-adjacent |
| **Unverifiable claims** | "Best-in-class" with no benchmark | Replace with specific, sourced metric |
| **Testimonial without context** | "Great product! 5 stars" | Request metric-backed quotes with specifics |

---

## Edge Cases

### Pre-Launch Products
- Investor logos replace customer logos (with permission)
- Founder credentials serve as trust signals (previous exits, domain expertise)
- Beta user metrics ("500 beta users, 94% weekly active")
- Waitlist size as social proof ("12,000 on waitlist")
- Advisory board names if notable

### Open Source Projects
- GitHub metrics are primary trust signals (stars, forks, contributors)
- Community size (Discord/Slack members) replaces customer logos
- Contributor diversity signals project health
- Download/install counts from package managers
- Adoption by recognizable projects or companies

### Certifications in Progress
- "SOC 2 Type II in progress — estimated completion Q2 2026"
- Link to security questionnaire or trust center with current practices
- Be transparent about timeline — in-progress is better than silent
- Display completed certifications alongside in-progress ones

### B2C or Consumer Products
- App store ratings replace enterprise badges
- User counts and DAU/MAU ratios are primary metrics
- Press coverage and media mentions carry more weight
- Influencer endorsements substitute for enterprise testimonials
- Awards (Product Hunt, Webby) serve as third-party validation

---

## Related Skills and Agents

- **`writing-brand-positioning` skill**: Differentiators (Field 4) need trust signals as proof
- **`analyzing-competitors` skill**: Competitor trust signal analysis reveals category baseline
- **`auditing-trust-engineering` skill**: Deep audit of privacy and compliance trust architecture
- **`validating-brand-packets` skill**: Validates trust signal completeness and quality
- **`reviewing-copy` skill**: Ensures copy claims are backed by trust signals
- **`copy_writer` agent**: Integrates trust signals into page copy

**When to use which**: Use this skill to source and catalog trust signals (Field 8). Use `auditing-trust-engineering` for deeper privacy/compliance trust analysis. Use `reviewing-copy` to verify that published copy includes adequate proof.
