---
name: writing-product-metadata
version: 1.0.0
user-invocable: false
description: Author and maintain product.yaml canonical metadata files with Schema.org-aligned fields including identity, audiences, capabilities, differentiators, integrations, and compliance posture. Use when creating product metadata, updating product YAML, maintaining product catalogs, or aligning product data with Schema.org. Triggered by: product.yaml, product metadata, schema.org product, canonical metadata, product catalog, product identity.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Product Metadata Authoring

## Purpose

Author and maintain `product.yaml` files that serve as the single source of truth for canonical product metadata. Every field is aligned with Schema.org Product/SoftwareApplication types, enabling downstream consumers (marketing sites, documentation portals, SEO pipelines) to extract structured data.

## Instructions

Follow these steps when authoring or updating a product's metadata:

1. **Locate the product profile**
   - Find the product directory under `/products/<slug>/`
   - Read the existing `product.yaml` if present
   - If no profile exists, suggest running the `scaffolding-product-profiles` skill first

2. **Gather product information**
   - Review any existing documentation, README files, or marketing materials
   - Ask the user for missing critical fields: `name`, `tagline`, `status`, `owner`, `audiences`, `problem_statement`

3. **Author metadata fields**
   - Work through each field systematically using the Schema.org mapping in `./references/schema-org-mapping.md`
   - Apply quality criteria per field:

   | Field | Quality Test |
   |---|---|
   | `tagline` | Under 120 characters? States value, not features? |
   | `audiences` | Each entry names a role + context? Pictureable? |
   | `problem_statement` | Includes trigger, impact, cost of inaction? |
   | `use_cases` | Verb-noun pairs? Describes outcomes, not features? |
   | `core_capabilities` | What the product does, not how it's built? |
   | `differentiators` | Each has a `capability` + `proof`? |
   | `non_goals` | Specific enough to reject feature requests against? |

4. **Set lifecycle status**
   - `discovery` — concept stage, no working product
   - `development` — actively being built
   - `beta` — available to limited users
   - `ga` — generally available
   - `sunset` — being phased out
   - `retired` — no longer available

5. **Cross-reference other profile files**
   - `positioning.md` → should inform `differentiators` and `audiences`
   - `roadmap.md` → should inform `roadmap_summary`
   - `technical.md` → should inform `integrations` and `security_compliance`

6. **Write the file**
   - Preserve YAML comments from the template for field documentation
   - Set `last_reviewed` to today's date in ISO 8601 format

## Guidelines

- **DO**: Keep `tagline` under 120 characters — it appears in listings and search results
- **DO**: Use verb-noun pairs for `use_cases` (e.g., "Automate schema reviews", not "Schema review automation")
- **DO**: Include proof for every differentiator — claims without evidence are marketing debt
- **DON'T**: Duplicate positioning.md content verbatim — metadata is structured, positioning is narrative
- **DON'T**: Leave `status` as `discovery` once the product has users

## Edge Cases

- **Multiple products sharing infrastructure**: Each gets its own `product.yaml` with distinct `id` values; shared components appear in both `integrations` lists
- **Pre-launch product**: Use `discovery` or `development` status; `proof_points` may be empty
- **Product sunset**: Set status to `sunset`, preserve all metadata for historical reference

## Supporting Files

- [Schema.org Property Mapping](./references/schema-org-mapping.md)
