---
name: assembling-pages
user-invocable: false
description: Assemble marketing page routes from section components in SvelteKit -- section ordering, SEO meta tags, inter-section spacing, anchor navigation, and pre-launch checklists. Use when composing sections into +page.svelte, adding SEO tags, setting up page routing, or preparing pages for launch. Triggered by: page assembly, compose page, page svelte, section ordering, SEO meta, og tags, twitter card, svelte head, anchor navigation, page route, inter-section spacing, page composition, landing page assembly, marketing page, pre-launch checklist, sitemap, canonical URL.
allowed-tools: Read, Write, Edit, Grep, Glob
---

# Assembling Marketing Pages

## Purpose

Compose scaffolded section components into complete SvelteKit page routes. This skill covers the page shell (`+page.svelte`), section import ordering to match the IA conviction arc, `<svelte:head>` SEO patterns, inter-section spacing ownership, anchor IDs for navigation, and a pre-launch checklist.

## Source Documents

- **IA (conviction arc)**: `/workspace/docs/marketing/information-architecture.md` -- section ordering, narrative flow
- **Wireframe Copy (meta text)**: `/workspace/docs/marketing/wireframe-copy.md` -- headline/description text for meta tags
- **Existing Page**: `apps/marketing-site/src/routes/+page.svelte` -- current page structure to reference
- **Design Tokens**: `/workspace/apps/marketing-site/src/app.css` -- spacing tokens for section gaps
- **SvelteKit Routing**: `apps/marketing-site/src/routes/` -- existing route structure

## Page Assembly Template

```svelte
<script lang="ts">
  // Import sections in IA conviction arc order
  import CategoryStake from '$lib/components/CategoryStake.svelte';
  import CredibilityBar from '$lib/components/CredibilityBar.svelte';
  import MechanismSection from '$lib/components/MechanismSection.svelte';
  import ValueGrid from '$lib/components/ValueGrid.svelte';
  import ProofBlock from '$lib/components/ProofBlock.svelte';
  import CommunityBlock from '$lib/components/CommunityBlock.svelte';
  import CtaBlock from '$lib/components/CtaBlock.svelte';
</script>

<svelte:head>
  <!-- Primary meta -->
  <title>Musher - The Control Plane for AI Coding Agents</title>
  <meta
    name="description"
    content="Bundle agent configs. Route events to deterministic pipelines. Stop babysitting your agents."
  />

  <!-- Open Graph -->
  <meta property="og:title" content="Musher - The Control Plane for AI Coding Agents" />
  <meta
    property="og:description"
    content="Bundle agent configs. Route events to deterministic pipelines. Stop babysitting your agents."
  />
  <meta property="og:image" content="/og-image.png" />
  <meta property="og:type" content="website" />
  <meta property="og:url" content="https://musher.dev" />
  <meta property="og:site_name" content="Musher" />

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content="Musher - The Control Plane for AI Coding Agents" />
  <meta
    name="twitter:description"
    content="Bundle agent configs. Route events to deterministic pipelines. Stop babysitting your agents."
  />
  <meta name="twitter:image" content="/og-image.png" />

  <!-- Canonical URL -->
  <link rel="canonical" href="https://musher.dev" />
</svelte:head>

<!-- Section 1: Category Stake (Hero) — Recognition -->
<CategoryStake />

<!-- Section 2: Credibility Signal Bar — Recognition/Belief bridge -->
<CredibilityBar />

<!-- Section 3: How the Control Plane Works (Mechanism) — Understanding -->
<MechanismSection />

<!-- Section 4: What You Control (Value Grid) — Desire -->
<ValueGrid />

<!-- Section 5: Built by Infrastructure Engineers (Proof Block) — Belief -->
<ProofBlock />

<!-- Section 6: Join the Build (Community Block) — Confidence -->
<CommunityBlock />

<!-- Section 7: Start Controlling Your Agents (CTA Block) — Action -->
<CtaBlock />
```

## Section Ordering Rules

The IA defines a conviction arc. Section order in `+page.svelte` **MUST** match the IA:

```
1. Category Stake (Hero)           → Recognition
2. Credibility Signal Bar          → Recognition/Belief bridge
3. How the Control Plane Works     → Understanding
4. What You Control (Value Grid)   → Desire
5. Built by Infrastructure Engineers → Belief
6. Join the Build (Community)      → Confidence
7. Start Controlling Your Agents   → Action
```

**NEVER reorder sections** without updating the IA document first. The conviction arc is a deliberate sequence -- earlier sections create the context for later ones.

## Import Ordering Convention

Imports should match the section order in the template:

```svelte
<script lang="ts">
  // Sections in IA order (top-to-bottom on page)
  import CategoryStake from '$lib/components/CategoryStake.svelte';
  import CredibilityBar from '$lib/components/CredibilityBar.svelte';
  import MechanismSection from '$lib/components/MechanismSection.svelte';
  import ValueGrid from '$lib/components/ValueGrid.svelte';
  import ProofBlock from '$lib/components/ProofBlock.svelte';
  import CommunityBlock from '$lib/components/CommunityBlock.svelte';
  import CtaBlock from '$lib/components/CtaBlock.svelte';

  // Shared layout components (header/footer) if not in +layout.svelte
  import MarketingHeader from '$lib/components/MarketingHeader.svelte';
  import MarketingFooter from '$lib/components/MarketingFooter.svelte';
</script>
```

## Inter-Section Spacing Ownership

**Each section owns its own vertical padding.** The page shell does NOT add spacing between sections.

| Section | Padding | Why |
|---------|---------|-----|
| Hero (Section 1) | `pt-24 pb-16 lg:pt-32 lg:pb-24` | Extra top padding for nav clearance; less bottom because social proof bar follows tightly |
| Social Proof Bar (Section 2) | `py-12` | Compact -- this is a signal bar, not a full section |
| Mechanism (Section 3) | `py-24 lg:py-32` | Full section spacing -- this is a heavy content section |
| Value Grid (Section 4) | `py-24` | Standard section spacing |
| Proof Block (Section 5) | `py-24` | Standard section spacing |
| Community (Section 6) | `py-24` | Standard section spacing |
| CTA Block (Section 7) | `py-32` | Extra padding -- final section needs breathing room |

**NEVER add `mb-*` or `mt-*` between section components in the page template.** If two sections feel too close or too far apart, adjust the section's own `py-*` value.

### Background Alternation for Visual Rhythm

Alternate section backgrounds to create visual separation:

| Section | Background |
|---------|-----------|
| 1: Hero | `bg-gradient-hero` or transparent (inherits `bg-surface-01`) |
| 2: Social Proof Bar | Transparent with `border-b border-border-subtle` |
| 3: Mechanism | `bg-surface-01` (default) |
| 4: Value Grid | `bg-surface-01` (default) |
| 5: Proof Block | `bg-surface-02` (elevated for contrast) |
| 6: Community | `bg-surface-01` (default) |
| 7: CTA | `bg-gradient-mesh-subtle` or `bg-surface-01` |

The existing page uses this pattern -- `FlowDiagram` uses `bg-surface-02` to distinguish itself.

## Anchor IDs for Navigation

Each section should have an `id` attribute for in-page navigation (e.g., from the header nav or CTAs):

```svelte
<section id="mechanism" class="py-24" aria-labelledby="mechanism-heading">
  <h2 id="mechanism-heading">...</h2>
</section>
```

| Section | Anchor ID | Use |
|---------|----------|-----|
| Hero | (none -- it's the page top) | N/A |
| Social Proof Bar | (none -- not a nav target) | N/A |
| Mechanism | `id="how-it-works"` | "How it works" nav link |
| Value Grid | `id="features"` | "Features" nav link |
| Proof Block | `id="about"` | "About" nav link |
| Community | `id="community"` | "Community" nav link |
| CTA | `id="install"` | "Install" CTA scrolls here |

Enable smooth scrolling via `html { scroll-behavior: smooth; }` (already set in `app.css`).

## `<svelte:head>` SEO Patterns

### Required Meta Tags

Every marketing page MUST have:

```svelte
<svelte:head>
  <!-- 1. Title (50-60 characters) -->
  <title>Musher - The Control Plane for AI Coding Agents</title>

  <!-- 2. Description (120-160 characters) -->
  <meta name="description" content="..." />

  <!-- 3. Open Graph (for social sharing) -->
  <meta property="og:title" content="..." />
  <meta property="og:description" content="..." />
  <meta property="og:image" content="/og-image.png" />
  <meta property="og:type" content="website" />
  <meta property="og:url" content="https://musher.dev" />
  <meta property="og:site_name" content="Musher" />

  <!-- 4. Twitter Card -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content="..." />
  <meta name="twitter:description" content="..." />
  <meta name="twitter:image" content="/og-image.png" />

  <!-- 5. Canonical URL -->
  <link rel="canonical" href="https://musher.dev" />
</svelte:head>
```

### Meta Tag Content Rules

- **Title**: Include product name + category. Match the hero headline. 50-60 characters.
- **Description**: Match the hero subheadline. 120-160 characters. Include primary keywords.
- **OG Image**: 1200x630px. Place in `static/og-image.png`. Show product name + tagline on dark background.
- **Canonical URL**: Always include to prevent duplicate content issues.

### Deriving Meta from Wireframe Copy

| Meta Field | Source |
|-----------|--------|
| `<title>` | `hero-headline` slot: "The control plane for AI coding agents." → "Musher - The Control Plane for AI Coding Agents" |
| `description` | `hero-subheadline` slot: "Bundle agent configs. Route events to deterministic pipelines. Stop babysitting your agents." |
| `og:title` | Same as `<title>` |
| `og:description` | Same as `description` |

## SvelteKit Routing for Future Pages

The marketing site uses file-based routing:

```
src/routes/
├── +page.svelte          ← Landing page (primary)
├── +layout.svelte        ← Shared layout (header/footer)
├── health/
│   └── +page.ts          ← Health check endpoint
├── pricing/              ← Future pricing page
│   └── +page.svelte
├── about/                ← Future about page
│   └── +page.svelte
└── docs/                 ← Redirects to docs.musher.dev
    └── +page.ts
```

For future pages, follow the same pattern:
1. Create a `+page.svelte` in the appropriate directory
2. Include full `<svelte:head>` with unique title/description
3. Import and compose section components
4. Set canonical URL to the page's full URL

## Header/Footer Integration

If `MarketingHeader` and `MarketingFooter` are in `+layout.svelte`, they wrap all pages automatically. If not, include them in each page:

```svelte
<MarketingHeader />

<!-- Page sections -->
<main>
  <CategoryStake />
  <!-- ... -->
  <CtaBlock />
</main>

<MarketingFooter />
```

The `<main>` element is recommended for accessibility -- it identifies the primary content region.

## Pre-Launch Checklist

Run through this checklist before shipping any marketing page:

### Content

- [ ] All content slots from the IA are filled (no placeholder text visible)
- [ ] Copy matches wireframe copy exactly (or has been approved via editorial review)
- [ ] No banned vocabulary (run `auditing-design-compliance` scan)
- [ ] All links point to valid destinations (docs, GitHub, Discord)
- [ ] Install command is correct and copyable

### SEO

- [ ] `<title>` is set (50-60 characters, includes product name)
- [ ] `<meta name="description">` is set (120-160 characters)
- [ ] Open Graph tags are complete (title, description, image, type, url, site_name)
- [ ] Twitter Card tags are complete (card, title, description, image)
- [ ] `<link rel="canonical">` is set
- [ ] OG image exists at the specified path (1200x630px)

### Accessibility

- [ ] All sections have `aria-labelledby` pointing to their heading
- [ ] All images have `alt` text
- [ ] All interactive elements have accessible names
- [ ] Color contrast meets WCAG AA (design tokens handle this -- verify custom elements)
- [ ] Focus order follows visual order (test with Tab key)
- [ ] Reduced motion handling works (verify animations skip with `prefers-reduced-motion`)

### Responsive

- [ ] Layout works at 375px (mobile)
- [ ] Layout works at 768px (tablet)
- [ ] Layout works at 1280px (desktop)
- [ ] No horizontal scrollbar at any width
- [ ] Touch targets are at least 44px on mobile
- [ ] Text is readable without zooming on mobile

### Performance

- [ ] No render-blocking scripts
- [ ] Images are optimized (WebP, lazy loading for below-fold)
- [ ] Fonts are preloaded
- [ ] No unnecessary third-party scripts
- [ ] Run `auditing-page-velocity` skill for full performance audit

### Design Compliance

- [ ] Run `auditing-design-compliance` skill for full token audit
- [ ] All sections use correct spacing tokens
- [ ] All colors use design token classes
- [ ] No Svelte 4 or React syntax
- [ ] Component classes used where applicable (.card, .btn, .badge, .bento-item)

## Anti-Patterns

| DO NOT | DO |
|--------|-----|
| Add margin between sections in the page shell | Let each section own its `py-*` padding |
| Reorder sections without updating the IA | Follow the IA conviction arc exactly |
| Put SEO meta directly in `app.html` | Use `<svelte:head>` per page (SvelteKit best practice) |
| Use generic meta descriptions | Derive from wireframe copy (specific, keyword-rich) |
| Skip canonical URL | Always include -- prevents duplicate content |
| Import sections in random order | Import in IA section order for readability |
| Use `<div>` for the page wrapper | Use `<main>` for accessibility |
| Add global styles in the page file | Use `app.css` or scoped `<style>` blocks |
| Duplicate header/footer in every page | Use `+layout.svelte` for shared chrome |

## Related Skills

- **`scaffolding-sections`** -- how to build each section component
- **`building-code-displays`** -- code block details for the Mechanism section
- **`animating-scroll-reveals`** -- animation timing across sections (no competing animations)
- **`auditing-design-compliance`** -- post-assembly QC pass
- **`auditing-page-velocity`** -- performance audit before launch
