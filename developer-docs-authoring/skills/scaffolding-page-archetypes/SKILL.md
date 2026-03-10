---
name: scaffolding-page-archetypes
version: 1.0.0
description: Scaffold structural templates for Diataxis-aligned page archetypes (Quickstart, How-To, API Reference, Webhooks, Troubleshooting) with concrete anatomy per type. Use when creating new documentation pages, choosing page structures, or enforcing archetype-specific anatomy. Triggered by: page archetype, quickstart template, how-to template, API reference layout, webhook docs, troubleshooting page, error dictionary, page scaffold, diataxis archetype, page anatomy, doc template, page structure.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Scaffolding Page Archetypes

## Purpose

Generate structural templates for five Diataxis-aligned documentation page types. Each archetype defines a concrete, opinionated anatomy — required sections, ordering, content slot types, and UX patterns — so that every page of a given type is structurally consistent across the documentation site.

This skill produces page skeletons with annotated content slots. It does not write final copy (handled by writing-focused skills) or assess Diataxis compliance generically (handled by the docs-designer agent).

## Page Archetypes

### 1. Quickstart

**Goal:** Zero to working code in under 5 minutes.

**Required Anatomy (in order):**

| Section | Content Slot | Notes |
|---------|-------------|-------|
| Header + Outcome | Title, one-sentence outcome promise | "By the end, you will have [concrete result]" |
| Prerequisites | Bulleted checklist | Account, API key, runtime version, installed tools |
| Step Blocks | Numbered sequential actions | Each step: instruction → code block → expected output |
| Verification | Success confirmation | "You should see [expected result]" with screenshot or output |
| Next Steps | 2-3 linked follow-on paths | Link to How-To guides, API Reference, or concepts |

**Rules:**
- Maximum 5 steps (split into multiple quickstarts if exceeding)
- Every code block must be copy-pasteable and runnable in a clean environment
- Show expected output after every code block that produces output
- Prerequisites must include exact version numbers, not "latest"
- No conceptual explanations — link to concept pages instead

### 2. How-To Guide

**Goal:** Accomplish a specific real-world task.

**Required Anatomy (in order):**

| Section | Content Slot | Notes |
|---------|-------------|-------|
| Action Header | Verb-first title | "Configure webhook retries", not "About webhook retries" |
| Context | 1-2 sentences | When and why you would do this |
| Solution Steps | Numbered actions | No setup — assume prerequisites from the quickstart |
| Edge Case Admonitions | Callout blocks | Warning/note boxes for gotchas at the relevant step |
| Verification | Confirmation of completion | How to confirm the task succeeded |

**Rules:**
- Title starts with an action verb (Configure, Deploy, Migrate, Enable)
- No prerequisites section — link to the quickstart for setup
- Each step is a single action, not a compound instruction
- Admonitions appear inline at the step where the edge case occurs, not grouped at the end
- No conceptual background — link to concept pages

### 3. API Reference

**Goal:** Complete, scannable reference for every endpoint.

**Required Anatomy (per endpoint):**

| Section | Content Slot | Notes |
|---------|-------------|-------|
| Endpoint Header | Method + path + short description | `POST /v1/messages` — Create a message |
| Authentication | Auth scheme indicator | Bearer token, API key, OAuth scope required |
| Parameter Table | Name, type, required, description, default | Grouped: path params → query params → body params |
| Request Example | Code block with realistic data | Multi-language tabs if supported |
| Response Example | JSON with all fields annotated | Include both success and error response shapes |
| Error Codes | Table of endpoint-specific errors | Code, message, resolution |

**Layout Pattern:** Three-column where viewport allows — navigation (left), content (center), code examples (right).

**Rules:**
- Every parameter documents its type, whether required/optional, default value, and constraints
- Request examples use realistic data, not `foo`/`bar`/`test`
- Response examples show complete shapes, not truncated
- Error codes include resolution steps, not just the error message
- Dynamic API key injection: authenticated users see their own keys pre-filled in examples

### 4. Webhooks / Events

**Goal:** Enable reliable event consumption.

**Required Anatomy (in order):**

| Section | Content Slot | Notes |
|---------|-------------|-------|
| Event Overview | Event name + trigger description | When this event fires and what it represents |
| Payload Schema | Typed field table | Distinguish thin events (ID + type) from snapshot events (full object) |
| Signature Verification | Step-by-step HMAC validation | Include HMAC-SHA256 code in multiple languages |
| Delivery Guarantees | At-least-once semantics, retry policy | Retry schedule, timeout (5-second response window) |
| Idempotency | Handling duplicate deliveries | Idempotency key location and deduplication strategy |
| Testing | Local testing instructions | CLI tools, webhook forwarding setup, sample payloads |

**Rules:**
- Clearly label thin vs snapshot payload strategy per event
- Signature verification code must be copy-pasteable and tested
- Document the 5-second response timeout explicitly
- Include retry schedule with exact intervals and max attempts
- Provide a table of all event types with one-line descriptions

### 5. Troubleshooting / Error Reference

**Goal:** Rapid symptom-to-resolution path.

**Required Anatomy (in order):**

| Section | Content Slot | Notes |
|---------|-------------|-------|
| Error Code Dictionary | Searchable code → message → cause → resolution table | One row per error code |
| Symptom Index | Common symptom descriptions | User-language symptoms mapped to error codes |
| Diagnosis Flow | Symptom → Diagnosis → Resolution | Decision tree or numbered steps per symptom |
| Common Patterns | Grouped resolution patterns | Auth errors, rate limits, validation errors, network errors |
| Escalation | When to contact support | What information to include in a support request |

**Rules:**
- Error codes are the primary lookup key, symptoms are the secondary path
- Every resolution includes a concrete action, not "check your configuration"
- Diagnosis flows use if/then logic, not paragraphs
- Group related errors by category (authentication, validation, rate limiting, server)
- Include the exact error message string users will see

## Archetype Selection Guide

| User Intent | Archetype | Signal Phrases |
|-------------|-----------|---------------|
| "How do I get started?" | Quickstart | first time, get started, hello world, initial setup |
| "How do I do X?" | How-To Guide | configure, set up, enable, migrate, deploy |
| "What does this endpoint do?" | API Reference | endpoint, parameter, request, response, API |
| "How do I receive events?" | Webhooks | webhook, event, callback, notification, subscription |
| "Why is this failing?" | Troubleshooting | error, failing, broken, not working, debug |

## Audit Checklist

For any documentation page, verify:

- [ ] Page matches exactly one archetype (no mixing)
- [ ] All required sections for that archetype are present and in order
- [ ] No sections from other archetypes are mixed in
- [ ] Title follows the archetype's naming convention
- [ ] Code blocks are copy-pasteable and use realistic data
- [ ] Cross-references link to the correct archetype (not inline explanations)
- [ ] Verification/confirmation section exists where required

## Output Format

### Page Scaffold: `{page title}`

**Archetype**: [Quickstart | How-To | API Reference | Webhooks | Troubleshooting]

**Content Slots**:
```markdown
[Complete markdown skeleton with annotated placeholders for each content slot]
```

**Checklist**:
- [x] Matches archetype anatomy
- [x] Sections in correct order
- [ ] Content slots filled (pending)

**Cross-References**:
- Links to: [related pages by archetype]
- Linked from: [pages that should reference this one]

## Guidelines

- **DO**: Enforce one archetype per page — split pages that mix types
- **DO**: Use the archetype selection guide when unsure which template to apply
- **DO**: Include verification sections in Quickstarts and How-To guides
- **DON'T**: Add conceptual explanations to Quickstarts or How-Tos — link to concept pages
- **DON'T**: Mix setup instructions into How-To guides — assume quickstart prerequisites
- **DON'T**: Use placeholder data in API Reference examples — use realistic values

## Related Components

- **docs-designer agent**: Generic Diataxis compliance checks — this skill provides archetype-specific anatomy
- **governing-taxonomy skill**: Taxonomy decisions inform how archetypes are organized in navigation
- **designing-navigation-systems skill**: Navigation structure must surface archetypes clearly
- **auditing-content-quality skill**: Quality rubric applied after scaffolding
- **designing-code-displays skill**: Code block patterns used within archetype templates
