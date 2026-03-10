---
name: writing-product-positioning
version: 1.0.0
user-invocable: false
description: Author product positioning documents using April Dunford's 5-component framework covering competitive alternatives, differentiated capabilities, customer value, target segments, and market category. Use when defining product positioning, analyzing competitive alternatives, mapping customer value, or framing market category. Triggered by: product positioning, competitive alternatives, dunford, positioning.md, market category, differentiated capabilities, target segment, customer value.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Product Positioning Authoring

## Purpose

Author `positioning.md` files using April Dunford's 5-component positioning framework. This document defines how a product is uniquely valuable to a specific set of customers — it is the upstream source of truth that informs marketing copy, sales decks, and brand positioning.

**Relationship to `writing-brand-positioning`**: The Brand Packet positioning in `marketing-site-authoring` is a downstream consumer. This skill produces the product-level positioning that Brand Packets draw from. Product positioning is about truth; brand positioning is about expression.

## Instructions

Follow these steps **in strict sequential order** — each component builds on the previous one. Do not skip ahead.

1. **Read existing product context**
   - Read `product.yaml` for canonical metadata
   - Read `overview.md` if it exists
   - Read any existing `positioning.md` to understand current state

2. **Component 1: Competitive Alternatives**
   - Ask: "What would customers do if this product didn't exist?"
   - Include direct competitors, indirect alternatives, and the status quo (manual processes, spreadsheets, "do nothing")
   - For each alternative, document its type and key limitation
   - Use the framework in `./references/dunford-framework.md`

3. **Component 2: Differentiated Capabilities**
   - Ask: "What can this product do that the alternatives identified in Component 1 cannot?"
   - Each capability must be traced back to a specific alternative's limitation
   - Include proof or evidence for each claim
   - **Sequence dependency**: These must address gaps in the alternatives from Component 1

4. **Component 3: Customer Value**
   - Ask: "What value do the differentiated capabilities from Component 2 deliver?"
   - Translate capabilities into outcomes customers care about
   - Include measurable outcomes where possible
   - **Sequence dependency**: Value must map directly to capabilities from Component 2

5. **Component 4: Target Customer Segments**
   - Ask: "Who cares most about the value identified in Component 3?"
   - Define primary and optional secondary segments with role, company type, trigger event, and must-have requirement
   - **Sequence dependency**: Segments are defined by who values Component 3's outcomes most

6. **Component 5: Market Category**
   - Ask: "What category makes this product's value obvious to the target segments from Component 4?"
   - Choose an existing, searchable category — not an invented one
   - Add a modifier that distinguishes this product's approach
   - Synthesize the positioning statement
   - **Sequence dependency**: Category framing serves the segments from Component 4

7. **Write the file**
   - Write to `products/<slug>/positioning.md`
   - Preserve the 5-component structure exactly

## Guidelines

- **DO**: Work through components in strict order — the sequence is the methodology
- **DO**: Start with competitive alternatives, not with your own product's features
- **DO**: Include the status quo ("do nothing" / "use spreadsheets") as a competitive alternative
- **DO**: Trace every claim through the chain: alternative limitation → capability → value → segment
- **DON'T**: Invent a market category — use an existing one with a differentiating modifier
- **DON'T**: List features as capabilities — capabilities are about what customers can achieve, not what buttons exist
- **DON'T**: Skip Component 1 and start with "our product is great because..." — that's features-out, not context-in

## Edge Cases

- **No direct competitors**: Focus on indirect alternatives and the status quo. Every product competes with "do nothing"
- **Multiple distinct use cases**: Create separate positioning for each if the target segments differ significantly
- **Positioning drift**: When updating, re-evaluate from Component 1 — don't just tweak Component 5

## Supporting Files

- [Dunford 5-Component Framework Reference](./references/dunford-framework.md)
