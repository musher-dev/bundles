# GitHub Config Templates

Reference templates and syntax guides for GitHub repository configuration files. Each section provides the canonical format, field definitions, and production-ready examples.

## Issue Template YAML Form Reference

### Schema Overview

GitHub YAML issue forms use a structured schema that renders as interactive forms in the browser. Each template is a `.yml` file in `.github/ISSUE_TEMPLATE/`.

### Top-Level Fields

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Template name shown in the issue template chooser |
| `description` | Yes | Short description shown below the template name |
| `title` | No | Pre-filled issue title (supports `[placeholder]` syntax) |
| `labels` | No | Array of labels automatically applied to issues |
| `assignees` | No | Array of GitHub usernames auto-assigned |
| `projects` | No | Array of project board numbers |
| `body` | Yes | Array of form field elements |

### Form Field Types

#### `markdown`

Renders static text. Use for instructions or section headers.

```yaml
- type: markdown
  attributes:
    value: |
      ## Before You Begin
      Please search existing issues to avoid duplicates.
```

#### `textarea`

Multi-line free text input. Supports markdown rendering in the submitted issue.

```yaml
- type: textarea
  id: description
  attributes:
    label: Description
    description: A clear description of the bug.
    placeholder: Tell us what happened...
    render: bash  # Optional: syntax highlighting for code blocks
  validations:
    required: true
```

#### `input`

Single-line text input.

```yaml
- type: input
  id: version
  attributes:
    label: Version
    description: What version are you using?
    placeholder: e.g., 2.1.0
  validations:
    required: true
```

#### `dropdown`

Single or multi-select dropdown.

```yaml
- type: dropdown
  id: severity
  attributes:
    label: Severity
    description: How severe is this issue?
    options:
      - Critical — system is unusable
      - Major — feature is broken
      - Minor — cosmetic or workaround exists
    multiple: false  # Set true for multi-select
  validations:
    required: true
```

#### `checkboxes`

Checkbox group. Commonly used for acknowledgments or categorization.

```yaml
- type: checkboxes
  id: terms
  attributes:
    label: Acknowledgments
    options:
      - label: I have searched existing issues for duplicates
        required: true
      - label: I have read the contributing guidelines
        required: true
```

### Bug Report Template

```yaml
name: Bug Report
description: Report a bug or unexpected behavior
title: "[Bug]: "
labels: ["bug", "triage"]
body:
  - type: markdown
    attributes:
      value: |
        Thank you for reporting a bug. Please fill out the sections below so we can reproduce and fix the issue.

  - type: textarea
    id: description
    attributes:
      label: Bug Description
      description: A clear and concise description of the bug.
      placeholder: Describe the unexpected behavior...
    validations:
      required: true

  - type: textarea
    id: reproduction
    attributes:
      label: Steps to Reproduce
      description: Minimal steps to reproduce the behavior.
      placeholder: |
        1. Go to '...'
        2. Click on '...'
        3. Scroll down to '...'
        4. See error
    validations:
      required: true

  - type: textarea
    id: expected
    attributes:
      label: Expected Behavior
      description: What did you expect to happen?
    validations:
      required: true

  - type: textarea
    id: actual
    attributes:
      label: Actual Behavior
      description: What actually happened?
    validations:
      required: true

  - type: input
    id: version
    attributes:
      label: Version
      description: What version are you using?
      placeholder: e.g., 2.1.0
    validations:
      required: true

  - type: dropdown
    id: os
    attributes:
      label: Operating System
      options:
        - macOS
        - Windows
        - Linux
        - Other
    validations:
      required: true

  - type: textarea
    id: logs
    attributes:
      label: Relevant Log Output
      description: Paste any relevant log output or error messages.
      render: bash

  - type: textarea
    id: context
    attributes:
      label: Additional Context
      description: Any other context, screenshots, or information.

  - type: checkboxes
    id: acknowledgments
    attributes:
      label: Acknowledgments
      options:
        - label: I have searched existing issues for duplicates
          required: true
```

### Feature Request Template

```yaml
name: Feature Request
description: Suggest a new feature or enhancement
title: "[Feature]: "
labels: ["enhancement"]
body:
  - type: markdown
    attributes:
      value: |
        Thank you for suggesting a feature. Please describe what you need and why it matters.

  - type: textarea
    id: problem
    attributes:
      label: Problem Statement
      description: What problem does this feature solve? Describe the frustration or gap.
      placeholder: I'm always frustrated when...
    validations:
      required: true

  - type: textarea
    id: solution
    attributes:
      label: Proposed Solution
      description: Describe the solution you'd like to see.
    validations:
      required: true

  - type: textarea
    id: alternatives
    attributes:
      label: Alternatives Considered
      description: What other solutions or workarounds have you considered?

  - type: dropdown
    id: priority
    attributes:
      label: Priority
      description: How important is this feature to your workflow?
      options:
        - Nice to have
        - Important — affects productivity
        - Critical — blocking my work
    validations:
      required: true

  - type: textarea
    id: context
    attributes:
      label: Additional Context
      description: Any mockups, examples, or references that help illustrate the request.

  - type: checkboxes
    id: acknowledgments
    attributes:
      label: Acknowledgments
      options:
        - label: I have searched existing issues and feature requests for duplicates
          required: true
```

### Issue Template Config (`config.yml`)

Controls blank issue creation and external links in the template chooser.

```yaml
blank_issues_enabled: false
contact_links:
  - name: Documentation
    url: https://example.com/docs
    about: Check the documentation before opening an issue
  - name: Community Discussions
    url: https://github.com/org/repo/discussions
    about: Ask questions and share ideas in Discussions
```

Set `blank_issues_enabled: false` to force contributors through structured templates. Add `contact_links` to redirect common questions to better channels.

## PR Template Reference

### Complete PR Template

```markdown
## Description

<!-- Describe your changes in detail. What problem does this PR solve? -->

## Type of Change

<!-- Check the relevant options -->

- [ ] Bug fix (non-breaking change that fixes an issue)
- [ ] New feature (non-breaking change that adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to change)
- [ ] Documentation update
- [ ] Refactoring (no functional changes)
- [ ] CI/CD or build changes

## Testing

<!-- Describe the tests you ran and how to reproduce them -->

- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist

- [ ] My code follows the project's style guidelines
- [ ] I have performed a self-review of my code
- [ ] I have added tests that prove my fix is effective or my feature works
- [ ] New and existing unit tests pass locally
- [ ] I have updated the documentation accordingly

## Related Issues

<!-- Link related issues using keywords: Closes #123, Fixes #456, Relates to #789 -->
```

### Design Principles

- **HTML comments** (`<!-- -->`) provide instructions without cluttering the rendered PR
- **Checkboxes** create scannable checklists that reviewers can verify at a glance
- **Related Issues** section uses GitHub keywords (`Closes`, `Fixes`) for automatic issue linking
- **Type of Change** helps reviewers gauge scope and risk before reading code
- Total completion time should stay under 2 minutes for a typical PR

## CODEOWNERS Syntax Reference

### Pattern Matching Rules

CODEOWNERS uses the same glob syntax as `.gitignore` with one critical difference: **the last matching pattern wins** (not the first).

```
# Syntax: <pattern> <owner> [<owner>...]

# Owners can be:
#   @username          — individual GitHub user
#   @org/team-name     — GitHub team (preferred)
#   email@example.com  — email address (least common)
```

### Precedence Order

Patterns are evaluated top-to-bottom. The **last matching rule** determines ownership. Structure your file from general to specific:

```
# 1. Default owner (catches everything not matched below)
*                       @org/engineering

# 2. Directory-specific owners
/src/api/               @org/backend-team
/src/frontend/          @org/frontend-team
/src/shared/            @org/platform-team
/docs/                  @org/docs-team
/infrastructure/        @org/devops-team

# 3. File-type-specific owners
*.sql                   @org/database-team
*.proto                 @org/api-team
Dockerfile              @org/devops-team

# 4. Critical files (highest specificity, listed last)
/src/api/auth/          @org/security-team
CODEOWNERS              @org/engineering-leads
```

### Common Patterns

| Pattern | Matches |
|---|---|
| `*` | Everything in the repository |
| `/docs/` | The `docs` directory at the repo root |
| `docs/` | Any `docs` directory at any depth |
| `*.js` | All JavaScript files anywhere |
| `/src/api/**` | Everything under `src/api/` recursively |
| `*.generated.*` | All generated files (e.g., `schema.generated.ts`) |

### Monorepo Example

```
# Default
*                           @org/engineering

# Package-level ownership
/packages/auth/             @org/identity-team
/packages/billing/          @org/payments-team
/packages/ui/               @org/design-system-team
/packages/shared/           @org/platform-team

# Infrastructure
/.github/                   @org/devops-team
/terraform/                 @org/infrastructure-team
docker-compose*.yml         @org/devops-team

# Cross-cutting concerns
*.test.ts                   @org/quality-team
/packages/*/migrations/     @org/database-team
```

### Validation Checklist

- Every actively-maintained directory has an explicit owner
- Team references (`@org/team-name`) exist in the GitHub organization
- No individual handles for directories with multiple contributors
- Critical paths (auth, billing, infrastructure) have dedicated owners
- The file ends with high-specificity overrides for sensitive files

## Keep a Changelog Reference

### Format Specification (v1.1.0)

Keep a Changelog defines a human-readable changelog format aligned with Semantic Versioning.

### File Header

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
```

### Change Types

Each version section uses these six categories, in this order:

| Type | Definition | Example |
|---|---|---|
| `Added` | New features | New CLI command, new API endpoint |
| `Changed` | Changes in existing functionality | Updated default timeout, modified response format |
| `Deprecated` | Soon-to-be removed features | Deprecated v1 endpoint, marked config key for removal |
| `Removed` | Now removed features | Removed legacy authentication, dropped Python 3.7 support |
| `Fixed` | Bug fixes | Fixed race condition in queue, corrected timezone handling |
| `Security` | Vulnerability fixes | Patched XSS in input parser, updated dependency with CVE |

### Version Entry Format

```markdown
## [Unreleased]

### Added

- New `--dry-run` flag for the deploy command

## [1.2.0] - 2026-03-15

### Added

- User avatar upload support
- Webhook retry configuration

### Fixed

- Race condition in concurrent session handling
- Incorrect timezone offset for UTC+13 regions

### Security

- Updated `lodash` to patch prototype pollution vulnerability (CVE-2025-XXXXX)

## [1.1.0] - 2026-02-01

### Added

- API rate limiting with configurable thresholds

### Changed

- Default page size increased from 20 to 50

### Deprecated

- `/v1/users/search` endpoint — use `/v2/users` with query parameters instead
```

### Comparison Links

At the bottom of the file, link each version header to a Git comparison URL:

```markdown
[Unreleased]: https://github.com/org/repo/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/org/repo/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/org/repo/compare/v1.0.0...v1.1.0
```

### Rules

- Always maintain an `[Unreleased]` section at the top for in-progress changes
- Use ISO 8601 date format (`YYYY-MM-DD`) for release dates
- List entries as bullet points, one change per line
- Write entries for humans — describe the impact, not the commit
- Link to issues or PRs where relevant (e.g., `Fixed timezone bug (#234)`)
- Order change types consistently: Added, Changed, Deprecated, Removed, Fixed, Security
- Remove empty change type sections — only include categories with entries

## Migration Guide: Markdown to YAML Issue Templates

### When to Migrate

Migrate when you have `.md` files in `.github/ISSUE_TEMPLATE/` and want structured form input, required field validation, and consistent issue formatting.

### Step-by-Step Conversion

#### 1. Inventory existing templates

List all `.md` templates and note their sections, placeholders, and any custom fields.

#### 2. Map markdown sections to form fields

| Markdown Pattern | YAML Field Type |
|---|---|
| `## Description` with free text | `textarea` with `id: description` |
| `**Version:**` placeholder | `input` with `id: version` |
| `## Steps to Reproduce` numbered list | `textarea` with numbered placeholder |
| `- [ ] Checkbox item` | `checkboxes` with `options` array |
| Dropdown-like instructions ("select one of...") | `dropdown` with `options` array |
| Static instructions or notes | `markdown` with `value` |

#### 3. Create the YAML file

- File extension: `.yml` (not `.yaml` — GitHub convention)
- Place in `.github/ISSUE_TEMPLATE/`
- Set `name`, `description`, and `labels` at the top level
- Convert each section into a `body` entry with the appropriate `type`

#### 4. Add validations

- Mark critical fields with `validations.required: true`
- Add `placeholder` text to guide contributors
- Use `render` attribute on `textarea` fields that expect code

#### 5. Create `config.yml`

If not already present, add `.github/ISSUE_TEMPLATE/config.yml` to control blank issue behavior and add contact links.

#### 6. Remove old markdown templates

Once the YAML forms are tested and working, delete the `.md` templates to avoid showing duplicate options in the template chooser.

### Common Pitfalls

- **Missing `id` fields**: Every `input`, `textarea`, `dropdown`, and `checkboxes` element needs a unique `id`
- **Invalid `render` values**: The `render` attribute only works on `textarea` fields and must be a valid language identifier
- **Label mismatch**: The `labels` array must reference labels that already exist in the repository — create them first
- **YAML indentation**: Body entries are array items (start with `-`); attributes are nested objects — mixing these up breaks the form
