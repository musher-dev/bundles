---
name: auditing-accessibility-compliance
version: 1.0.0
description: Audit developer documentation sites for WCAG AA accessibility compliance including keyboard navigation, screen reader compatibility, color contrast, ARIA attributes, and skip navigation. Use when reviewing docs accessibility, running WCAG audits, or checking keyboard navigation and screen reader support. Triggered by: accessibility audit, WCAG, keyboard navigation, screen reader, color contrast, ARIA, a11y, focus management, skip navigation, accessible docs, tab order, focus trapping.
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Auditing Accessibility Compliance

## Purpose

Evaluate developer documentation sites against WCAG 2.1 AA criteria specific to documentation contexts: keyboard navigation patterns, screen reader compatibility for code blocks and interactive elements, color contrast verification, and ARIA attribute requirements.

This skill is a read-only audit tool. It produces accessibility assessments but does not fix issues (handled by writing-focused skills and the docs-designer agent). It does not cover editorial quality (handled by auditing-content-quality) or quantitative analytics (handled by auditing-docs-metrics).

## WCAG AA Evaluation Criteria for Documentation

### Perceivable

#### 1.1 Text Alternatives

| Element | Requirement | Common Docs Violation |
|---------|------------|----------------------|
| Architecture diagrams | Descriptive alt text summarizing the diagram's meaning | `alt="diagram"` or missing alt entirely |
| Screenshots | Alt text describing the UI state shown | `alt="screenshot"` |
| Icons (functional) | Accessible name via `aria-label` or visually hidden text | Decorative icons without `aria-hidden="true"` |
| Code output images | Full text alternative of the output content | Terminal screenshot with no text fallback |

**Rule:** If an image conveys information not available in surrounding text, it needs a meaningful alt text. If it is purely decorative, it needs `aria-hidden="true"` or an empty `alt=""`.

#### 1.3 Adaptable Structure

| Requirement | Docs Context |
|------------|-------------|
| Heading hierarchy | Sequential levels (H1 > H2 > H3), no skips |
| Lists | Use semantic `<ul>`/`<ol>`, not styled paragraphs |
| Tables | Use `<th>` with `scope` attributes for data tables |
| Code blocks | Wrap in `<pre><code>` with `role="region"` and `aria-label` for long blocks |
| Landmarks | `<main>`, `<nav>`, `<aside>` for three-pane layout regions |

#### 1.4 Distinguishable

| Criterion | Minimum | Target | Test Method |
|-----------|---------|--------|-------------|
| Text contrast (body) | 4.5:1 | 7:1 | Check foreground/background ratio |
| Text contrast (large, ≥18pt) | 3:1 | 4.5:1 | Check foreground/background ratio |
| Non-text contrast (UI components) | 3:1 | 4.5:1 | Borders, icons, focus indicators |
| Syntax highlighting tokens | 4.5:1 per token color | — | Each token color against code block background |
| Focus indicator contrast | 3:1 against adjacent colors | — | Focus ring against page background and element background |

**Syntax Highlighting Audit:**

Check each token color in the syntax highlighting theme:

| Token Type | Example | Must Pass Against |
|------------|---------|-------------------|
| Keywords | `const`, `function`, `import` | Code block background |
| Strings | `"hello world"` | Code block background |
| Comments | `// this is a comment` | Code block background |
| Numbers | `42`, `3.14` | Code block background |
| Types | `string`, `number` | Code block background |
| Operators | `=`, `=>`, `+` | Code block background |

### Operable

#### 2.1 Keyboard Accessible

All functionality must be operable via keyboard alone.

**Documentation-Specific Keyboard Patterns:**

| Interaction | Expected Keys | Failure Mode |
|------------|---------------|--------------|
| Search (Cmd+K / Ctrl+K) | Opens search overlay, `Escape` closes | No keyboard shortcut, or overlay traps focus |
| Search results navigation | `Arrow Up`/`Down` to navigate, `Enter` to select | Mouse-only result selection |
| Sidebar navigation | `Tab`/`Shift+Tab` between items, `Enter` to expand/select | Click-only expand, no keyboard focus |
| Code block copy | `Tab` to copy button, `Enter`/`Space` to activate | Copy button unreachable via keyboard |
| Language tabs | `Arrow Left`/`Right` between tabs, `Enter` to select | Tab key cycles through all tabs instead of arrow keys |
| Accordion/collapsible | `Enter`/`Space` to toggle, `Tab` to move between | Click-only toggle |
| Table of Contents | `Tab` to links, `Enter` to navigate | No keyboard access |
| Version selector | `Arrow Up`/`Down` to select, `Enter` to confirm | Mouse-only dropdown |

**Focus Management Rules:**

| Scenario | Expected Behavior | Anti-Pattern |
|----------|-------------------|--------------|
| Search overlay opens | Focus moves to search input | Focus stays on trigger button |
| Search overlay closes | Focus returns to trigger element | Focus jumps to page top |
| Tab panel changes | Focus moves to panel content | Focus stays on tab |
| Accordion expands | Focus stays on trigger, content revealed | Focus jumps into content |
| Modal/dialog opens | Focus moves to first focusable element in dialog | Focus stays behind modal |
| Modal/dialog closes | Focus returns to trigger element | Focus lost |

#### 2.1.2 No Keyboard Trap

| Component | Trap Risk | Prevention |
|-----------|-----------|------------|
| Search overlay | High — modal pattern | `Escape` must close; focus must return |
| Interactive code playground | High — embedded editor | `Escape` must exit playground; document the shortcut |
| Embedded iframe (API explorer) | High — iframe focus | Provide skip link to exit iframe |
| Tabbed interface | Low | Use `aria-selected` and roving tabindex |

#### 2.4 Navigable

**Skip Navigation:**

| Requirement | Implementation |
|------------|---------------|
| Skip to main content | First focusable element on page; visible on focus; targets `<main>` |
| Skip to search | Second skip link for docs sites; targets search input |
| Skip to navigation | Optional third skip link for three-pane layout |
| Skip within page | "Skip to content" within sidebar-heavy layouts |

**Focus Order:**

For a standard three-pane documentation layout:

```
1. Skip links (visible on focus)
2. Top navigation bar
3. Search trigger (or search input)
4. Left sidebar navigation
5. Main content area
6. Right sidebar (table of contents)
7. Footer
```

**Page Title:**

| Pattern | Example | Anti-Pattern |
|---------|---------|--------------|
| `{Page Title} - {Section} - {Site}` | "Configure SSO - Guides - Acme Docs" | "Docs" |
| Unique per page | Each page has a distinct `<title>` | All pages share "Documentation" |

### Understandable

#### 3.1 Readable

| Requirement | Implementation |
|------------|---------------|
| Language attribute | `<html lang="en">` (or appropriate language) |
| Language changes | Inline code in another language marked with `lang` attribute |
| Abbreviations | `<abbr title="Application Programming Interface">API</abbr>` on first use |

#### 3.2 Predictable

| Pattern | Requirement |
|---------|------------|
| Navigation consistency | Sidebar order is identical across all pages in a section |
| Search behavior | Same search component on every page |
| External links | Marked with icon and `target="_blank"` with `rel="noopener"` |
| Code block behavior | Copy button in same position for all code blocks |

### Robust

#### 4.1 Compatible

| Requirement | Implementation |
|------------|---------------|
| Valid HTML | No duplicate IDs, proper nesting |
| ARIA roles | Interactive components use correct ARIA patterns |
| Name, Role, Value | All custom components expose accessible name and role |

**Documentation-Specific ARIA Patterns:**

| Component | ARIA Pattern |
|-----------|-------------|
| Search | `role="search"` on container, `role="combobox"` on input with `aria-expanded`, `aria-controls` |
| Sidebar | `role="navigation"` with `aria-label="Documentation"`, expandable sections use `aria-expanded` |
| Code block | `role="region"` with `aria-label="{language} code example"` for blocks >10 lines |
| Tabs (language) | `role="tablist"`, `role="tab"` with `aria-selected`, `role="tabpanel"` |
| Copy button | `aria-label="Copy code"`, status announcement via `aria-live="polite"` |
| Table of Contents | `role="navigation"` with `aria-label="On this page"` |
| Version selector | `role="listbox"` or `role="combobox"` with `aria-selected` |
| Breadcrumbs | `role="navigation"` with `aria-label="Breadcrumb"`, current page marked `aria-current="page"` |

## Screen Reader Compatibility

### Code Block Announcements

| Feature | Expected Behavior |
|---------|-------------------|
| Language label | Screen reader announces language before code content |
| Line numbers | Not announced as content (use `aria-hidden` or CSS) |
| Syntax highlighting | Token colors not announced; semantic meaning preserved in text |
| Copy button | "Copy code, button" announced; "Copied" status announced via live region |
| Diff annotations (`++`/`--`) | Added/removed lines announced with semantic meaning |

### Interactive Element Announcements

| Component | Expected Announcement |
|-----------|----------------------|
| Sidebar expand | "{Section name}, collapsed, button" / "{Section name}, expanded, button" |
| Tab selection | "{Language}, tab, 2 of 4, selected" |
| Search result | "{Result title}, {section}, link, 3 of 10" |
| Version selector | "{Version}, option, selected" |

## Audit Checklist

### Perceivable
- [ ] All images have meaningful alt text or are marked decorative
- [ ] Heading hierarchy is sequential with no skipped levels
- [ ] Data tables use `<th>` with `scope` attributes
- [ ] Body text contrast ratio ≥ 4.5:1
- [ ] Large text contrast ratio ≥ 3:1
- [ ] All syntax highlighting token colors pass 4.5:1 against background
- [ ] Focus indicators have ≥ 3:1 contrast
- [ ] UI component borders/icons have ≥ 3:1 contrast

### Operable
- [ ] Cmd+K / Ctrl+K opens search; Escape closes it
- [ ] Search results navigable with arrow keys
- [ ] Sidebar navigable with Tab and Enter
- [ ] Code copy buttons reachable and activatable via keyboard
- [ ] Language tabs use arrow key navigation pattern
- [ ] No keyboard traps in any interactive component
- [ ] Skip navigation link present and targets `<main>`
- [ ] Focus order follows logical reading order
- [ ] Focus returns to trigger when overlays close

### Understandable
- [ ] `<html lang>` attribute is set correctly
- [ ] Navigation order is consistent across pages
- [ ] External links are identifiable before activation
- [ ] Interactive element behavior is predictable

### Robust
- [ ] No duplicate IDs in rendered HTML
- [ ] All custom interactive components have ARIA roles
- [ ] Expandable elements use `aria-expanded`
- [ ] Search uses combobox or search ARIA pattern
- [ ] Code blocks >10 lines have `role="region"` with label
- [ ] Copy status announced via `aria-live` region

## Scoring Template

```markdown
## Accessibility Audit: {page or site URL}

**Scope**: {Single page | Section | Full site}
**Standard**: WCAG 2.1 AA
**Date**: {date}

### Summary

| Category | Pass | Fail | N/A | Score |
|----------|------|------|-----|-------|
| Perceivable | [n] | [n] | [n] | [%] |
| Operable | [n] | [n] | [n] | [%] |
| Understandable | [n] | [n] | [n] | [%] |
| Robust | [n] | [n] | [n] | [%] |
| **Total** | [n] | [n] | [n] | [%] |

**Verdict**: [PASS | FAIL] — {pass rate}% ({fail count} failures)

### Failures

| # | WCAG Criterion | Severity | Element | Issue | Remediation |
|---|---------------|----------|---------|-------|-------------|
| 1 | {e.g., 1.4.3} | Critical/High/Medium | {selector or description} | {what fails} | {how to fix} |

### Warnings

| # | WCAG Criterion | Element | Recommendation |
|---|---------------|---------|----------------|
| 1 | {criterion} | {element} | {improvement} |

### Keyboard Navigation Map

| Action | Keys | Result | Pass/Fail |
|--------|------|--------|-----------|
| Open search | Cmd+K | {observed} | Pass/Fail |
| Navigate results | Arrow Down | {observed} | Pass/Fail |
| ... | ... | ... | ... |
```

## Output Format

### Accessibility Audit: `{url or scope}`

**Summary**: [PASS/FAIL] — {pass rate}% compliance, {critical count} critical, {high count} high

**Critical Failures** (must fix before launch):
- {WCAG criterion}: {issue and remediation}

**High-Priority Issues** (fix within sprint):
- {WCAG criterion}: {issue and remediation}

**Keyboard Navigation**: [PASS/FAIL] — {issues found}

**Screen Reader Compatibility**: [PASS/FAIL] — {issues found}

**Contrast Audit**: [PASS/FAIL] — {failures in syntax highlighting or UI components}

## Guidelines

- **DO**: Test every interactive component with keyboard-only navigation
- **DO**: Check each syntax highlighting token color independently for contrast
- **DO**: Verify focus management when overlays open and close
- **DO**: Test with an actual screen reader for code block announcements
- **DON'T**: Assume a design system's built-in tokens pass contrast checks
- **DON'T**: Skip code block accessibility — developers spend most time in code blocks
- **DON'T**: Treat skip navigation as optional — it's critical for keyboard users on docs sites

## Related Components

- **auditing-content-quality skill**: Quality rubric includes Readability dimension; this skill provides deeper accessibility evaluation
- **designing-code-displays skill**: Code block patterns that this skill audits for accessibility
- **designing-navigation-systems skill**: Navigation patterns that this skill audits for keyboard accessibility
- **designing-search-discovery skill**: Search UX patterns that this skill audits for keyboard and screen reader compatibility
- **designing-typography-layout skill**: Typography standards that feed into contrast and readability checks
- **docs-designer agent**: Can remediate accessibility issues found by this audit
