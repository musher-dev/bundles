---
name: summarizing-changes
description: >
  Summarize git history, staged diffs, or PR changes into clear, human-readable
  summaries grouped by theme.
---

# Summarizing Changes

Produce concise, theme-grouped summaries from git history, staged diffs, or pull request changes.

## Approach

### 1. Identify the Change Source

Determine what to summarize:

- **Git history** — a range of commits (e.g., last 10 commits, since a tag, between branches)
- **Staged diff** — changes currently staged for commit
- **PR changes** — the full diff of a pull request

Use `git log`, `git diff --staged`, or the PR diff depending on the source.

### 2. Categorize Changes by Theme

Group related changes into logical themes:

- **Features** — new functionality or capabilities added
- **Fixes** — bug fixes and error corrections
- **Refactoring** — structural changes that don't alter behavior
- **Documentation** — updates to docs, comments, or READMEs
- **Infrastructure** — CI/CD, build config, dependency updates
- **Tests** — new or updated test cases

Assign each change to the most appropriate theme. A single commit may span multiple themes if it touches unrelated areas.

### 3. Write the Summary

For each theme with changes:

- Lead with the theme name as a heading
- List 2-5 bullet points describing the most significant changes
- Use past tense, active voice ("Added pagination to the users endpoint")
- Include file paths or function names when they add clarity
- Omit trivial changes (whitespace, formatting) unless they are the only changes

### 4. Add Context

After the themed sections:

- State the total number of commits, files changed, and lines added/removed
- Note any breaking changes or migration requirements
- Flag large diffs that may need closer review

## Output Format

```
## Summary

### {Theme}
- {Change description}
- {Change description}

### {Theme}
- {Change description}

---
{N} commits | {N} files changed | +{N} -{N} lines
```

## Guidelines

- Keep each bullet to one sentence
- Prioritize impact over volume — a one-line security fix matters more than a 500-line rename
- When summarizing a large range, focus on the most meaningful changes and note how many minor changes were omitted
- Do not editorialize — describe what changed, not whether it was a good idea
