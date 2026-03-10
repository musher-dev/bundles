---
name: building-code-displays
user-invocable: false
description: Build marketing-grade code displays for the Musher marketing site -- split-panel code-then-output layouts, CSS-only syntax highlighting, tabbed views, and terminal chrome. Use when implementing IA Section 3 (Mechanism), building code blocks, creating terminal demos, highlighting YAML/CLI output, or adding copy buttons to code. Triggered by: code display, code block, syntax highlighting, code-then-output, split panel code, terminal chrome, YAML highlight, CLI output, code snippet, tabbed code, multi-language code, mechanism section code, bundle yaml, pipeline yaml, marketing code.
allowed-tools: Read, Write, Edit, Grep, Glob
---

# Building Marketing Code Displays

## Purpose

Build code display components for the marketing site without external syntax highlighting dependencies. The IA's Mechanism section (Section 3) requires `code-then-output` split panels showing YAML config and CLI output. This skill covers the full taxonomy of code displays and how to build each one.

## Source Documents

- **IA Section 3**: `/workspace/docs/marketing/information-architecture.md` -- `code-then-output` layout pattern, content slot definitions
- **Wireframe Copy**: `/workspace/docs/marketing/wireframe-copy.md` -- exact YAML and CLI output content for bundle and pipeline code blocks
- **Design Tokens**: `/workspace/apps/marketing-site/src/app.css` -- `@theme` colors for syntax highlighting
- **Existing CodeSnippet**: `apps/marketing-site/src/lib/components/CodeSnippet.svelte` -- reference for code block with copy button
- **Existing Terminal**: `apps/marketing-site/src/lib/components/InteractiveTerminal.svelte` -- reference for terminal chrome pattern

## Code Display Taxonomy

### 1. Static Code Snippet

Single code block with language label and copy button. Already implemented as `CodeSnippet.svelte`.

**Use for**: Standalone code examples, install commands.

### 2. Code-Then-Output Split (Primary -- IA Section 3)

Side-by-side or stacked panels: config on left, CLI output on right.

**Use for**: Mechanism section showing bundle YAML + `mush bundle apply` output, pipeline YAML + `mush pipeline run` output.

### 3. Tabbed Multi-File View

Multiple files shown via tab switching -- only one visible at a time.

**Use for**: When showing related files (e.g., `bundle.yaml` + `pipeline.yaml` in a tabbed interface).

### 4. Interactive Terminal

Animated terminal with typewriter effect and state transitions. Already implemented as `InteractiveTerminal.svelte`.

**Use for**: Hero section demo.

## Code-Then-Output Split Panel

This is the primary pattern needed for IA Section 3.

### Layout Structure

```svelte
<div class="grid gap-4 lg:grid-cols-2">
  <!-- Left panel: Configuration file -->
  <div class="code-panel">
    <div class="code-chrome">
      <span class="filename">bundle.yaml</span>
      <CopyButton code={bundleYaml} />
    </div>
    <pre class="code-body"><code>{@html highlightYaml(bundleYaml)}</code></pre>
  </div>

  <!-- Right panel: CLI output -->
  <div class="code-panel">
    <div class="code-chrome terminal">
      <div class="terminal-dots">
        <span class="dot-red"></span>
        <span class="dot-yellow"></span>
        <span class="dot-green"></span>
      </div>
      <span class="filename">output</span>
    </div>
    <pre class="code-body"><code>{@html highlightCliOutput(cliOutput)}</code></pre>
  </div>
</div>
```

### Responsive Behavior

- **Desktop** (lg+): `grid-cols-2` -- side by side
- **Mobile** (<lg): stacked vertically, config on top, output below
- Both panels have equal visual weight -- same height via `min-h` or grid stretching

### Chrome Pattern

The code panel chrome matches existing `CodeSnippet.svelte` and `InteractiveTerminal.svelte`:

```svelte
<!-- Config file chrome -->
<div class="bg-surface-04/50 border-border-subtle flex items-center justify-between border-b px-4 py-2">
  <span class="text-label text-text-tertiary font-mono">{filename}</span>
  <CopyButton {code} />
</div>

<!-- Terminal output chrome -->
<div class="bg-surface-04/50 border-border-subtle flex items-center gap-2 border-b px-4 py-2">
  <div class="flex items-center gap-1.5">
    <div class="bg-signal-error h-2.5 w-2.5 rounded-full"></div>
    <div class="bg-signal-warning h-2.5 w-2.5 rounded-full"></div>
    <div class="bg-signal-success h-2.5 w-2.5 rounded-full"></div>
  </div>
  <span class="text-label text-text-tertiary font-mono ml-2">output</span>
</div>
```

### Panel Styling

```svelte
<!-- Outer panel wrapper -->
<div class="bg-surface-02 border-border-subtle shadow-panel overflow-hidden rounded-lg border">
  <!-- Chrome header (above) -->
  <!-- Code body -->
  <pre class="overflow-x-auto p-4 leading-relaxed"><code class="text-data font-mono">{@html highlighted}</code></pre>
</div>
```

## CSS-Only Syntax Highlighting

Use design token colors for syntax highlighting. No external dependencies (no Shiki, no Prism, no highlight.js).

### Color Mapping

| YAML Token | Token Color | Tailwind Class | CSS Variable |
|------------|------------|---------------|-------------|
| Keys | `text-signal-action` | `color: var(--color-signal-action)` | #4589ff |
| String values | `text-signal-success` | `color: var(--color-signal-success)` | #42be65 |
| Comments | `text-text-tertiary` | `color: var(--color-text-tertiary)` | #8d8d8d |
| Booleans/numbers | `text-signal-warning` | `color: var(--color-signal-warning)` | #ff832b |
| Punctuation (`:`, `-`) | `text-text-secondary` | `color: var(--color-text-secondary)` | #c6c6c6 |
| Plain text | `text-text-primary` | `color: var(--color-text-primary)` | #f4f4f4 |

| CLI Token | Token Color | Tailwind Class |
|-----------|------------|---------------|
| Command prompt (`$`) | `text-signal-action` | #4589ff |
| Command name | `text-text-primary` | #f4f4f4 |
| Flags (`--repo`) | `text-signal-action` | #4589ff |
| Status labels (`[OK]`, `Done.`) | `text-signal-success` | #42be65 |
| Indented output | `text-text-secondary` | #c6c6c6 |
| Timing (`14.2s`) | `text-signal-warning` | #ff832b |

### Highlighting Functions

Implement highlighting as pure functions that return HTML strings with `<span>` tags:

```typescript
function highlightYaml(code: string): string {
  return code
    .split('\n')
    .map((line) => {
      // Comments
      if (line.trimStart().startsWith('#')) {
        return `<span style="color: var(--color-text-tertiary)">${escapeHtml(line)}</span>`;
      }

      // Key: value pairs
      const keyMatch = line.match(/^(\s*)([\w-]+)(:)(.*)/);
      if (keyMatch) {
        const [, indent, key, colon, value] = keyMatch;
        const highlightedValue = highlightYamlValue(value.trim());
        return `${indent}<span style="color: var(--color-signal-action)">${escapeHtml(key)}</span><span style="color: var(--color-text-secondary)">${colon}</span> ${highlightedValue}`;
      }

      // List items
      const listMatch = line.match(/^(\s*)(- )(.*)/);
      if (listMatch) {
        const [, indent, dash, value] = listMatch;
        return `${indent}<span style="color: var(--color-text-secondary)">${dash}</span>${highlightYamlValue(value)}`;
      }

      return escapeHtml(line);
    })
    .join('\n');
}

function highlightYamlValue(value: string): string {
  if (!value) return '';

  // Quoted strings
  if (/^["'].*["']$/.test(value)) {
    return `<span style="color: var(--color-signal-success)">${escapeHtml(value)}</span>`;
  }

  // Booleans and numbers
  if (/^(true|false|\d+(\.\d+)?)$/.test(value)) {
    return `<span style="color: var(--color-signal-warning)">${escapeHtml(value)}</span>`;
  }

  // Plain values
  return `<span style="color: var(--color-text-primary)">${escapeHtml(value)}</span>`;
}

function highlightCliOutput(output: string): string {
  return output
    .split('\n')
    .map((line) => {
      // Command prompt lines
      if (line.startsWith('$ ')) {
        const cmd = line.slice(2);
        return `<span style="color: var(--color-signal-action)">$</span> <span style="color: var(--color-text-primary)">${escapeHtml(cmd)}</span>`;
      }

      // Status lines with success indicators
      if (/\b(Done|complete|completed|OK)\b/i.test(line)) {
        return `<span style="color: var(--color-signal-success)">${escapeHtml(line)}</span>`;
      }

      // Indented output
      if (line.startsWith('  ')) {
        // Highlight timing values
        const withTiming = escapeHtml(line).replace(
          /(\d+\.?\d*s)/g,
          '<span style="color: var(--color-signal-warning)">$1</span>'
        );
        return `<span style="color: var(--color-text-secondary)">${withTiming}</span>`;
      }

      return `<span style="color: var(--color-text-secondary)">${escapeHtml(line)}</span>`;
    })
    .join('\n');
}

function escapeHtml(text: string): string {
  return text
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');
}
```

**Important**: Always escape HTML before inserting into `{@html}` to prevent XSS. The `escapeHtml` function must run before any `<span>` wrapping.

### Using `{@html}` Safely

The highlighting functions produce trusted HTML from hardcoded IA content. This is safe because:

1. The input is static wireframe copy, not user input
2. All values are escaped via `escapeHtml()` before span-wrapping
3. The `{@html}` directive is used only for the pre-formatted code content

## Tabbed Multi-File View

For showing multiple related files with tab switching:

```svelte
<script lang="ts">
  interface Tab {
    label: string;
    filename: string;
    code: string;
    language: string;
  }

  interface Props {
    tabs: Tab[];
  }

  let { tabs }: Props = $props();
  let activeIndex = $state(0);
</script>

<div class="bg-surface-02 border-border-subtle shadow-panel overflow-hidden rounded-lg border">
  <!-- Tab bar -->
  <div class="bg-surface-04/50 border-border-subtle flex border-b">
    {#each tabs as tab, i}
      <button
        type="button"
        onclick={() => activeIndex = i}
        class="text-label px-4 py-2 font-mono transition-colors duration-fast
          {i === activeIndex
            ? 'text-text-primary border-b-2 border-signal-action bg-surface-02'
            : 'text-text-tertiary hover:text-text-secondary'}"
      >
        {tab.filename}
      </button>
    {/each}
  </div>

  <!-- Active panel -->
  <pre class="overflow-x-auto p-4 leading-relaxed"><code class="text-data font-mono">{@html highlight(tabs[activeIndex])}</code></pre>
</div>
```

## IA Section 3 Content

The wireframe copy specifies exact YAML and CLI content. Here are the content slots to implement:

### Bundle Code Block

**Left panel** (`bundle.yaml`):
```yaml
name: code-review
agent: claude-code
allowed_tools:
  - read_file
  - edit_file
  - run_terminal_cmd
rules:
  - "Review for correctness, not style"
  - "Flag security issues as blocking"
```

**Right panel** (CLI output):
```
$ mush bundle apply --repo my-org/api
Applying bundle 'code-review' to my-org/api...
  Agent: claude-code
  Rules: 2 applied
  Tools: 3 allowed
  Done. Agent config written to .mush/bundles/code-review.yaml
```

### Pipeline Code Block

**Left panel** (`pipeline.yaml`):
```yaml
trigger:
  event: pull_request.opened
  source: github
job:
  bundle: code-review
  retry:
    attempts: 3
    backoff: exponential
```

**Right panel** (CLI output):
```
$ mush pipeline run --event pr-opened-42
Event received: pull_request.opened #42
Dispatching job: code-review
  Attempt 1/3... running
  Agent: claude-code
  Status: completed (14.2s)
Pipeline complete.
```

## Copy Button Integration

Reuse the existing `CopyButton.svelte` pattern from `CodeSnippet.svelte`:

```svelte
<script lang="ts">
  interface Props {
    code: string;
  }

  let { code }: Props = $props();
  let copied = $state(false);

  async function copy() {
    await navigator.clipboard.writeText(code);
    copied = true;
    setTimeout(() => (copied = false), 2000);
  }
</script>

<button
  type="button"
  onclick={copy}
  class="text-label text-text-tertiary hover:text-text-primary flex items-center gap-1 transition-colors duration-fast"
>
  {#if copied}
    <svg class="text-signal-success h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
    </svg>
    <span>Copied!</span>
  {:else}
    <svg class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z" />
    </svg>
    <span>Copy</span>
  {/if}
</button>
```

Or import the existing `CopyButton` from `$lib/components/CopyButton.svelte`.

## Typography in Code Blocks

- **Font**: `font-mono` (JetBrains Mono from `@theme`)
- **Size**: `text-data` (13px/20px) for code content
- **Line height**: `leading-relaxed` (1.625) for readability in code blocks
- **Chrome labels**: `text-label` (11px/16px) with `font-mono`
- **Tab size**: 2 spaces (standard for YAML)

## Anti-Patterns

| DO NOT | DO |
|--------|-----|
| Install Shiki, Prism, or highlight.js | Use CSS-only highlighting with design token colors |
| Use `<SyntaxHighlighter>` from React | Build highlighting as Svelte-native pure functions |
| Use colors outside the token system | Map syntax tokens to `--color-signal-*` and `--color-text-*` |
| Highlight every token with regex | Keep it simple -- keys, values, comments, prompts are enough |
| Hard-code hex values in highlighting | Use `var(--color-*)` CSS variables for all colors |
| Use `dangerouslySetInnerHTML` | Use Svelte's `{@html}` with proper escaping |
| Auto-detect language | Explicitly pass language/content type per panel |

## Related Skills

- **`scaffolding-sections`** -- how Mechanism section fits into the overall section scaffold
- **`animating-scroll-reveals`** -- optional scroll reveal for code blocks appearing on scroll
- **`auditing-design-compliance`** -- verify code displays use correct tokens
