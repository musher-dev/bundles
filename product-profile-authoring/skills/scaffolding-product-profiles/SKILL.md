---
name: scaffolding-product-profiles
version: 1.0.0
user-invocable: true
description: Scaffold a new product profile directory with template files for metadata, overview, positioning, roadmap, and technical specifications. Use when creating a new product profile, initializing product documentation, or bootstrapping a product directory. Triggered by: new product profile, scaffold product, create product directory, initialize product docs, product template.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
argument-hint: <product-slug>
---

# Scaffold Product Profiles

## Purpose

Create a new product profile directory under `/products/<slug>/` with all required template files. This is the entry point for the product profile workflow — run this first, then use the individual writing skills to flesh out each file.

## Instructions

Follow these steps when this skill is activated:

1. **Parse the product slug**
   - Extract the slug from the user's input. If not provided, ask for a kebab-case product identifier (e.g., `acme-analytics`)
   - Validate: lowercase alphanumeric + hyphens only, no leading/trailing hyphens

2. **Check for existing profile**
   - Search for an existing `/products/<slug>/` directory
   - If it exists, warn the user and ask whether to overwrite or abort

3. **Create directory structure**
   ```
   products/<slug>/
   ├── product.yaml
   ├── overview.md
   ├── positioning.md
   ├── roadmap.md
   ├── technical.md
   └── assets/
       └── .gitkeep
   ```

4. **Generate `product.yaml`**
   - Use the annotated template from `./references/product-yaml-template.yaml`
   - Pre-fill `id` with the slug and `last_reviewed` with today's date
   - Leave all other fields as annotated placeholders

5. **Generate markdown files**
   - Use the templates from `./references/file-templates.md`
   - Each file gets its framework-specific structure with placeholder sections

6. **Output summary**
   - List all created files
   - Suggest which writing skill to use next (typically `writing-product-metadata` or `writing-product-overviews`)

## Guidelines

- **DO**: Use the exact templates from references — consistency across products matters
- **DO**: Create the `assets/` directory with `.gitkeep` for logos, diagrams, screenshots
- **DON'T**: Fill in substantive content — that's the job of the individual writing skills
- **DON'T**: Create additional files beyond the standard set without explicit user request

## Edge Cases

- **Monorepo with existing `/products/` directory**: Create profile alongside existing entries
- **No `/products/` directory exists**: Create it at the repository root
- **Slug conflicts with existing profile**: Always ask before overwriting

## Supporting Files

- [Product YAML Template](./references/product-yaml-template.yaml)
- [Markdown File Templates](./references/file-templates.md)
