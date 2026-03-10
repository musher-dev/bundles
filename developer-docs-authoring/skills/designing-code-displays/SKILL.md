---
name: designing-code-displays
version: 1.0.0
description: Design code block UX patterns including syntax highlighting, copy buttons, language tab synchronization, interactive playgrounds, dynamic API key injection, and CI-verified examples. Use when building code displays, improving code sample UX, or designing interactive documentation features. Triggered by: code block, syntax highlighting, copy button, language tabs, code playground, interactive examples, API key injection, code display, code UX, live code, code testing, runnable examples.
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
---

# Designing Code Displays

## Purpose

Define UX patterns and design specifications for code blocks in developer documentation. This skill covers the full spectrum from static syntax-highlighted snippets to interactive playgrounds, ensuring code examples are accessible, consistent, and functional across the documentation site.

This skill prescribes design patterns and implementation requirements. It does not score code sample quality (handled by analyzing-docs-benchmarks) or scaffold page structures containing code blocks (handled by scaffolding-page-archetypes).

## Code Display Components

### 1. Syntax Highlighting

**Requirements:**
- High-contrast color scheme meeting WCAG AA (4.5:1 ratio for normal text)
- Light and dark mode variants respecting OS preference (`prefers-color-scheme`)
- Consistent token coloring across all supported languages
- Line numbers optional, enabled by default for blocks exceeding 5 lines

**Token Color Categories:**

| Token Type | Purpose | Example |
|-----------|---------|---------|
| Keywords | Language reserved words | `const`, `function`, `import` |
| Strings | String literals | `"hello world"` |
| Numbers | Numeric literals | `42`, `3.14` |
| Comments | Code comments | `// explanation` |
| Functions | Function/method names | `createUser()` |
| Types | Type annotations | `string`, `number` |
| Operators | Operators and punctuation | `=`, `=>`, `{}` |

**Rules:**
- Never rely on color alone to convey meaning — use font weight or style as secondary signal
- Test highlighting against colorblind simulation (protanopia, deuteranopia)
- Support a minimum of: JavaScript/TypeScript, Python, Go, Ruby, cURL, JSON, YAML, Bash

### 2. Copy-to-Clipboard

**Requirements:**
- Persistent copy button visible on every code block (not hover-only)
- Button position: top-right corner of the code block
- Visual feedback: icon changes to checkmark for 2 seconds after copy
- Copies raw code only — no line numbers, no prompt characters (`$`, `>`)
- Keyboard accessible: focusable and activatable via Enter/Space

**Rules:**
- Strip prompt prefixes (`$ `, `> `) from shell examples when copying
- Strip line numbers if displayed
- Preserve indentation and whitespace exactly
- Multi-block copy: if a code group has tabs, copy only the active tab content

### 3. Language Tab Synchronization

**Requirements:**
- Language selector tabs above code blocks when multiple languages are available
- Selection persists across all code blocks on the current page
- Selection persists across pages via session storage (key: `docs-preferred-language`)
- Default language determined by: stored preference → browser Accept-Language → first tab

**Tab Behavior:**

| State | Behavior |
|-------|----------|
| First visit, no preference | Show first tab, no stored preference |
| User selects language | Store preference, update all blocks on page |
| Navigate to new page | Read preference, pre-select matching tabs |
| Block missing preferred language | Show first available tab, do not clear preference |
| Preference cleared | Revert to default behavior |

**Rules:**
- Tab labels use official language names: "JavaScript", not "JS" (abbreviation in tab tooltip is acceptable)
- Tab order is consistent across all blocks on the site
- If a code block has only one language, show no tabs (not a single disabled tab)

### 4. Interactive Playgrounds

**Requirements:**
- Editable code area with syntax highlighting
- "Run" button that executes against a staging/sandbox backend
- Response panel showing formatted output (JSON pretty-printed, headers collapsible)
- Parameter controls for adjustable inputs (sliders, dropdowns for enum values)
- Reset button to restore original example code

**Implementation Tiers:**

| Tier | Description | Use Case |
|------|-------------|----------|
| Static | Syntax highlighted, copy button only | All code blocks (baseline) |
| Editable | Edit code, no execution | Schema examples, configuration |
| Runnable | Edit + execute against sandbox | API endpoints, SDK methods |
| Full playground | Multi-file, persistent state | Tutorials, complex integrations |

**Rules:**
- Playground backend must be isolated — never hit production APIs
- Rate limit playground requests (suggest: 10 requests/minute per session)
- Show loading state during execution, timeout after 10 seconds
- Display errors in the response panel, not as alerts/modals
- Playground availability does not gate page access — static fallback always present

### 5. Dynamic API Key Injection

**Requirements:**
- Authenticated users see their own API keys pre-filled in code examples
- Unauthenticated users see a placeholder: `YOUR_API_KEY`
- Key injection is client-side (no server rendering of secrets)
- Visual indicator showing which key is injected (e.g., "Using key: sk-...abc")
- One-click toggle to reveal/hide the full key

**Rules:**
- Never render full API keys in page source or server responses
- Injection targets: Authorization headers, SDK initialization, environment variable examples
- If the user has multiple keys, prefer the most recently created test/development key
- Include a "Get your API key" link for unauthenticated users, pointing to the dashboard
- Key injection must not break copy-to-clipboard — copied code includes the actual key

### 6. Code Example Testing

**Requirements:**
- CI pipeline that extracts and executes code blocks from documentation source
- Test categories: syntax validation, execution (where runnable), output verification
- Failure blocks the docs build — broken examples never ship

**Testing Matrix:**

| Code Type | Validation | Execution | Output Check |
|-----------|-----------|-----------|--------------|
| Shell commands | Syntax parse | Run in container | Exit code 0 |
| API requests (cURL) | Syntax parse | Run against staging | Status code match |
| SDK code | Compile/interpret | Run against staging | Output match |
| Configuration | Schema validate | N/A | N/A |
| Pseudocode | Skip | Skip | Skip |

**Rules:**
- Mark non-executable blocks explicitly (e.g., ```` ```pseudo ```` or ```` ```text ````)
- Test against the same API version documented on the page
- Flaky tests get quarantined, not deleted — fix the flakiness
- Test results visible in CI dashboard with per-page breakdown

## Audit Checklist

For any documentation page with code blocks, verify:

- [ ] Every code block has syntax highlighting with correct language identifier
- [ ] Every code block has a visible copy button (not hover-only)
- [ ] Multi-language blocks use synchronized tabs with session persistence
- [ ] API examples use realistic data, not placeholder values (except API keys)
- [ ] Authenticated users see their keys injected in examples
- [ ] Interactive playgrounds have static fallbacks
- [ ] Code examples pass CI validation
- [ ] Dark mode renders correctly with adequate contrast
- [ ] Copy strips line numbers and prompt characters

## Output Format

### Code Display Specification: `{component or page}`

**Current State**:
- [Inventory of existing code block patterns]

**Issues Found**:
| Issue | Location | Severity | Fix |
|-------|----------|----------|-----|
| [issue] | [where] | High/Med/Low | [fix] |

**Recommendations**:
1. [Prioritized improvements]

**Implementation Spec** (if building):
- Component: [component name/path]
- Props: [configurable properties]
- Behavior: [interaction specification]

## Guidelines

- **DO**: Ensure every code block is copy-pasteable without manual cleanup
- **DO**: Test code examples in CI against the documented API version
- **DO**: Provide static fallbacks for all interactive features
- **DON'T**: Rely on hover states for essential controls (copy button)
- **DON'T**: Auto-execute code in playgrounds — require explicit "Run" action
- **DON'T**: Show API keys in page source, even for authenticated users

## Related Components

- **scaffolding-page-archetypes skill**: Archetypes define where code blocks appear in page anatomy
- **analyzing-docs-benchmarks skill**: Benchmark dimension 4 (Code Sample Quality) scores these patterns
- **designing-typography-layout skill**: Code block typography (monospace font, line height) standards
- **auditing-content-quality skill**: Quality rubric includes code executability checks
