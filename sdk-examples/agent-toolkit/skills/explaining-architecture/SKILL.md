---
name: explaining-architecture
description: >
  Generate clear architecture explanations for codebases and systems.
  Use when a developer needs to understand how components interact,
  where data flows, or how a system is structured at a high level.
---

# Explaining Architecture

When asked to explain a system's architecture, produce explanations that help developers build accurate mental models quickly.

## Approach

### 1. Identify the Audience

Tailor the explanation to the developer's context:

- New team member → start with the big picture, define domain terms
- Experienced contributor → focus on non-obvious interactions and edge cases
- External reviewer → emphasize boundaries, contracts, and security surfaces

### 2. Layer the Explanation

Structure from general to specific:

1. **One-sentence summary** — what the system does and for whom
2. **Component inventory** — the major pieces and their single-sentence responsibilities
3. **Interaction map** — how components communicate (sync/async, protocols, data formats)
4. **Data flow** — trace a representative request from entry to response
5. **Hard parts** — where complexity concentrates and why

### 3. Use Concrete References

Ground every claim in the code:

- Name the files, classes, or functions that implement each component
- Reference configuration that governs behavior
- Point to tests that demonstrate expected interactions
- Cite error handling paths, not just the happy path

## Formatting

- Use headers to separate architectural layers
- Use bullet lists for component inventories
- Use numbered steps for data flow traces
- Include file paths as `code references` so developers can navigate directly
- Avoid diagrams unless explicitly requested — text descriptions are more searchable and maintainable

## Anti-Patterns to Avoid

- Don't describe what the code *should* do — describe what it *does*
- Don't use generic architecture jargon without tying it to specific code
- Don't skip the error/failure paths — they reveal the real architecture
- Don't assume the README is accurate — verify against the implementation
