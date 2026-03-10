---
name: enforcing-quality-gates
version: 1.0.0
user-invocable: false
description: >-
  Design and review PostgreSQL CI/CD quality gate pipelines including 5-phase
  enforcement workflows (local, pre-commit, PR/CI, staging, production),
  ephemeral database provisioning, tool orchestration (Atlas + Squawk + SQLFluff),
  hard-block vs advisory rule classification, MVQG vs high-maturity gate
  progression, and break-glass exception procedures. Use when designing database
  CI pipelines, reviewing enforcement architecture, or debugging quality gate
  failures. Triggered by: quality gate, CI pipeline database, enforcement
  architecture, hard block, advisory warning, break-glass, MVQG.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Quality Gate Enforcement

Structured framework for designing and reviewing CI/CD quality gates that enforce PostgreSQL schema standards across the development lifecycle.

## Purpose

Quality gates prevent schema defects from reaching production by enforcing rules at progressively stricter checkpoints. Without structured enforcement, naming violations, unsafe migrations, and missing documentation accumulate as technical debt. This skill provides the architecture for a layered enforcement pipeline — from local development through production deployment — covering tool orchestration, rule classification, and exception handling.

Use this skill when designing a database CI/CD pipeline, reviewing an existing enforcement architecture, classifying rules as hard-block vs advisory, or establishing break-glass exception procedures.

---

## 5-Phase Enforcement Pipeline

Quality enforcement is layered across five phases. Each phase catches progressively harder-to-fix issues.

### Phase Overview

| Phase | Environment | Feedback Time | Enforcement Level |
|-------|-------------|---------------|-------------------|
| 1. Local | Developer machine | Seconds | Advisory only |
| 2. Pre-commit | Git hooks | Seconds | Advisory + select hard blocks |
| 3. PR/CI | CI runner + ephemeral DB | Minutes | Full hard blocks + advisory |
| 4. Staging | Staging environment | Minutes | Full enforcement + behavioral tests |
| 5. Production | Production deploy pipeline | Minutes | Migration safety + drift detection |

### Phase 1: Local Development

Developers run checks manually or via editor integration.

**Tools:**
- `atlas migrate lint --env local --latest 1` — lint the most recent migration
- `sqlfluff lint` or `pg_formatter` — SQL formatting checks
- `atlas fmt` — HCL formatting for `atlas.hcl`

**Enforcement:** Advisory only. No blocks. Fast feedback encourages adoption.

### Phase 2: Pre-commit Hooks

Git pre-commit hooks run automatically before each commit.

**Recommended hooks:**
- `atlas migrate validate` — verify migration directory integrity (checksums, linear history)
- `sqlfluff lint --diff` or `diff-quality` — lint only changed SQL files
- `atlas fmt --check` — verify HCL formatting

**Enforcement:** Formatting and integrity checks can hard-block. Lint rules remain advisory to avoid slowing commits.

### Phase 3: PR/CI Pipeline

The primary enforcement layer. Runs on every pull request that touches schema-relevant files.

**Trigger files:** `**/models/**`, `**/migrations/**`, `atlas.hcl`, `**/schema*`

**Pipeline steps:**
1. Provision ephemeral database (Docker, GitHub Actions service, or cloud ephemeral)
2. `atlas migrate validate` — checksum integrity, linear history
3. `atlas migrate lint --env ci --latest N` — lint new migrations against ephemeral DB
4. `squawk` — PostgreSQL-specific migration safety analysis
5. `sqlfluff lint` — SQL style enforcement
6. Naming convention validation (custom script or Atlas analyzers)
7. Documentation coverage check (`COMMENT ON` presence)

**Enforcement:** Hard-block rules fail the pipeline. Advisory rules produce warnings in PR comments.

### Phase 4: Staging

Post-merge enforcement in a staging environment that mirrors production.

**Additional checks:**
- Behavioral tests (pgTAP) run against the staged schema
- RLS policy validation with tenant isolation proofs
- Performance regression baselines (EXPLAIN cost comparison)
- Full migration apply + rollback verification

**Enforcement:** Full enforcement. Failures block promotion to production.

### Phase 5: Production Deploy

Final gate before production migration execution.

**Checks:**
- Drift detection (catalog vs VCS comparison)
- Migration plan review (`atlas migrate apply --dry-run`)
- Lock timeout and statement timeout verification
- Replication lag pre-check

**Enforcement:** Hard-block on drift, invalid migrations, or missing timeouts.

---

## Ephemeral Database Provisioning

CI pipelines require a clean PostgreSQL instance for each run to replay migrations and validate against a real database.

### Provisioning Strategies

| Strategy | Speed | Fidelity | Cost |
|----------|-------|----------|------|
| Docker service container | Fast (seconds) | High | Free |
| GitHub Actions `services` | Fast | High | Free (within CI minutes) |
| Cloud ephemeral (RDS, Neon) | Moderate | Production-like | Per-minute billing |
| Template database (`createdb -T`) | Very fast | High | Requires persistent CI DB |

### Docker Example

```yaml
services:
  postgres:
    image: postgres:16
    env:
      POSTGRES_DB: test
      POSTGRES_USER: ci
      POSTGRES_PASSWORD: ci
    ports:
      - 5432:5432
    options: >-
      --health-cmd "pg_isready -U ci"
      --health-interval 5s
      --health-timeout 5s
      --health-retries 5
```

The ephemeral database is destroyed after each CI run, ensuring clean state.

---

## Tool Orchestration

Multiple tools serve complementary roles. No single tool covers all quality dimensions.

### Tool Responsibilities

| Tool | Role | Phase |
|------|------|-------|
| Atlas `migrate lint` | Destructive change detection, backward compatibility | Pre-commit, CI |
| Atlas `migrate validate` | Migration integrity (checksums, linear history) | Pre-commit, CI |
| Squawk | PostgreSQL migration safety (locks, CONCURRENTLY) | CI |
| SQLFluff | SQL formatting and style | Local, Pre-commit, CI |
| pgTAP | Behavioral testing (RLS, constraints, triggers) | Staging |
| Custom scripts | Naming conventions, documentation coverage | CI |

### Overlap Management

Atlas lint and Squawk overlap on migration safety (both detect missing CONCURRENTLY). When both are present:
- Use Atlas lint as the authority for Atlas-specific concerns (checksums, directive validation)
- Use Squawk as the authority for PostgreSQL-specific safety (lock analysis, type safety)
- Suppress duplicate rules in one tool to avoid conflicting messages

---

## Rule Classification

Every enforced rule is classified as either hard-block (fails the pipeline) or advisory (produces a warning).

### Classification Criteria

| Factor | Hard-Block | Advisory |
|--------|-----------|----------|
| Data loss risk | Yes | — |
| Production downtime risk | Yes | — |
| Security vulnerability | Yes | — |
| Style / formatting | — | Yes |
| Documentation gap | — | Yes |
| Performance suggestion | — | Yes (unless proven regression) |

### Classification Application

Hard-block and advisory classifications are consumed by the `enforcement-policy` reference, which provides the complete rule matrix. When reviewing a pipeline, verify that each rule's classification matches the reference.

---

## MVQG: Minimum Viable Quality Gate

Teams adopting quality gates incrementally should start with MVQG — the smallest set of hard blocks that prevent the highest-impact defects.

### MVQG Rules (Start Here)

| Rule | Tool | Why |
|------|------|-----|
| Migration integrity | `atlas migrate validate` | Prevents corrupted migration history |
| Destructive change detection | `atlas migrate lint` (DS codes) | Prevents accidental data loss |
| CONCURRENTLY enforcement | Squawk or Atlas lint (PG codes) | Prevents production lock-outs |
| Missing `lock_timeout` | Squawk | Prevents unbounded lock waits |

### High-Maturity Additions

Once MVQG is stable, add incrementally:

| Rule | Tool | Gate Level |
|------|------|------------|
| Naming convention compliance | Custom script | Hard-block |
| Missing RLS on tenant tables | Custom analyzer | Hard-block |
| `COMMENT ON` coverage | Custom script | Advisory |
| SQL formatting | SQLFluff | Advisory |
| Query plan regression | RegreSQL | Advisory |
| FK index coverage | Custom script | Advisory (promote after baseline) |

---

## Break-Glass Exception Procedures

Emergency changes sometimes require bypassing quality gates. The break-glass procedure ensures exceptions are tracked, justified, and reconciled.

### When Break-Glass Applies

- Production incident requiring immediate schema change
- Hotfix for data corruption that cannot wait for CI
- Vendor-mandated emergency migration

### Procedure

1. **Declare:** Notify the team that a break-glass change is in progress
2. **Execute:** Apply the change directly, bypassing CI
3. **Document:** Create an incident record with the change details and justification
4. **Backport:** Within 24 hours, commit the change to VCS and run it through CI retroactively
5. **Reconcile:** Verify no drift exists between production and VCS after backport
6. **Review:** Post-incident review to determine if the gate gap can be prevented

### Tracking

Every break-glass event should produce:
- An incident record (linked to the migration)
- A backport PR (passing all gates retroactively)
- A drift reconciliation confirmation

---

## Anti-Patterns

### 1. Enforcement Without Ephemeral Databases

Running `atlas migrate lint` without replaying migrations against a real database. Lint catches syntax and policy issues but misses runtime failures (FK references to non-existent tables, type cast failures).

### 2. All Rules as Hard-Block

Making every rule a hard-block from day one. Developers bypass the system (force-merge, disable hooks) because the friction is too high. Start with MVQG and promote rules incrementally.

### 3. No Break-Glass Procedure

Assuming quality gates will never need to be bypassed. When an emergency arrives, developers improvise — often leaving production and VCS in a drifted state.

### 4. Duplicate Tool Coverage Without Suppression

Running both Atlas lint and Squawk without suppressing overlapping rules. Developers see contradictory or duplicated messages and lose trust in the pipeline.

### 5. Skipping Staging Phase

Jumping from CI directly to production. Behavioral tests (RLS proofs, performance baselines) cannot run in CI without a production-like dataset.

---

## Examples

### Example 1: MVQG Pipeline Configuration

A team adopting quality gates for the first time. Start with the minimum enforcement set.

**CI pipeline:**
1. `atlas migrate validate` — hard-block on integrity failure
2. `atlas migrate lint --env ci --latest 1` — hard-block on DS and PG codes only
3. Squawk — hard-block on missing `lock_timeout`

**All other rules:** Advisory warnings in PR comments. No blocks.

**Result:** High-impact defects are caught. Developer friction is low. Gate adoption succeeds.

### Example 2: High-Maturity Pipeline

A mature team with full enforcement.

**CI pipeline:**
1. All MVQG rules (hard-block)
2. Naming convention validation (hard-block)
3. RLS coverage for tenant tables (hard-block)
4. SQLFluff formatting (advisory)
5. `COMMENT ON` coverage (advisory)

**Staging pipeline:**
6. pgTAP behavioral tests (hard-block)
7. RegreSQL query plan baselines (advisory)

**Production deploy:**
8. Drift detection (hard-block)
9. Dry-run migration plan review

### Example 3: Break-Glass Event

A production incident requires adding an index immediately.

1. DBA declares break-glass, applies `CREATE INDEX CONCURRENTLY` directly to production
2. Documents the change in the incident channel
3. Within 24 hours, commits the migration file to VCS with proper `-- atlas:txmode none` annotation
4. CI runs and passes on the backport PR
5. Drift check confirms production matches VCS
6. Post-incident review notes: "Index was needed urgently due to query timeout spike. No gate gap — the change would have passed CI."
