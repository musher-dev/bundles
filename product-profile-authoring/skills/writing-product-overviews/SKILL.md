---
name: writing-product-overviews
version: 1.0.0
user-invocable: false
description: Author one-page product overviews using the Amazon Working Backwards PR/FAQ format with press release structure, external FAQ, and internal FAQ sections. Use when writing product overviews, creating PR/FAQ documents, drafting product announcements, or using the Working Backwards method. Triggered by: product overview, PR/FAQ, press release, working backwards, product announcement, one-pager, product summary, overview.md.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Product Overview Authoring

## Purpose

Author `overview.md` files using Amazon's Working Backwards PR/FAQ format. This document serves as the one-page product overview — the artifact a stakeholder reads to understand what the product is, who it's for, why it matters, and what the customer experience looks like.

## Instructions

Follow these steps when authoring a product overview:

1. **Read existing product context**
   - Read `product.yaml` for canonical metadata (name, tagline, audiences, problem statement)
   - Read `positioning.md` if it exists — positioning informs the press release framing
   - Read any existing `overview.md` to understand current state

2. **Draft the Press Release sections**
   Follow the structure in `./references/working-backwards-format.md`. Work through each section in order:

   | Section | Source | Quality Test |
   |---|---|---|
   | Heading | `product.yaml` name + category | Under 10 words? |
   | Subheading | `product.yaml` tagline / audiences | Names the customer and benefit? |
   | Summary | `product.yaml` problem_statement | Answers what, who, why in 2-3 sentences? |
   | Problem | Stakeholder input / research | Describes pain without mentioning the product? |
   | Solution | `product.yaml` core_capabilities | Focuses on mechanism, not feature list? |
   | Stakeholder Quote | Product owner input | Explains "why now"? |
   | Customer Experience | User journey research | Walks through first-use step by step? |
   | Customer Quote | Proof points / testimonials | Describes outcome, not feature? |
   | Call to Action | Current product state | One clear, specific next step? |

3. **Draft the External FAQ**
   - 3-5 questions a customer would ask after reading the press release
   - Must include: pricing/availability, comparison to alternatives, getting started
   - Answers should be direct — no hedging or "it depends"

4. **Draft the Internal FAQ**
   - 3-5 questions stakeholders or executives would ask
   - Must include: feasibility, resource requirements, risks
   - Answers should be honest about unknowns

5. **Quality review**
   - Apply the quality criteria from `./references/working-backwards-format.md`
   - Verify all sections are complete
   - Check that the document reads coherently end-to-end

6. **Write the file**
   - Write to `products/<slug>/overview.md`
   - Preserve the section structure exactly as templated

## Guidelines

- **DO**: Write in future tense for pre-launch products ("customers will...") and present tense for GA products
- **DO**: Make the problem section vivid — the reader should feel the pain before seeing the solution
- **DO**: Use the customer quote to demonstrate outcome, not to praise features
- **DON'T**: List features in the solution section — describe the mechanism and outcome
- **DON'T**: Use internal jargon in the press release sections (save that for Internal FAQ)
- **DON'T**: Skip the quotes — they force you to articulate the "why" from different perspectives

## Edge Cases

- **Pre-launch product with no customers**: Use representative customer quotes based on design partner feedback
- **Internal tool (no external customers)**: Adapt press release to internal audience; external FAQ becomes "user FAQ"
- **Product pivot**: Archive the old overview.md and start fresh — don't try to incrementally edit a fundamentally changed narrative

## Supporting Files

- [Working Backwards Format Reference](./references/working-backwards-format.md)
