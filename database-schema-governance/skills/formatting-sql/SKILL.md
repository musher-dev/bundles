---
name: formatting-sql
version: 1.0.0
user-invocable: false
description: >-
  Configure and enforce SQL formatting standards using SQLFluff, pg_formatter,
  and atlas fmt. Covers tool comparison, diff-quality integration for PR-scoped
  linting, pre-commit hook configuration, formatting as advisory-only policy,
  and resolution of formatting debates. Use when setting up SQL formatters,
  configuring linters, or deciding between formatting tools. Triggered by: SQL
  formatting, SQLFluff, pg_formatter, atlas fmt, diff-quality, SQL style, SQL
  linting, format SQL.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# SQL Formatting

Standards and tooling for consistent SQL formatting across migration files, schema definitions, and ad-hoc queries.

## Purpose

Inconsistent SQL formatting creates noisy diffs, slows code review, and causes unnecessary merge conflicts. Formatting rules must be automated and enforced consistently — but never at the cost of blocking production-critical changes. This skill covers tool selection, configuration, CI integration, and the critical principle that formatting is always advisory, never a hard block.

Use this skill when choosing between SQL formatting tools, configuring SQLFluff or pg_formatter, setting up pre-commit hooks for SQL files, integrating diff-quality for PR-scoped linting, or resolving formatting debates within a team.

---

## Tool Comparison

### SQLFluff

A Python-based SQL linter and formatter with rule-based configuration.

| Aspect | Details |
|--------|---------|
| Language | Python |
| Install | `pip install sqlfluff` |
| Dialects | PostgreSQL, MySQL, BigQuery, Snowflake, and others |
| Configuration | `.sqlfluff` file (INI format) |
| Fixable | Yes (`sqlfluff fix`) |
| CI Integration | Native CLI, diff-quality compatible |
| Strengths | Highly configurable rules, dialect-aware parsing, auto-fix |
| Weaknesses | Slower on large files, complex configuration surface |

### pg_formatter

A Perl-based PostgreSQL-specific formatter.

| Aspect | Details |
|--------|---------|
| Language | Perl |
| Install | `apt install pgformatter` or `cpan pgFormatter` |
| Dialects | PostgreSQL only |
| Configuration | Command-line flags or `.pg_format` file |
| Fixable | Yes (reformats in place) |
| CI Integration | CLI only, requires wrapper for diff-quality |
| Strengths | Fast, simple, PostgreSQL-native |
| Weaknesses | Less configurable, no lint rules beyond formatting |

### atlas fmt

Atlas's built-in formatter for HCL files (not SQL).

| Aspect | Details |
|--------|---------|
| Scope | `atlas.hcl` and `.hcl` files only |
| Install | Bundled with Atlas CLI |
| Usage | `atlas fmt` or `atlas fmt --check` |
| Purpose | Consistent HCL formatting in Atlas configuration |

**Key distinction:** `atlas fmt` formats HCL configuration files, not SQL migration files. SQL formatting requires SQLFluff or pg_formatter.

---

## Recommended Configuration

### SQLFluff for PostgreSQL

```ini
# .sqlfluff
[sqlfluff]
dialect = postgres
templater = raw
max_line_length = 100
exclude_rules = L031

[sqlfluff:rules:capitalisation.keywords]
capitalisation_policy = upper

[sqlfluff:rules:capitalisation.identifiers]
capitalisation_policy = lower

[sqlfluff:rules:capitalisation.functions]
capitalisation_policy = upper

[sqlfluff:rules:capitalisation.types]
capitalisation_policy = upper

[sqlfluff:rules:layout.long_lines]
ignore_comment_lines = true

[sqlfluff:rules:convention.terminator]
multiline_newline = true
require_final_semicolon = true
```

### Key Rules Explained

| Rule | Effect | Rationale |
|------|--------|-----------|
| Keywords UPPER | `SELECT`, `FROM`, `WHERE` | Visual distinction from identifiers |
| Identifiers lower | `user_id`, `created_at` | Matches PostgreSQL fold-to-lowercase behavior |
| Functions UPPER | `COUNT()`, `COALESCE()` | Consistent with keyword casing |
| Types UPPER | `TEXT`, `BIGINT`, `TIMESTAMPTZ` | Consistent with keyword casing |
| Max line length 100 | Wrap long lines | Readable in code review without horizontal scrolling |
| Final semicolon | Require `;` at end | Explicit statement termination |

---

## PR-Scoped Linting with diff-quality

Full-file linting on every PR creates noise — existing files may have pre-existing violations that are not the author's responsibility. diff-quality restricts lint output to only the lines changed in the PR.

### Setup

```bash
pip install diff-quality sqlfluff
```

### Usage in CI

```bash
# Lint only changed SQL lines in the current PR
diff-quality --violations sqlfluff --compare-branch origin/main --fail-under 100
```

### How It Works

1. `diff-quality` computes the diff between the PR branch and the base branch
2. Runs `sqlfluff lint` on changed files
3. Filters output to only violations on changed lines
4. Exits non-zero if any changed line has a violation

### Benefits

| Approach | Noise Level | Adoption Friction |
|----------|-------------|-------------------|
| Full-file lint | High (pre-existing violations) | High (developers reject unrelated failures) |
| diff-quality | Low (only your changes) | Low (fair, proportional enforcement) |

---

## Pre-commit Hook Configuration

### Using pre-commit Framework

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/sqlfluff/sqlfluff
    rev: 3.0.0
    hooks:
      - id: sqlfluff-lint
        args: ['--dialect', 'postgres']
        files: '\.sql$'
      - id: sqlfluff-fix
        args: ['--dialect', 'postgres', '--force']
        files: '\.sql$'
```

### Atlas HCL Formatting Hook

```yaml
  - repo: local
    hooks:
      - id: atlas-fmt
        name: atlas fmt
        entry: atlas fmt --check
        language: system
        files: '\.hcl$'
```

### Hook Behavior

Pre-commit hooks should:
- Run on staged files only (pre-commit framework handles this)
- Auto-fix when possible (`sqlfluff-fix`)
- Be fast (< 5 seconds for typical changesets)
- Never block commits for formatting alone — use `--exit-zero` if needed

---

## Formatting as Advisory Policy

**Formatting violations must never be hard-block rules.** This is a deliberate policy decision, not a limitation.

### Rationale

| Factor | Hard-Block Risk |
|--------|----------------|
| Emergency hotfixes | Blocked by trailing whitespace — unacceptable |
| New team member onboarding | First PR rejected for formatting — discouraging |
| Generated SQL | Tool output may not match formatter rules — false positives |
| Cross-tool conflicts | SQLFluff and pg_formatter may disagree on edge cases |

### Implementation

- **Pre-commit:** Auto-fix (best effort), never block
- **CI:** Report violations in PR comments as advisory warnings
- **Never:** Fail the CI pipeline for formatting-only violations

### Escalation Path

If formatting violations persist across multiple PRs from the same author:
1. PR comment with specific rule explanations
2. Team discussion about adopting auto-fix in editor/save hooks
3. Pair session on formatter setup

Never escalate to hard-block. The cost of blocking production changes for formatting exceeds the cost of inconsistent style.

---

## Resolving Formatting Debates

Formatting debates consume disproportionate engineering time. Use this framework to resolve them quickly.

### Decision Framework

1. **Does it affect correctness?** If yes, it's not a formatting rule — it's a lint rule. Handle separately.
2. **Can the formatter enforce it automatically?** If yes, configure it and move on. The tool decides.
3. **Do team members disagree?** Pick the formatter's default and close the discussion. Consistency matters more than any specific style.

### Common Debates and Resolutions

| Debate | Resolution | Rationale |
|--------|-----------|-----------|
| Keyword casing (upper vs lower) | UPPER keywords | Industry convention, visual distinction |
| Trailing commas in column lists | Formatter default | Reduces diff noise on additions |
| Single-line vs multi-line SELECT | Multi-line when > 2 columns | Readable diffs |
| Indent size (2 vs 4 spaces) | 4 spaces | Matches Python ecosystem convention |
| Alias with or without `AS` | Always use `AS` | Explicit is better than implicit |

---

## Anti-Patterns

### 1. Hard-Blocking on Formatting

Failing the CI pipeline for formatting violations. Developers bypass the system with `--no-verify`, eroding trust in all quality gates — including the important ones.

### 2. Full-File Linting Without diff-quality

Linting entire files instead of just changed lines. Authors are penalized for pre-existing violations they didn't introduce, creating resentment and merge delays.

### 3. Running Multiple Formatters Without Coordination

Running both SQLFluff and pg_formatter on the same files without ensuring they agree. The tools may produce conflicting output, creating an unresolvable formatting loop.

### 4. Formatting Generated SQL

Applying formatting rules to Atlas-generated migration SQL. Atlas output follows its own conventions. Reformatting it creates diffs that obscure the actual schema change and breaks `atlas.sum` checksums.

### 5. No Auto-Fix in Pre-commit

Running `sqlfluff lint` in pre-commit without `sqlfluff fix`. Developers see violations but must fix them manually, adding friction without providing the easy path.

---

## Examples

### Example 1: Initial SQLFluff Setup

A team adopting SQL formatting for the first time.

1. Install SQLFluff: `pip install sqlfluff`
2. Create `.sqlfluff` with the recommended PostgreSQL configuration above
3. Add pre-commit hooks with both `sqlfluff-lint` and `sqlfluff-fix`
4. Add `diff-quality` to CI as advisory (no `--fail-under`)
5. After 2 weeks of advisory, set `--fail-under 100` on diff-quality for new lines only

### Example 2: Migrating from pg_formatter to SQLFluff

A team currently using pg_formatter wants richer lint rules.

1. Run `sqlfluff lint` on the existing codebase to baseline violations
2. Configure SQLFluff rules to match pg_formatter's current output where possible
3. Suppress rules that create excessive diff against existing files
4. Switch pre-commit hooks from pg_formatter to SQLFluff
5. Use diff-quality in CI so existing files are not penalized
6. Remove pg_formatter from the toolchain after one sprint of dual-running

### Example 3: Handling Atlas-Generated SQL

A developer asks why `sqlfluff lint` fails on a migration file generated by `atlas migrate diff`.

**Explanation:** Atlas generates SQL with its own formatting conventions. Do not reformat Atlas-generated files — this would change the file content and invalidate `atlas.sum`.

**Resolution:** Exclude migration directories from SQLFluff:
```ini
# .sqlfluff
[sqlfluff]
exclude_rules_path = migrations/
```

Or use a `.sqlfluffignore` file:
```
migrations/
```

Format only hand-written SQL (seed scripts, backfill scripts, ad-hoc queries).
