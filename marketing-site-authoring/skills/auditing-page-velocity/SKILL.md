---
name: auditing-page-velocity
user-invocable: false
description: Audit marketing pages for Core Web Vitals compliance and perceived performance. Use when checking page speed, optimizing load times, or troubleshooting performance issues. Triggered by: page speed, Core Web Vitals, LCP, INP, CLS, performance audit, lazy loading, skeleton screens, TTFB, lighthouse, page velocity, slow loading, web performance.
allowed-tools: Read, Grep, Glob
---

# Page Velocity Audit

## Purpose

Provide a systematic framework for auditing marketing page performance against Core Web Vitals benchmarks and perceived performance patterns. Marketing pages that load slowly lose conversions - every 100ms of latency costs ~1% of revenue.

## Core Web Vitals Framework

```
┌─────────────────────────────────────────────────────────────────┐
│                    2025 PERFORMANCE BENCHMARKS                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │     LCP     │  │     INP     │  │     CLS     │              │
│  │  Largest    │  │ Interaction │  │ Cumulative  │              │
│  │ Contentful  │  │  to Next    │  │   Layout    │              │
│  │   Paint     │  │   Paint     │  │    Shift    │              │
│  │             │  │             │  │             │              │
│  │  ≤ 2.5s     │  │  ≤ 200ms    │  │   < 0.1     │              │
│  │   GOOD      │  │    GOOD     │  │    GOOD     │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                  │
│  ┌─────────────┐                                                │
│  │    TTFB     │  Time to First Byte ≤ 200ms                    │
│  │  ≤ 200ms    │  (Server response time)                        │
│  └─────────────┘                                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Metric Definitions

| Metric | What It Measures | Target | Critical Threshold |
|--------|------------------|--------|-------------------|
| **LCP** | Time until largest visible element renders | ≤ 2.5s | > 4.0s |
| **INP** | Responsiveness to user interactions | ≤ 200ms | > 500ms |
| **CLS** | Visual stability during load | < 0.1 | > 0.25 |
| **TTFB** | Server response time | ≤ 200ms | > 600ms |

---

## Industry Benchmarks

Reference these when setting performance targets:

| Company | LCP | Notes |
|---------|-----|-------|
| **Stripe** | ~1.0s | Heavy use of skeleton screens |
| **Vercel** | ~0.8s | Edge-rendered, optimized images |
| **Tableau** | ~1.1s | Static marketing pages |
| **Trello** | ~0.8s | Aggressive lazy loading |
| **Linear** | ~1.2s | Animation-heavy but optimized |
| **ClickUp** | ~8.4s | Anti-pattern: slow marketing page |

**Target**: Match or beat Stripe/Vercel benchmarks for developer-focused products.

---

## Audit Checklist

### 1. Image Optimization

```
┌─────────────────────────────────────────────────────────────────┐
│                    IMAGE AUDIT CHECKLIST                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Format Selection:                                               │
│  ├── [ ] Hero images: WebP with JPEG fallback                   │
│  ├── [ ] Icons/logos: SVG (vector) or WebP                      │
│  ├── [ ] Photos: WebP or AVIF with quality 75-85                │
│  └── [ ] Avoid: PNG for photos, uncompressed images             │
│                                                                  │
│  Loading Strategy:                                               │
│  ├── [ ] Above-fold images: eager loading, preload hints        │
│  ├── [ ] Below-fold images: loading="lazy"                      │
│  ├── [ ] Background images: loaded via CSS after critical       │
│  └── [ ] Responsive images: srcset with appropriate breakpoints │
│                                                                  │
│  Sizing:                                                         │
│  ├── [ ] Explicit width/height to prevent CLS                   │
│  ├── [ ] aspect-ratio CSS as fallback                           │
│  └── [ ] Images not larger than display size (2x max for retina)│
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 2. Script Optimization

```
Priority Order:
1. Critical CSS (inline in <head>)
2. Critical JS (deferred, not blocking)
3. Third-party scripts (delayed until interaction)

Anti-patterns to detect:
├── Blocking scripts in <head>
├── render-blocking CSS imports
├── Analytics loading before content
├── Chat widgets loading on page load
└── Multiple jQuery versions
```

**Script Loading Strategies**:

| Script Type | Strategy | Implementation |
|-------------|----------|----------------|
| Critical JS | Defer | `<script defer>` |
| Analytics | Delay | Load after `load` event or user interaction |
| Chat widgets | Lazy | Load on scroll or interaction |
| Third-party embeds | Facade | Show static preview, load on click |

### 3. Rendering Optimization

**Skeleton Screens** - Show structure before content:

```html
<!-- Good: Skeleton that matches final layout -->
<div class="hero-skeleton">
  <div class="skeleton-headline animate-pulse"></div>
  <div class="skeleton-subhead animate-pulse"></div>
  <div class="skeleton-cta animate-pulse"></div>
</div>

<!-- Replace with real content when loaded -->
```

**Critical CSS Inlining**:
- Inline above-fold CSS in `<head>` (typically < 14KB)
- Load remaining CSS asynchronously
- Use tools like Critical or Critters

### 4. Font Optimization

```
Font Loading Checklist:
├── [ ] font-display: swap (prevent FOIT)
├── [ ] Preload critical fonts
├── [ ] Subset fonts to used characters
├── [ ] Self-host fonts (avoid Google Fonts latency)
├── [ ] Limit font families (max 2-3)
└── [ ] Use variable fonts where possible
```

---

## Common Anti-Patterns

### Anti-Pattern 1: Unoptimized Hero Images

```
BAD:
<img src="hero.png" />  <!-- 2.4MB PNG -->

GOOD:
<picture>
  <source srcset="hero.avif" type="image/avif">
  <source srcset="hero.webp" type="image/webp">
  <img src="hero.jpg"
       width="1200"
       height="600"
       loading="eager"
       fetchpriority="high"
       alt="Hero description">
</picture>
```

### Anti-Pattern 2: Render-Blocking Scripts

```
BAD:
<head>
  <script src="analytics.js"></script>
  <script src="chat-widget.js"></script>
  <script src="app.js"></script>
</head>

GOOD:
<head>
  <script defer src="app.js"></script>
</head>
<body>
  <!-- Content first -->
  <script>
    // Load analytics after page load
    window.addEventListener('load', () => {
      setTimeout(() => loadAnalytics(), 2000);
    });
  </script>
</body>
```

### Anti-Pattern 3: Layout Shift from Images

```
BAD:
<img src="feature.jpg" />  <!-- No dimensions = CLS -->

GOOD:
<img src="feature.jpg"
     width="800"
     height="450"
     style="aspect-ratio: 16/9"
     loading="lazy" />
```

### Anti-Pattern 4: Web Font FOIT

```
BAD:
@font-face {
  font-family: 'Custom';
  src: url('custom.woff2');
  /* No font-display = invisible text */
}

GOOD:
@font-face {
  font-family: 'Custom';
  src: url('custom.woff2');
  font-display: swap;
}

/* Preload in <head> */
<link rel="preload" href="custom.woff2" as="font" crossorigin>
```

---

## Performance Audit Template

Use this template when auditing a marketing page:

```markdown
## Page Velocity Audit: [Page Name]

**URL**: [url]
**Date**: [date]
**Tool Used**: Lighthouse / PageSpeed Insights / WebPageTest

### Core Web Vitals

| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| LCP    |       | ≤ 2.5s |        |
| INP    |       | ≤ 200ms|        |
| CLS    |       | < 0.1  |        |
| TTFB   |       | ≤ 200ms|        |

### Image Audit

- [ ] Hero image optimized (WebP/AVIF)
- [ ] Below-fold images lazy loaded
- [ ] All images have explicit dimensions
- [ ] No images larger than 200KB

### Script Audit

- [ ] No render-blocking scripts
- [ ] Third-party scripts deferred
- [ ] Analytics loads after content
- [ ] Total JS bundle < 200KB (gzipped)

### Font Audit

- [ ] font-display: swap on all fonts
- [ ] Critical fonts preloaded
- [ ] Max 3 font families

### Findings

| Issue | Impact | Fix |
|-------|--------|-----|
|       |        |     |

### Recommendations

1. [Priority 1 fix]
2. [Priority 2 fix]
3. [Priority 3 fix]
```

---

## Measurement Tools

| Tool | Purpose | When to Use |
|------|---------|-------------|
| **Lighthouse** | Overall audit | Initial assessment |
| **PageSpeed Insights** | Field + lab data | Production pages |
| **WebPageTest** | Detailed waterfall | Debugging specific issues |
| **Chrome DevTools** | Real-time debugging | Development |
| **CrUX** | Real user data | Long-term monitoring |

### Quick Lighthouse Command

```bash
# Run Lighthouse from CLI
npx lighthouse https://example.com \
  --output=json \
  --output-path=./lighthouse-report.json \
  --only-categories=performance
```

---

## Guidelines

### DO
- Measure before and after every optimization
- Test on throttled connections (3G, slow 4G)
- Monitor Core Web Vitals in production (CrUX)
- Prioritize LCP for marketing pages (first impression)
- Use CDN for all static assets

### DON'T
- Optimize without measuring impact
- Sacrifice visual quality for marginal speed gains
- Ignore mobile performance (often 60%+ of traffic)
- Add third-party scripts without performance budget
- Trust synthetic tests alone (use real user metrics)

---

## Related Skills and Agents

- **`ux_auditor` agent**: For component structure and IA audits
- **`copy_writer` agent**: For headline and copy optimization
- **`auditing-trust-engineering` skill**: For compliance badge and privacy UX
- **`design-engineer` agent**: For INP optimization in interactive components

**When to use which**: Use this skill for performance metrics. Use `ux_auditor` for layout and component audits. Use `design-engineer` for interaction responsiveness in complex UIs.
