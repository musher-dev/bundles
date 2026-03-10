# Tooling Ecosystem

Comparative analysis of tools used in PostgreSQL schema governance pipelines. Each tool serves a distinct role; this reference maps strengths, weaknesses, licensing, overlap, and recommended combinations.

## Tool Inventory

### Atlas

| Attribute | Details |
|-----------|---------|
| Role | Schema-as-code: declarative schema diffing, versioned migrations, lint, CI validation |
| License | Apache 2.0 (Community), Commercial (Pro) |
| Pro Constraint | Schema Monitoring, cloud dashboard, and some analyzers require Atlas Pro v0.38+ |
| Install | `curl -sSf https://atlasgo.sh \| sh` |
| Languages | Go binary, language-agnostic |

**Strengths:**
- Declarative schema diffing against any source (ORM models, HCL, SQL)
- Built-in migration linting (destructive changes, backward compatibility)
- Migration integrity enforcement (checksums, linear history)
- HCL configuration with environment support
- Dev database pattern for safe diffing

**Weaknesses:**
- Pro features require commercial license for production monitoring
- HCL configuration has a learning curve
- Limited SQL formatting (HCL only via `atlas fmt`)

### Squawk

| Attribute | Details |
|-----------|---------|
| Role | PostgreSQL migration safety linter |
| License | MIT |
| Install | `npm install -g squawk-cli` or `cargo install squawk` |
| Languages | Rust binary |

**Strengths:**
- PostgreSQL-specific safety rules (lock analysis, CONCURRENTLY, type safety)
- Fast execution
- Simple configuration
- Good CI integration (GitHub Actions, GitLab CI)

**Weaknesses:**
- Migration safety only — no schema diffing, no integrity checks
- PostgreSQL-only (by design)
- Smaller rule set than Atlas lint

### SQLFluff

| Attribute | Details |
|-----------|---------|
| Role | SQL linter and formatter |
| License | MIT |
| Install | `pip install sqlfluff` |
| Languages | Python |

**Strengths:**
- Rich rule set for formatting and style
- Multi-dialect support (PostgreSQL, MySQL, BigQuery, Snowflake)
- Auto-fix capability (`sqlfluff fix`)
- diff-quality integration for PR-scoped linting
- Highly configurable (`.sqlfluff` INI file)

**Weaknesses:**
- Slower than native tools on large files
- Complex configuration surface
- No migration safety analysis
- Python dependency

### pg_formatter

| Attribute | Details |
|-----------|---------|
| Role | PostgreSQL SQL formatter |
| License | BSD |
| Install | `apt install pgformatter` or `cpan pgFormatter` |
| Languages | Perl |

**Strengths:**
- Fast execution
- Simple configuration
- PostgreSQL-native formatting rules
- Minimal dependencies

**Weaknesses:**
- Formatting only — no lint rules
- Less configurable than SQLFluff
- No diff-quality integration without wrapper scripts
- Perl dependency

### pgTAP

| Attribute | Details |
|-----------|---------|
| Role | PostgreSQL unit testing framework |
| License | MIT |
| Install | `CREATE EXTENSION pgtap` (from PGXN or package manager) |
| Languages | PL/pgSQL extension |

**Strengths:**
- TAP-compatible output (works with `pg_prove`, CI reporters)
- Rich assertion library (schema, data, function, RLS testing)
- Tests run inside transactions (automatic rollback)
- Battle-tested in PostgreSQL ecosystem

**Weaknesses:**
- Requires PostgreSQL extension installation
- Tests written in SQL/PL/pgSQL (unfamiliar to some developers)
- No performance testing capabilities
- Requires a running PostgreSQL instance

### RegreSQL

| Attribute | Details |
|-----------|---------|
| Role | Query plan regression testing |
| License | MIT |
| Install | `go install github.com/dimitri/regresql@latest` |
| Languages | Go binary |

**Strengths:**
- Captures query plan baselines as files
- Compares plans across runs
- Simple file-based workflow
- Git-friendly baseline storage

**Weaknesses:**
- Less actively maintained than other tools
- Limited to plan comparison (no cost threshold analysis)
- Requires manual baseline management
- Plans are sensitive to data distribution changes

## Overlap Map

Some tools cover similar ground. When using multiple tools, suppress overlapping rules to avoid conflicting or duplicate output.

| Concern | Atlas | Squawk | SQLFluff | pg_formatter | pgTAP | RegreSQL |
|---------|-------|--------|----------|--------------|-------|----------|
| Destructive change detection | Yes | — | — | — | — | — |
| Backward compatibility | Yes | — | — | — | — | — |
| Migration integrity | Yes | — | — | — | — | — |
| Lock safety analysis | Partial | Yes | — | — | — | — |
| CONCURRENTLY enforcement | Yes | Yes | — | — | — | — |
| SQL formatting | — | — | Yes | Yes | — | — |
| HCL formatting | Yes | — | — | — | — | — |
| Schema structure tests | — | — | — | — | Yes | — |
| RLS behavioral tests | — | — | — | — | Yes | — |
| Query plan regression | — | — | — | — | — | Yes |

**Key overlaps to manage:**
- **Atlas lint + Squawk** on CONCURRENTLY enforcement — use Atlas for directive validation, Squawk for lock analysis
- **SQLFluff + pg_formatter** on SQL formatting — choose one, not both

## Recommended Combinations by Maturity

### MVQG (Starting Out)

| Tool | Purpose |
|------|---------|
| Atlas (Community) | Migration diffing, linting, integrity |
| Squawk | Migration safety (locks, CONCURRENTLY) |

Two tools, covering the highest-impact concerns. No formatting, no behavioral testing yet.

### Growing Team

| Tool | Purpose |
|------|---------|
| Atlas (Community) | Migration diffing, linting, integrity |
| Squawk | Migration safety |
| SQLFluff | SQL formatting (advisory) |
| pgTAP | RLS and constraint testing |

Four tools. Formatting is advisory. Behavioral tests cover security-critical RLS.

### High Maturity

| Tool | Purpose |
|------|---------|
| Atlas (Pro) | Migration management + Schema Monitoring |
| Squawk | Migration safety |
| SQLFluff | SQL formatting (advisory) |
| pgTAP | Full behavioral test suite |
| RegreSQL | Query plan regression baselines |

Five tools. Atlas Pro enables drift detection. RegreSQL catches performance regressions.
