---
name: generating-llm-indexes
version: 1.0.0
description: Generate machine-readable documentation indexes (llms.txt, llms-full.txt) for AI assistant consumption and RAG ingestion. Use when creating llms.txt files, optimizing docs for AI coding assistants, or structuring documentation for LLM consumption. Triggered by: llms.txt, llms-full.txt, machine-readable docs, AI assistant, RAG, LLM index, AI-readable, Cursor docs, Copilot docs, Claude Code docs, AI documentation, doc index.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Generating LLM Indexes

## Purpose

Generate and maintain machine-readable documentation indexes that enable AI coding assistants (Cursor, GitHub Copilot, Claude Code) and RAG pipelines to efficiently consume developer documentation. This skill covers the llms.txt specification, full-content dumps, content selection strategy, and update cadence.

This skill produces index files for AI consumption. It does not govern human-facing search (handled by designing-search-discovery) or assess documentation quality (handled by auditing-content-quality).

## Index File Types

### 1. llms.txt — Documentation Architecture Map

**Location:** `{docs-root}/llms.txt`

**Purpose:** Curated map of the documentation structure. AI assistants read this first to understand what exists and where to find it. Think of it as a sitemap optimized for LLM context windows.

**Format Specification:**

```text
# {Product Name} Documentation

> {One-sentence product description}

## Quickstarts
- [Getting Started]({url}): {one-line description}
- [Framework Quickstart]({url}): {one-line description}

## API Reference
- [Authentication]({url}): {one-line description}
- [Messages]({url}): {one-line description}
- [Models]({url}): {one-line description}

## Guides
- [Error Handling]({url}): {one-line description}
- [Webhooks]({url}): {one-line description}

## SDKs
- [Python SDK]({url}): {one-line description}
- [Node.js SDK]({url}): {one-line description}

## Optional
- [Changelog]({url})
- [Status Page]({url})
```

**Rules:**
- Header block: product name as H1, one-sentence description as blockquote
- Group pages by Diataxis archetype (Quickstarts, Guides, Reference, Explanation)
- Each entry: markdown link with URL + colon + one-line description
- Descriptions are factual summaries, not marketing copy
- Optional section for supplementary resources (changelog, status, blog)
- Maximum 100 entries — curate aggressively, link to section indexes for deep content
- URLs must be absolute (full domain), not relative paths

### 2. llms-full.txt — Complete Content Dump

**Location:** `{docs-root}/llms-full.txt`

**Purpose:** Comprehensive content dump for deep RAG ingestion. Contains the full text of all documentation pages concatenated with clear page boundaries.

**Format Specification:**

```text
# {Product Name} — Full Documentation

---
url: {absolute-url}
title: {page-title}
---

{Full markdown content of the page}

---
url: {absolute-url}
title: {page-title}
---

{Full markdown content of the page}
```

**Rules:**
- Page boundaries marked with YAML frontmatter blocks (url + title)
- Content is the rendered markdown, not raw source with template syntax
- Strip navigation elements, headers, footers, and sidebar content
- Preserve all code blocks, tables, and admonitions
- Order pages by: Quickstarts → Guides → API Reference → Webhooks → Troubleshooting → Concepts
- Include only published pages (not draft or deprecated)
- No file size target — completeness over brevity

### 3. Per-Section Index Files (Optional)

**Location:** `{docs-root}/llms-{section}.txt`

For large documentation sites, provide section-specific indexes:

| File | Content |
|------|---------|
| `llms-api.txt` | All API reference content |
| `llms-guides.txt` | All how-to guides |
| `llms-sdks.txt` | All SDK documentation |

**Rules:**
- Only create section files if llms-full.txt exceeds 200,000 tokens
- Reference section files from the main llms.txt under an "## Index Files" heading
- Same format as llms-full.txt within each section file

## Content Selection Strategy

### Include

| Content Type | Reason |
|-------------|--------|
| API reference (all endpoints) | Most queried by AI assistants |
| Quickstarts | Essential for "how do I get started" queries |
| How-to guides | Task-oriented, high-value for code generation |
| Error reference | Directly aids debugging assistance |
| SDK method signatures | Enables accurate code completion |
| Configuration reference | Prevents hallucinated config options |
| Webhook event schemas | Structured data AI assistants can reason about |

### Exclude

| Content Type | Reason |
|-------------|--------|
| Marketing pages | Not useful for coding assistance |
| Blog posts | Temporal, may contain outdated patterns |
| Changelog (in full dump) | Include in llms.txt as link only |
| Legal/compliance pages | Not relevant to coding queries |
| Community/forum content | Unvetted, may contain incorrect answers |
| Deprecated API versions | Could cause AI to suggest outdated patterns |

## Update Cadence

| Trigger | Action |
|---------|--------|
| Documentation deploy | Regenerate llms.txt and llms-full.txt |
| New API endpoint added | Add entry to llms.txt, regenerate llms-full.txt |
| Page deprecated | Remove from both files |
| Quarterly review | Audit llms.txt ordering and descriptions |

**Automation:** Build llms.txt generation into the docs CI/CD pipeline. The file should never be manually maintained after initial creation.

## Structuring Docs for AI Assistants

Beyond index files, optimize documentation content for AI consumption:

| Practice | Why |
|----------|-----|
| Use consistent heading hierarchy | AI assistants parse headings to understand structure |
| Include complete code examples | Partial snippets force AI to hallucinate the rest |
| Document all parameters with types | Enables accurate type-aware code generation |
| Use standard admonition syntax | AI can identify warnings, notes, and important callouts |
| Include explicit version numbers | Prevents version-confused responses |
| Provide JSON schemas for payloads | Structured schemas are more parseable than prose descriptions |

## Audit Checklist

For an existing llms.txt implementation, verify:

- [ ] llms.txt exists at the docs root URL
- [ ] Header block contains product name and description
- [ ] Pages are grouped by Diataxis archetype
- [ ] Every entry has an absolute URL and factual description
- [ ] No broken links (validate all URLs)
- [ ] No deprecated or draft pages included
- [ ] Entry count is under 100 (curated, not exhaustive)
- [ ] llms-full.txt exists and contains all published pages
- [ ] Page boundaries in llms-full.txt use consistent frontmatter format
- [ ] Generation is automated in CI/CD pipeline
- [ ] Files are updated on every documentation deploy

## Output Format

### LLM Index: `{product name}`

**Files Generated**:
- `llms.txt` — [entry count] entries across [section count] sections
- `llms-full.txt` — [page count] pages, [estimated token count] tokens

**Content Coverage**:
| Section | Pages Included | Pages Excluded | Reason |
|---------|---------------|----------------|--------|
| [section] | [count] | [count] | [reason for exclusions] |

**Validation**:
- [ ] All URLs resolve (HTTP 200)
- [ ] No deprecated content included
- [ ] Ordered by archetype priority
- [ ] Automated generation configured

## Guidelines

- **DO**: Regenerate index files on every documentation deploy
- **DO**: Curate llms.txt aggressively — quality over quantity
- **DO**: Use absolute URLs for all links
- **DO**: Include complete code examples in the full dump
- **DON'T**: Manually maintain llms.txt after initial generation setup
- **DON'T**: Include marketing copy, blog posts, or deprecated content
- **DON'T**: Truncate code blocks or tables in llms-full.txt

## Related Components

- **designing-search-discovery skill**: Human-facing search complements AI-facing indexes
- **governing-docs-lifecycle skill**: Page lifecycle states determine inclusion/exclusion
- **scaffolding-page-archetypes skill**: Archetype classification drives llms.txt section grouping
- **auditing-content-quality skill**: Quality gates applied before content enters index files
