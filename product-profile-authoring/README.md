# Product Profile Authoring

Create canonical product profiles — metadata, positioning, roadmaps, and technical specs in structured formats. This bundle scaffolds a product profile directory and populates it with machine-readable metadata (Schema.org-aligned YAML), strategic positioning (April Dunford's framework), product overviews (Amazon Working Backwards PR/FAQ), temporal roadmaps (Now/Next/Later), and technical specifications.

## Quick Start

Install via the [Musher CLI](https://github.com/musher-dev/musher-cli):

```sh
musher bundle install musher-dev/product-profile-authoring
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Scaffold a product profile directory for our new API platform
```

## What's Inside

The bundle ships six skills that together produce a complete product profile directory. There are no agents — each skill is invoked directly.

The `scaffolding-product-profiles` skill creates the directory structure with template files. From there, each writing skill fills a specific document:

- **product.yaml** — Machine-readable canonical metadata aligned with Schema.org fields: identity, audiences, capabilities, differentiators, integrations, and compliance posture.
- **overview.md** — One-page product summary using the Amazon Working Backwards PR/FAQ format with press release structure, external FAQ, and internal FAQ.
- **positioning.md** — Strategic positioning using April Dunford's 5-component framework: competitive alternatives, differentiated capabilities, customer value, target segments, and market category.
- **roadmap.md** — Temporal planning using the Now/Next/Later format with three time-horizon buckets, dependencies, assumptions, and open questions.
- **technical.md** — Technical specification covering architecture overview, integrations, data flows, system limits, compliance posture, and non-functional requirements.

## Usage Examples

**Scaffold a new product profile**
```
Scaffold a product profile directory for our deployment platform with all template files
```
Creates the directory structure with starter templates for all five documents.

**Write product positioning**
```
Write positioning for our API gateway — identify competitive alternatives, differentiated capabilities, and target segments
```
Produces a structured positioning document using April Dunford's framework.

**Create a product roadmap**
```
Create a Now/Next/Later roadmap for our developer tools platform
```
Organizes priorities into three time-horizon buckets with dependencies and open questions.

**Write a PR/FAQ overview**
```
Write a Working Backwards PR/FAQ for our new observability product
```
Produces a press release, external FAQ (customer questions), and internal FAQ (stakeholder questions).

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `scaffolding-product-profiles` | Scaffold directory structure with template files |
| `writing-product-metadata` | Schema.org-aligned product.yaml — identity, audiences, capabilities, compliance |
| `writing-product-overviews` | Amazon Working Backwards PR/FAQ format — press release, external FAQ, internal FAQ |
| `writing-product-positioning` | April Dunford's 5-component framework — alternatives, capabilities, value, segments, category |
| `writing-product-roadmaps` | Now/Next/Later temporal planning — three horizons, dependencies, open questions |
| `writing-technical-specifications` | Architecture, integrations, data flows, system limits, compliance, non-functional requirements |

</details>

---

**Domain:** product-profile · **Technologies:** Harness-agnostic · **Aliases:** product-docs, profile-authoring, product-profiles
