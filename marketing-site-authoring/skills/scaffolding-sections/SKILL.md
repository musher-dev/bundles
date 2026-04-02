---
name: scaffolding-sections
user-invocable: false
description: Scaffold marketing page sections from IA definitions into Svelte 5 components. Use when building section components from the Information Architecture, translating archetypes to markup, mapping content slots to elements, or creating section scaffolds. Triggered by: scaffold section, build section, IA to component, section component, hero section, mechanism section, value grid, proof block, community block, CTA block, social proof bar, content slot, archetype component, layout pattern, section scaffold, marketing component.
allowed-tools: Read, Write, Edit, Grep, Glob
---

# Scaffolding Marketing Sections

## Purpose

Translate Information Architecture section definitions into Svelte 5 component scaffolds. Each IA section has an archetype, layout pattern, and content slots -- this skill maps those to concrete markup, props, and design token usage.

## Source Documents

Read these before scaffolding any section:

- **IA**: `/workspace/docs/marketing/information-architecture.md` -- section definitions, archetypes, layout patterns, content slots
- **Wireframe Copy**: `/workspace/docs/marketing/wireframe-copy.md` -- exact copy for every content slot
- **Brand Packet**: `/workspace/docs/marketing/brand-packet.md` -- voice, color tokens, typography, spacing
- **Design Tokens**: `/workspace/apps/marketing-site/src/app.css` -- `@theme` block, component classes, animation classes
- **Design Rule**: `/workspace/.claude/rules/design-system.md` -- enforcement guardrails

## Existing Components (Reference Implementations)

Study these before writing new sections -- they demonstrate the project's patterns:

| Component | Path | Demonstrates |
|-----------|------|-------------|
| `Hero.svelte` | `apps/marketing-site/src/lib/components/Hero.svelte` | `hero-with-demo` layout, gradient backgrounds, CTA buttons, badge, asymmetric grid |
| `TrustBar.svelte` | `apps/marketing-site/src/lib/components/TrustBar.svelte` | `metric-plus-logos` layout, metric display, integration logos, badges |
| `BentoGrid.svelte` | `apps/marketing-site/src/lib/components/BentoGrid.svelte` | `bento-grid` layout, `.bento-item`/`.bento-large`/`.bento-tall` classes, GSAP stagger |
| `FlowDiagram.svelte` | `apps/marketing-site/src/lib/components/FlowDiagram.svelte` | Complex GSAP ScrollTrigger, node-based layouts, mobile/desktop variants |
| `CodeSnippet.svelte` | `apps/marketing-site/src/lib/components/CodeSnippet.svelte` | Code block with copy button, `$props()` pattern |
| `InteractiveTerminal.svelte` | `apps/marketing-site/src/lib/components/InteractiveTerminal.svelte` | Terminal chrome, GSAP timeline, `$state()` + `$derived()` runes |

## Archetype-to-Component Mapping

### Hero (`hero-with-demo`)

```
Structure:
  <section> (full-width, gradient background)
    container (max-w-[1200px] mx-auto)
      grid (lg:grid-cols-12)
        left-col (lg:col-span-5) -- text content
          badge (optional)
          h1 (text-display-lg, font-bold, tracking-tight)
          p.subheadline (text-body-large, text-text-secondary)
          CTA row (flex gap-4)
            a.btn.btn-primary.btn-lg
            a.btn.btn-secondary
        right-col (lg:col-span-7) -- visual/demo
          <InteractiveTerminal /> or hero visual
      trust-mechanic (optional logo row below)

Section padding: pt-24 pb-16 lg:pt-32 lg:pb-24
Side padding: px-6 lg:px-16
Background: bg-gradient-hero or subtle radial gradients
```

### Social Proof Bar (`metric-plus-logos`)

```
Structure:
  <section> (border-b border-border-subtle)
    container
      metrics-row (flex justify-center gap-12)
        metric (text-center)
          value (text-display, text-signal-action)
          label (text-label, text-text-tertiary, uppercase, tracking-wide)
      logos-row (flex items-center justify-center gap-6)
        "Works with" label (text-label, text-text-tertiary)
        logo pills (bg-surface-02, border, rounded-full, px-3 py-1.5)
      badges-row (flex justify-center gap-4)
        badge (bg-surface-02/50, border, rounded-full, text-label)

Section padding: py-12
No background change -- inherits page bg
```

### Mechanism (`code-then-output`)

```
Structure:
  <section>
    container
      text-center header
        h2 (text-display, font-bold, tracking-tight)
        p.subheadline (text-body-large, text-text-secondary)
      code-block-pair (repeated for bundle + pipeline)
        grid (lg:grid-cols-2, gap-6)
          left-panel -- YAML config
            code-chrome (header bar + language label)
            pre > code (font-mono, text-data)
          right-panel -- CLI output
            terminal-chrome (header bar + "output" label)
            pre > code (font-mono, text-data)
      body-paragraph (text-body-large, text-text-secondary, max-w-3xl mx-auto)

Section padding: py-24 lg:py-32
See: building-code-displays skill for code block details
```

### Value Grid (`bento-grid`)

```
Structure:
  <section>
    container
      text-center header
        h2 (text-display, font-bold, tracking-tight)
        p.subheadline (text-body-large, text-text-secondary)
      div.bento-grid
        card-1.bento-item.bento-large -- primary differentiator
          icon (w-10 h-10, bg-signal-action/10, text-signal-action)
          h3 (text-heading, text-text-primary)
          p (text-body, text-text-secondary)
          optional inline code reference
        card-2.bento-item.bento-large -- primary differentiator
          (same structure)
        card-3.bento-item -- supporting differentiator
          (same structure, no bento-large)
        card-4.bento-item -- supporting differentiator
          (same structure, no bento-large)

Section padding: py-24
Card classes: use .bento-item from app.css (NOT custom card styles)
Card hover: built into .bento-item (border-signal-action/50 on hover)
```

### Proof Block (`metric-showcase`)

```
Structure:
  <section>
    container
      text-center header
        h2 (text-display, font-bold, tracking-tight)
        p.subheadline (text-body-large, text-text-secondary)
      founder-credential (card or glass panel)
        avatar/icon + name + role
        p (text-body-large, text-text-secondary)
      metrics-grid (grid md:grid-cols-3 gap-6)
        metric-card (card or bg-surface-02 rounded-lg p-6)
          value (text-display, text-signal-action or text-text-primary)
          label (text-body, text-text-secondary)

Section padding: py-24
Background: bg-surface-02 or subtle gradient for contrast
```

### Community Block (`centered-statement`)

```
Structure:
  <section>
    container (text-center, max-w-2xl mx-auto)
      h2 (text-display, font-bold, tracking-tight)
      p.subheadline (text-body-large, text-text-secondary)
      links-row (flex justify-center gap-4)
        a.btn.btn-primary -- GitHub link
        a.btn.btn-secondary -- Discord link
      optional metrics (hidden until populated)

Section padding: py-24
Minimal decoration -- centered-statement should breathe
```

### CTA Block (`centered-statement`)

```
Structure:
  <section>
    container (text-center, max-w-2xl mx-auto)
      h2 (text-display-lg or text-display, font-bold, tracking-tight)
      p.subheadline (text-body-large, text-text-secondary)
      install-command (code block, copyable, centered)
        pre > code (font-mono, text-body-large)
        CopyButton
      secondary-links (flex justify-center gap-4)
        a (text-signal-action, hover:underline)
      friction-reducers (flex justify-center gap-4)
        span.badge or inline pill per reducer

Section padding: py-32 (extra breathing room for final CTA)
Background: optional subtle gradient-mesh-subtle
```

## Content Slot-to-Markup Translation Table

| Slot Type | HTML Element | Tailwind Classes |
|-----------|-------------|-----------------|
| `headline` | `<h1>` | `text-display-lg font-bold tracking-tight text-text-primary` |
| `section-headline` | `<h2>` | `text-display font-bold tracking-tight text-text-primary` |
| `subheadline` | `<p>` | `text-body-large text-text-secondary` |
| `section-subheadline` | `<p>` | `text-body-large text-text-secondary max-w-2xl mx-auto` |
| `body-paragraph` | `<p>` | `text-body-large text-text-secondary max-w-3xl mx-auto` |
| `cta-primary` | `<a>` | `btn btn-primary btn-lg` |
| `cta-secondary` | `<a>` | `btn btn-secondary` |
| `feature-card-title` | `<h3>` | `text-heading font-semibold text-text-primary` |
| `feature-card-description` | `<p>` | `text-body text-text-secondary` |
| `metric-value` | `<span>` or `<div>` | `text-display text-signal-action tabular-nums` |
| `metric-label` | `<span>` or `<div>` | `text-label text-text-tertiary uppercase tracking-wide` |
| `friction-reducer` | `<span>` | `badge-muted` or inline pill (bg-surface-03, border, rounded-full, text-label) |
| `community-link` | `<a>` | `btn btn-primary` (primary) or `btn btn-secondary` (secondary) |
| `community-metric` | `<div>` | Same as metric-value/metric-label pair (hidden until populated) |
| `logo-set` | `<div>` | Flex row with logo pills: `flex items-center gap-4` |
| `code-then-output` | Composite | See `building-code-displays` skill |
| `hero-visual` | Component | `<InteractiveTerminal />` or custom terminal component |

## Component File Pattern

```svelte
<script lang="ts">
  // 1. Imports (Svelte lifecycle, GSAP if animated, child components)
  import { onMount } from 'svelte';

  // 2. Props (if the section takes external data)
  // Most sections are self-contained with hardcoded IA copy

  // 3. State (if interactive)
  let sectionRef: HTMLElement;
</script>

<!-- Section with semantic ID for anchor navigation -->
<section
  id="section-anchor-id"
  bind:this={sectionRef}
  class="py-24 px-6 lg:px-16"
  aria-labelledby="section-heading-id"
>
  <div class="mx-auto max-w-[1200px]">
    <!-- Section header -->
    <div class="text-center mb-12">
      <h2
        id="section-heading-id"
        class="text-display font-bold tracking-tight text-text-primary mb-4"
      >
        {/* IA section-headline slot */}
      </h2>
      <p class="text-body-large text-text-secondary max-w-2xl mx-auto">
        {/* IA section-subheadline slot */}
      </p>
    </div>

    <!-- Section body (archetype-specific layout) -->
    <!-- ... -->
  </div>
</section>
```

**File naming**: `SectionName.svelte` in PascalCase. Place in `apps/marketing-site/src/lib/components/`.

**Examples**: `CategoryStake.svelte`, `CredibilityBar.svelte`, `MechanismSection.svelte`, `ValueGrid.svelte`, `ProofBlock.svelte`, `CommunityBlock.svelte`, `CtaBlock.svelte`.

## Props Interface Pattern

Most marketing sections are self-contained (copy comes from wireframe, not props). Use `$props()` only when a component needs external data:

```svelte
<script lang="ts">
  // Props for reusable sub-components
  interface Props {
    title: string;
    description: string;
    icon?: string;
  }

  let { title, description, icon }: Props = $props();
</script>
```

For top-level sections, hardcode the IA copy directly -- this makes the component a self-documenting artifact that matches the IA exactly.

## Trust Mechanic Integration

Each IA section specifies a trust mechanic. Integrate them as follows:

| Section | Trust Mechanic | Implementation |
|---------|---------------|----------------|
| 1: Hero | Integration logos | Logo pills row below terminal: "Works with Claude Code, Codex, OpenCode" |
| 2: Social Proof Bar | Metric + logos + badge | Metric-value/label pairs + logo pills + badge pills |
| 3: Mechanism | Architecture proof | The code blocks themselves are the proof -- show real YAML and CLI output |
| 4: Value Grid | Per-card micro-proof | Inline `code` references within card descriptions (e.g., `musher bundle apply`) |
| 5: Proof Block | Founder credential + metrics | Credential paragraph + metric-showcase grid |
| 6: Community | GitHub + Discord links | Link buttons (inspect the repo yourself) |
| 7: CTA | Friction reducers | Badge pills below install command |

## Responsive Patterns

Every section must work at 375px, 768px, and 1280px:

- **Grid columns**: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3` (or `lg:grid-cols-12` for hero)
- **Side padding**: `px-6 md:px-10 lg:px-16`
- **Section padding**: `py-24` standard, `py-32` for CTA block
- **Text alignment**: `text-center lg:text-left` for hero; `text-center` for all others
- **Stacking**: Grid items stack to single column on mobile

## Anti-Patterns

| DO NOT | DO |
|--------|-----|
| Create custom card styles | Use `.card`, `.glass`, `.bento-item` from `app.css` |
| Use `className` | Use `class` (this is Svelte, not React) |
| Use `on:click` | Use `onclick` (Svelte 5 event syntax) |
| Use `export let` for props | Use `let { prop } = $props()` |
| Use `$:` for reactivity | Use `$state()`, `$derived()`, `$effect()` |
| Add arbitrary spacing (`p-[37px]`) | Use the spacing scale from `@theme` |
| Use raw hex colors | Use design token classes (`bg-surface-02`, `text-text-primary`) |
| Use Tailwind default palette | Use project tokens (`bg-signal-action`, not `bg-blue-500`) |
| Create separate CSS file | Use Tailwind utilities or `app.css` component classes |
| Import from React/Vue | This is SvelteKit 2.x + Svelte 5 |

## Workflow

1. **Read the IA section definition** -- archetype, layout pattern, content slots, trust mechanic
2. **Read the wireframe copy** -- exact text for every content slot
3. **Check existing components** -- does a component already cover this archetype?
4. **Scaffold the component** using the archetype-to-component mapping above
5. **Fill content slots** using the slot-to-markup translation table
6. **Add trust mechanic** per the integration table
7. **Add anchor `id`** for in-page navigation (e.g., `id="mechanism"`)
8. **Add `aria-labelledby`** pointing to the section heading
9. **Test responsive** at 375px, 768px, 1280px widths (mentally or via dev tools)

## Related Skills

- **`building-code-displays`** -- detailed guidance for `code-then-output` layout pattern (Mechanism section)
- **`animating-scroll-reveals`** -- when and how to add scroll animations to scaffolded sections
- **`assembling-pages`** -- how to compose sections into the final `+page.svelte`
- **`auditing-design-compliance`** -- post-implementation QC pass
