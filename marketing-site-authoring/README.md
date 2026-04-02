# Marketing Site Authoring

Run a multi-stage content pipeline from brand strategy through page implementation and design audit. This bundle orchestrates the full journey — competitive analysis, brand positioning, information architecture, wireframe copy, editorial review, component scaffolding, page assembly, animation, and compliance auditing — with specialized agents at each stage.

## Quick Start

Install via the [Musher CLI](https://github.com/musher-dev/musher-cli):

```sh
musher bundle install musher-dev/marketing-site-authoring
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Run the marketing content pipeline for our landing page, starting from our brand packet
```

## What's Inside

The bundle ships nine agents and twenty skills organized around a multi-stage production pipeline.

**Stage 1 — Brand Strategy.** Skills for competitive analysis, brand positioning, voice definition, visual identity, and trust signal sourcing feed into a validated Brand Packet. The `validating-brand-packets` skill gates quality before proceeding.

**Stage 2 — Information Architecture.** The **ia_producer** agent transforms the Brand Packet into a section-by-section IA document defining narrative flow, layout patterns, and conversion strategy.

**Stage 3 — Copy Production.** The **copy_writer** agent drafts marketing copy using cognitive science principles. The **copy_producer** fills every content slot defined in the IA. The **copy_reviewer** runs an anti-generic editorial pass, stripping banned vocabulary, filler, and hyperbolic claims.

**Stage 4 — Implementation.** The **section_builder** scaffolds Svelte 5 components from IA definitions. The **animation_engineer** adds scroll-triggered animations. The **page_assembler** composes sections into SvelteKit page routes with SEO meta tags.

**Stage 5 — Quality Assurance.** The **design_auditor** scans components for design system compliance. The **ux_auditor** evaluates information architecture, component presence, and trust signals.

Three orchestration skills (`orchestrating-content-pipeline`, `orchestrating-site-implementation`, `orchestrating-page-review`) coordinate agents across stages with approval gates between them.

## Usage Examples

**Run the full content pipeline**
```
Run the marketing content pipeline from brand packet through IA, wireframe copy, and editorial review
```
Orchestrates stages 1-3 sequentially with user approval gates between stages.

**Build and audit the site**
```
Implement the landing page — build section components, add animations, assemble the page, and audit for design compliance
```
Orchestrates stages 4-5: scaffolding, animation, assembly, and compliance audit.

**Review an existing page**
```
Run a comprehensive review of our landing page — check copy, UX, components, and trust signals
```
Delegates to specialized agents for deep analysis across copy quality, UX patterns, and design compliance.

**Analyze a competitor**
```
Deconstruct stripe.com's marketing site — hero patterns, copy strategy, trust signals, design system, and tech stack
```
Produces a structured 5-dimension competitive analysis.

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `analyzing-competitors` | Deconstruct competitor sites across 5 dimensions |
| `writing-brand-positioning` | Positioning statements, ICP profiles, differentiators |
| `writing-brand-voice` | Tonal boundaries, vibe/anti-vibe adjectives, banned vocabulary |
| `writing-visual-identity` | Color tokens, typography system, spatial rules |
| `writing-trust-signals` | Metrics, logos, badges, quotes, community signals |
| `validating-brand-packets` | 11-field completeness check, 6-dimension Polish rubric |
| `writing-information-architecture` | Section-by-section IA with narrative flow and conversion strategy |
| `writing-wireframe-copy` | Exact copy for every content slot from the IA document |
| `reviewing-copy` | Anti-generic editorial pass — banned vocabulary, filler, hyperbole |
| `scaffolding-sections` | IA definitions to Svelte 5 components with design tokens |
| `building-code-displays` | Marketing-grade code blocks with syntax highlighting and terminal chrome |
| `animating-scroll-reveals` | CSS `.reveal` classes or GSAP ScrollTrigger patterns |
| `assembling-pages` | SvelteKit page routes with SEO meta, section ordering, anchor nav |
| `generating-design-rules` | Design system enforcement rules for the harness |
| `auditing-design-compliance` | Automated grep scans for token, typography, and spacing violations |
| `auditing-page-velocity` | Core Web Vitals compliance and perceived performance |
| `auditing-trust-engineering` | Privacy-first trust architecture and consent UX |
| `orchestrating-content-pipeline` | Brand → IA → copy → editorial pipeline orchestration |
| `orchestrating-site-implementation` | Build → animate → assemble → audit pipeline orchestration |
| `orchestrating-page-review` | Comprehensive review across copy, UX, components, and trust |

### Agents

| Agent | Role |
|-------|------|
| `ia_producer` | Transforms Brand Packet into structural IA blueprint |
| `copy_writer` | Drafts marketing copy using cognitive science principles |
| `copy_producer` | Fills content slots from IA with 90%-final wireframe copy |
| `copy_reviewer` | Anti-generic editorial pass stripping banned vocabulary and filler |
| `section_builder` | Builds Svelte 5 section components from IA and copy |
| `animation_engineer` | Adds scroll-triggered animations to section components |
| `page_assembler` | Composes sections into SvelteKit page routes with SEO |
| `design_auditor` | Scans components for design system compliance violations |
| `ux_auditor` | Audits page UX, information architecture, and visual system |

</details>

---

**Domain:** marketing-site · **Technologies:** Harness-agnostic · **Aliases:** marketing, marketing-site, landing-page
