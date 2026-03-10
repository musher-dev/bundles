---
name: animating-scroll-reveals
user-invocable: false
description: Add scroll-triggered animations to marketing sections using CSS .reveal classes or GSAP ScrollTrigger. Use when adding entrance animations, staggered reveals, scroll-triggered effects, or deciding between CSS and GSAP animation approaches. Triggered by: scroll animation, scroll reveal, entrance animation, stagger animation, GSAP ScrollTrigger, CSS reveal, reveal-visible, scroll effect, animate on scroll, fade in on scroll, scroll trigger, motion, animation decision, reduced motion, cleanup scroll triggers.
allowed-tools: Read, Write, Edit, Grep, Glob
---

# Animating Scroll Reveals

## Purpose

Document the animation patterns available in the Musher marketing site and provide a decision tree for choosing the right approach. Three tiers exist: CSS-only `.reveal` classes, GSAP ScrollTrigger for groups, and GSAP timelines for stateful sequences.

## Source Files

- **GSAP Config**: `/workspace/apps/marketing-site/src/lib/gsap-config.ts` -- singleton GSAP + ScrollTrigger registration, `prefersReducedMotion()`, `cleanupScrollTriggers()`
- **CSS Reveal Classes**: `/workspace/apps/marketing-site/src/app.css` -- `.reveal`, `.reveal-left`, `.reveal-right`, `.reveal-scale`, `.reveal-visible`
- **BentoGrid Example**: `apps/marketing-site/src/lib/components/BentoGrid.svelte` -- GSAP stagger with ScrollTrigger
- **FlowDiagram Example**: `apps/marketing-site/src/lib/components/FlowDiagram.svelte` -- GSAP timeline with ScrollTrigger, hover effects
- **InteractiveTerminal Example**: `apps/marketing-site/src/lib/components/InteractiveTerminal.svelte` -- GSAP timeline (user-triggered, not scroll)

## Animation Decision Tree

```
Is this a single element fading/sliding into view?
├── YES → Use CSS .reveal classes (Tier 1)
│
└── NO → Are multiple elements animating in a coordinated group?
    ├── YES → Is the stagger simple (same animation, sequential timing)?
    │   ├── YES → Use GSAP ScrollTrigger with stagger (Tier 2)
    │   └── NO → Use GSAP timeline with ScrollTrigger (Tier 3)
    │
    └── NO → Is it triggered by user interaction (click, hover)?
        ├── YES → Use GSAP timeline without ScrollTrigger (Tier 3)
        └── NO → Use CSS .reveal (Tier 1)
```

**Rule of thumb**: Start with the simplest tier that works. CSS `.reveal` covers 60% of marketing animation needs.

## Tier 1: CSS `.reveal` Classes

Defined in `app.css`. No JavaScript required. Animation triggers by adding `.reveal-visible` class via IntersectionObserver or GSAP ScrollTrigger.

### Available Classes

| Class | Effect | Transition |
|-------|--------|-----------|
| `.reveal` | Fade up (translateY 20px) | 0.6s ease-out |
| `.reveal-left` | Fade from left (translateX -20px) | 0.6s ease-out |
| `.reveal-right` | Fade from right (translateX 20px) | 0.6s ease-out |
| `.reveal-scale` | Fade + scale up (scale 0.95) | 0.6s ease-out |

### Triggering with `.reveal-visible`

Add `.reveal-visible` when the element enters the viewport. Two approaches:

**Approach A: Svelte `use:` action (lightweight):**

```svelte
<script lang="ts">
  function revealOnScroll(node: HTMLElement) {
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            entry.target.classList.add('reveal-visible');
          }
        });
      },
      { threshold: 0.1 }
    );

    observer.observe(node);

    return {
      destroy() {
        observer.disconnect();
      },
    };
  }
</script>

<div class="reveal" use:revealOnScroll>
  <h2>Section headline</h2>
</div>
```

**Approach B: GSAP ScrollTrigger toggleClass (when GSAP is already imported):**

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import { ScrollTrigger, cleanupScrollTriggers } from '$lib/gsap-config';

  let el: HTMLElement;

  onMount(() => {
    ScrollTrigger.create({
      trigger: el,
      start: 'top 80%',
      onEnter: () => el.classList.add('reveal-visible'),
      onLeaveBack: () => el.classList.remove('reveal-visible'),
    });

    return cleanupScrollTriggers;
  });
</script>

<div bind:this={el} class="reveal">
  <h2>Section headline</h2>
</div>
```

### Reduced Motion

CSS `.reveal` classes already have a `@media (prefers-reduced-motion: reduce)` block in `app.css` that sets `opacity: 1; transform: none; transition: none`. No additional handling needed.

## Tier 2: GSAP ScrollTrigger with Stagger

For groups of elements that animate in sequence (e.g., bento grid items, feature cards).

### Pattern (from BentoGrid.svelte)

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import {
    gsap,
    prefersReducedMotion as checkReducedMotion,
    cleanupScrollTriggers,
  } from '$lib/gsap-config';

  let containerRef: HTMLElement;

  onMount(() => {
    // ALWAYS check reduced motion first
    if (checkReducedMotion()) {
      return;
    }

    const items = containerRef.querySelectorAll('.bento-item');

    // Set initial state (hidden)
    gsap.set(items, { opacity: 0, y: 30 });

    // Animate on scroll
    gsap.to(items, {
      scrollTrigger: {
        trigger: containerRef,
        start: 'top 80%',
        toggleActions: 'play none none reverse',
      },
      opacity: 1,
      y: 0,
      stagger: 0.1,
      duration: 0.6,
      ease: 'power2.out',
    });

    // ALWAYS return cleanup function
    return cleanupScrollTriggers;
  });
</script>

<div bind:this={containerRef} class="bento-grid">
  <!-- items with .bento-item class -->
</div>
```

### Key Settings

| Setting | Value | Why |
|---------|-------|-----|
| `start` | `'top 80%'` | Triggers when top of element hits 80% viewport height (visible but not centered) |
| `toggleActions` | `'play none none reverse'` | Plays on enter, reverses on leave-back (scroll up past it) |
| `stagger` | `0.1` (100ms) | Subtle cascading effect -- not too fast, not too slow |
| `duration` | `0.6` (600ms) | Matches CSS `.reveal` timing |
| `ease` | `'power2.out'` | Decelerating ease -- fast start, gentle stop |

### Stagger Variations

| Effect | `stagger` value | Use case |
|--------|----------------|----------|
| Quick cascade | `0.08` | Small items (badges, pills) |
| Standard | `0.1` | Cards, grid items |
| Deliberate | `0.15` | Large sections, metrics |
| Grid stagger | `{ each: 0.1, grid: 'auto', from: 'start' }` | 2D grid layouts |

## Tier 3: GSAP Timeline with ScrollTrigger

For complex multi-step animations where elements animate with different properties or timing.

### Pattern (from FlowDiagram.svelte)

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import {
    gsap,
    prefersReducedMotion as checkReducedMotion,
    cleanupScrollTriggers,
  } from '$lib/gsap-config';

  let sectionRef: HTMLElement;

  onMount(() => {
    if (checkReducedMotion()) {
      return;
    }

    const nodes = sectionRef.querySelectorAll('.flow-node');
    const connectors = sectionRef.querySelectorAll('.flow-connector');

    // Set initial states
    gsap.set(nodes, { opacity: 0, y: 20, scale: 0.9 });
    gsap.set(connectors, { scaleX: 0, transformOrigin: 'left center' });

    // Create timeline
    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: sectionRef,
        start: 'top 70%',
        toggleActions: 'play none none reverse',
      },
    });

    // Step 1: Nodes appear
    tl.to(nodes, {
      opacity: 1,
      y: 0,
      scale: 1,
      stagger: 0.12,
      duration: 0.5,
      ease: 'back.out(1.4)',
    });

    // Step 2: Connectors draw (overlapping with step 1)
    tl.to(connectors, {
      scaleX: 1,
      stagger: 0.08,
      duration: 0.4,
      ease: 'power2.out',
    }, '-=0.3'); // Start 0.3s before previous step ends

    return cleanupScrollTriggers;
  });
</script>
```

## GSAP Import Rules

**ALWAYS import from `$lib/gsap-config.ts`**, never directly from `'gsap'` or `'gsap/ScrollTrigger'`:

```typescript
// CORRECT
import { gsap, ScrollTrigger, prefersReducedMotion, cleanupScrollTriggers } from '$lib/gsap-config';

// WRONG -- bypasses singleton registration
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
```

The config file (`gsap-config.ts`) handles:
- Plugin registration (once, in browser only)
- `prefersReducedMotion()` utility
- `cleanupScrollTriggers()` utility

## Reduced Motion: Mandatory Pattern

Every component using GSAP **MUST** check reduced motion and bail early:

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import { gsap, prefersReducedMotion as checkReducedMotion, cleanupScrollTriggers } from '$lib/gsap-config';

  onMount(() => {
    // MUST be the first check
    if (checkReducedMotion()) {
      return; // No animations, no cleanup needed
    }

    // ... animation code ...

    return cleanupScrollTriggers;
  });
</script>
```

When reduced motion is active:
- CSS `.reveal` classes automatically show content (handled by `app.css` media query)
- GSAP animations are skipped entirely
- Content is immediately visible at its final state

## ScrollTrigger Cleanup: Mandatory Pattern

Every `onMount` that creates ScrollTrigger instances **MUST** return the cleanup function:

```svelte
onMount(() => {
  if (checkReducedMotion()) return;

  // ... create ScrollTrigger instances ...

  // MUST return cleanup to prevent memory leaks on navigation
  return cleanupScrollTriggers;
});
```

`cleanupScrollTriggers()` calls `ScrollTrigger.getAll().forEach(st => st.kill())`. This is safe to call even if no triggers were created.

## Performance Guardrails

### Only Animate Compositor Properties

**ALLOWED** (GPU-accelerated, no layout recalculation):
- `opacity`
- `transform` (translate, scale, rotate)

**FORBIDDEN** (triggers layout/paint, janky on scroll):
- `width`, `height`
- `top`, `left`, `right`, `bottom`
- `margin`, `padding`
- `border-width`
- `font-size`
- `box-shadow` (except for hover states via CSS transitions)

### Duration Guidelines

| Context | Duration | Why |
|---------|----------|-----|
| Scroll reveal (single element) | 0.6s | Perceptible but not slow |
| Staggered group reveal | 0.5-0.6s per item, 0.1s stagger | Total animation < 2s |
| Timeline sequence | 0.3-0.5s per step | Keep total < 2.5s |
| Hover states | 0.15s (use CSS `duration-fast`) | Must feel instant |

### Easing

| Context | Easing | GSAP equivalent |
|---------|--------|----------------|
| Elements entering view | Decelerate | `power2.out` |
| Elements with overshoot | Back ease | `back.out(1.4)` |
| Continuous animations | Sine | `sine.inOut` |
| CSS transitions | Cockpit timing | `cubic-bezier(0.2, 0, 0.38, 0.9)` |

## Section Animation Recommendations

Based on the IA section archetypes:

| Section | Animation Tier | Implementation |
|---------|---------------|---------------|
| 1: Hero | CSS `.animate-fade-in` + `.animate-slide-up` | Already uses entrance animations (no scroll trigger -- it's above the fold) |
| 2: Social Proof Bar | Tier 1: CSS `.reveal` | Simple fade-up for the entire bar |
| 3: Mechanism | Tier 2: GSAP stagger | Code blocks appear sequentially (left panel, then right panel) |
| 4: Value Grid | Tier 2: GSAP stagger | Bento items stagger in (same pattern as current `BentoGrid.svelte`) |
| 5: Proof Block | Tier 2: GSAP stagger | Metrics cards stagger in; founder credential fades up |
| 6: Community | Tier 1: CSS `.reveal` | Simple fade-up (minimal section, minimal animation) |
| 7: CTA Block | Tier 1: CSS `.reveal-scale` | Install command scales up for emphasis |

## Anti-Patterns

| DO NOT | DO |
|--------|-----|
| Import `gsap` directly from `'gsap'` | Import from `$lib/gsap-config` |
| Skip `prefersReducedMotion()` check | Always check first in `onMount` |
| Skip `cleanupScrollTriggers` return | Always return cleanup from `onMount` |
| Animate `width`, `height`, `margin` | Animate only `opacity` and `transform` |
| Use `gsap.from()` for scroll reveals | Use `gsap.set()` + `gsap.to()` (avoids FOUC) |
| Set duration > 1s for scroll reveals | Keep reveals under 0.6s for snappiness |
| Animate every element independently | Group elements and use stagger |
| Use `IntersectionObserver` when GSAP is already imported | Use GSAP `ScrollTrigger` for consistency |
| Add animation before content/layout is right | Get the static layout working first, then animate |

## Related Skills

- **`scaffolding-sections`** -- build the static section first, then add animation
- **`assembling-pages`** -- page-level animation considerations (no competing animations)
- **`auditing-design-compliance`** -- verify reduced motion handling
