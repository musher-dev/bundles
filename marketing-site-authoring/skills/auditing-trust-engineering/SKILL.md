---
name: auditing-trust-engineering
user-invocable: false
description: Audit and design privacy-first trust architecture for marketing pages. Use when reviewing compliance badges, consent UX, trust centers, or privacy messaging. Triggered by: privacy first, trust engineering, consent design, zero knowledge, trust center, compliance badges, GDPR consent, cookie consent, privacy UX, data transparency, trust signals.
allowed-tools: Read, Grep, Glob
---

# Trust Engineering Audit

## Purpose

Provide a systematic framework for auditing and designing trust architecture on marketing pages. Privacy-first design is a competitive differentiator - 79% of consumers are concerned about data usage, and transparent trust signals directly impact conversion rates.

## Trust Signal Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRUST SIGNAL HIERARCHY                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TIER 1: Immediate Trust (Above Fold)                           │
│  ├── Company logos of notable customers                         │
│  ├── Key compliance badges (SOC 2, GDPR)                        │
│  └── Trust-building headline language                           │
│                                                                  │
│  TIER 2: Contextual Trust (Near CTAs)                           │
│  ├── Security badges adjacent to forms                          │
│  ├── Privacy reassurance near data collection                   │
│  └── Social proof near conversion points                        │
│                                                                  │
│  TIER 3: Deep Trust (Discoverable)                              │
│  ├── Trust Center link in footer/nav                            │
│  ├── Detailed security documentation                            │
│  └── Compliance certifications page                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Compliance Badge Strategy

### Placement Principles

```
┌─────────────────────────────────────────────────────────────────┐
│                    BADGE PLACEMENT RULES                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  GOOD: Badges near CTAs and forms                               │
│  ┌─────────────────────────────────────────┐                    │
│  │  [Email input____________]  [Sign Up]   │                    │
│  │  🔒 SOC 2 Certified  |  GDPR Compliant  │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
│  BAD: Badges buried in footer only                              │
│  ┌─────────────────────────────────────────┐                    │
│  │  [Email input____________]  [Sign Up]   │                    │
│  │                                         │                    │
│  │  ... 2000px of content ...              │                    │
│  │                                         │                    │
│  │  ───────────────────────────────────    │                    │
│  │  Footer: SOC 2 | GDPR | HIPAA           │  ← Too late!       │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Badge Selection Matrix

| Badge | When to Display | Placement Priority |
|-------|-----------------|-------------------|
| **SOC 2 Type II** | B2B SaaS, handles customer data | Near signup forms, pricing |
| **GDPR Compliant** | EU traffic > 10%, any EU data | Near data collection points |
| **HIPAA** | Healthcare data | Near healthcare-specific CTAs |
| **ISO 27001** | Enterprise sales | Trust Center, security page |
| **PCI DSS** | Payment processing | Near payment forms |
| **SSL/TLS** | All sites | Browser shows this; don't badge |

### Anti-Pattern: Badge Overload

```
BAD:
[SOC2] [GDPR] [HIPAA] [ISO27001] [PCI] [SSL] [Privacy Shield] [CCPA]
                    ↑
        Too many badges = diluted trust

GOOD:
[SOC 2 Type II] [GDPR Compliant]
"View all certifications →"
                    ↑
        Focused badges + link to Trust Center
```

---

## Privacy-First Design Patterns

### Pattern 1: Zero-Knowledge Messaging

Communicate architectural privacy guarantees upfront:

```
SIGNAL (messaging app):
"Messages are end-to-end encrypted.
We can't read your messages, and neither can anyone else."
                    ↑
        Zero-knowledge claim + explanation

1PASSWORD:
"Only you can access your vaults.
Not even 1Password employees can see your data."
                    ↑
        Explicit "we can't see" statement
```

**Implementation Checklist**:
- [ ] State what data you DON'T collect
- [ ] Explain what you CAN'T access
- [ ] Use "we never" language for strong guarantees
- [ ] Link to technical documentation for verification

### Pattern 2: Fear-Free Messaging

Address data concerns before they become objections:

```
DUCKDUCKGO:
"We don't track you."
"Your search history is private, even from us."
"No personal information. No history. No tracking."
                    ↑
        Direct negation of fear

APPLY TO YOUR PAGES:
Instead of: "We use your data responsibly"
Write:      "We don't sell your data. Ever."

Instead of: "Your data is secure"
Write:      "Your data never leaves your control"
```

### Pattern 3: Transparent Data Flow

Show users exactly what happens to their data:

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATA FLOW TRANSPARENCY                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  When you sign up, here's what happens:                         │
│                                                                  │
│  1. Email → Stored encrypted in US data center                  │
│  2. Usage → Anonymized for product improvement                  │
│  3. Payment → Processed by Stripe (we never see card numbers)   │
│                                                                  │
│  What we DON'T do:                                              │
│  ✗ Sell data to third parties                                   │
│  ✗ Track you across other websites                              │
│  ✗ Store data longer than needed                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Consent UX Framework

### Granular Consent Modal Design

```
┌─────────────────────────────────────────────────────────────────┐
│                    COOKIE CONSENT PATTERNS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BAD: Dark Pattern                                               │
│  ┌─────────────────────────────────────────┐                    │
│  │  We use cookies.                        │                    │
│  │  [Accept All]  [Manage (tiny text)]     │  ← Manipulative   │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
│  GOOD: Equal Visual Weight                                       │
│  ┌─────────────────────────────────────────┐                    │
│  │  We use cookies to improve your         │                    │
│  │  experience. Choose what to allow:      │                    │
│  │                                         │                    │
│  │  [Accept All]  [Reject Non-Essential]   │  ← Equal buttons  │
│  │  [Customize]                            │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
│  EXCELLENT: Granular + Default-Off                               │
│  ┌─────────────────────────────────────────┐                    │
│  │  Choose your cookie preferences:        │                    │
│  │                                         │                    │
│  │  [✓] Essential (required for site)      │                    │
│  │  [ ] Analytics (help us improve)        │  ← Default OFF     │
│  │  [ ] Marketing (personalized ads)       │  ← Default OFF     │
│  │                                         │                    │
│  │  [Save Preferences]                     │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Consent UX Checklist

- [ ] **Equal visual weight** for accept/reject options
- [ ] **Reject button visible** without scrolling or clicking
- [ ] **No pre-checked boxes** for non-essential cookies
- [ ] **Plain language** (not legalese)
- [ ] **Granular toggles** for each cookie category
- [ ] **Easy to change later** (preference center accessible)
- [ ] **No consent wall** blocking content access
- [ ] **Remember choice** (don't ask repeatedly)

---

## Trust Center Implementation

A Trust Center centralizes security and privacy information:

### Essential Trust Center Sections

```
Trust Center Structure:
├── Overview
│   ├── Security philosophy statement
│   ├── Key certifications (with dates)
│   └── Contact for security inquiries
│
├── Data Protection
│   ├── Encryption (at rest, in transit)
│   ├── Data residency options
│   ├── Retention policies
│   └── Deletion/export procedures
│
├── Compliance
│   ├── SOC 2 report (gated or summary)
│   ├── GDPR compliance documentation
│   ├── Industry-specific (HIPAA, PCI, etc.)
│   └── Audit frequency and results
│
├── Infrastructure
│   ├── Cloud provider(s)
│   ├── Data center locations
│   ├── Uptime SLA
│   └── Disaster recovery
│
└── Vendor Security
    ├── Sub-processor list
    ├── Third-party audit results
    └── Vendor security requirements
```

### Trust Center Anti-Patterns

| Anti-Pattern | Problem | Fix |
|--------------|---------|-----|
| Gated behind sales | Hides information | Make publicly accessible |
| PDF-only docs | Not searchable, outdated | Use web pages with version dates |
| No contact info | Can't ask questions | Include security@email |
| Generic statements | "We take security seriously" | Specific claims with evidence |
| Outdated certs | Shows negligence | Display cert dates, auto-expire badges |

---

## Social Proof Placement

### Contextual Social Proof Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOCIAL PROOF PLACEMENT                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Hero Section:                                                   │
│  → Logo bar of recognizable customers                           │
│  → "Trusted by 10,000+ companies"                               │
│                                                                  │
│  Near Signup Form:                                               │
│  → Testimonial quote from similar company                       │
│  → "Join [Company] and 500+ others"                             │
│                                                                  │
│  Pricing Page:                                                   │
│  → Enterprise logo bar (reduce enterprise buyer anxiety)        │
│  → "Used by Fortune 500 companies"                              │
│                                                                  │
│  Feature Section:                                                │
│  → Testimonial relevant to that feature                         │
│  → Case study link for that use case                            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Effective Testimonial Patterns

```
WEAK:
"Great product!" - John D.
                ↑
    No specifics, no credibility

STRONG:
"Reduced our deployment time from 2 hours to 5 minutes"
- Jane Smith, CTO at TechCorp
  [Photo] [Company Logo]
                ↑
    Specific metric + credibility signals

STRONGEST:
"After switching to [Product], we saw:
 • 80% reduction in deployment time
 • 99.99% uptime
 • $50K/year saved in infrastructure"
- Jane Smith, CTO at TechCorp [Photo]
  Read the full case study →
                ↑
    Multiple metrics + full story available
```

---

## Audit Template

```markdown
## Trust Engineering Audit: [Page Name]

**URL**: [url]
**Date**: [date]

### Trust Signal Inventory

| Signal Type | Present | Placement | Effectiveness |
|-------------|---------|-----------|---------------|
| Compliance badges |   |   |   |
| Customer logos |   |   |   |
| Testimonials |   |   |   |
| Trust Center link |   |   |   |
| Security messaging |   |   |   |

### Compliance Badge Audit

- [ ] Badges placed near CTAs (not just footer)
- [ ] Badge selection appropriate for audience
- [ ] No badge overload (max 3-4 visible at once)
- [ ] Badges link to verification/Trust Center

### Privacy Messaging Audit

- [ ] "We don't" statements present
- [ ] Zero-knowledge claims (if applicable)
- [ ] Fear-free messaging near data collection
- [ ] Data flow transparency available

### Consent UX Audit

- [ ] Equal visual weight for accept/reject
- [ ] Granular cookie controls
- [ ] No pre-checked non-essential boxes
- [ ] Plain language (not legalese)
- [ ] Easy to change preferences later

### Trust Center Audit

- [ ] Trust Center exists and is findable
- [ ] Contains certifications with dates
- [ ] Includes data protection details
- [ ] Has security contact information
- [ ] Updated within last 12 months

### Findings

| Issue | Impact | Priority | Fix |
|-------|--------|----------|-----|
|       |        |          |     |

### Recommendations

1. [Priority 1]
2. [Priority 2]
3. [Priority 3]
```

---

## Guidelines

### DO
- Place compliance badges near conversion points
- Use specific "we don't" language for privacy claims
- Provide granular consent options with equal visual weight
- Link to verifiable Trust Center with dated certifications
- Match testimonials to page context (feature → relevant quote)

### DON'T
- Bury all trust signals in footer
- Use dark patterns in consent UI (pre-checked, hidden reject)
- Make vague "we take security seriously" claims
- Gate Trust Center behind sales conversations
- Display outdated certifications

---

## Related Skills and Agents

- **`ux_auditor` agent**: For overall page structure and component audit
- **`copy_writer` agent**: For trust-building headline copy
- **`auditing-page-velocity` skill**: For performance optimization
- **`ux-analyzing-friction` skill**: For identifying consent UX friction

**When to use which**: Use this skill for trust signals, privacy messaging, and consent UX. Use `ux_auditor` for overall page structure. Use `copy_writer` for copy optimization.
