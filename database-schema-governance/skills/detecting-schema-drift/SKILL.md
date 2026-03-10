---
name: detecting-schema-drift
version: 1.0.0
user-invocable: false
description: >-
  Detect and reconcile production schema drift between the database catalog and
  version-controlled schema definitions. Covers catalog-vs-VCS comparison, Atlas
  Schema Monitoring, hourly drift checks, emergency hotfix reconciliation,
  dynamic partition exclusions, and break-glass backport procedures. Use when
  investigating production drift, setting up drift detection, or reconciling
  out-of-band changes. Triggered by: schema drift, catalog divergence, production
  drift, out-of-band changes, drift detection, break-glass, schema monitoring.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Schema Drift Detection

Framework for detecting, investigating, and reconciling divergence between production database schemas and version-controlled schema definitions.

## Purpose

Schema drift occurs when the production database catalog diverges from the schema defined in version control. This happens through emergency hotfixes applied directly to production, manual DBA interventions, dynamic partition creation, or failed migration rollbacks. Undetected drift undermines the guarantee that VCS is the source of truth — migrations may fail silently, CI checks become meaningless, and disaster recovery from VCS produces a different schema than production.

Use this skill when setting up drift detection infrastructure, investigating a drift alert, reconciling an out-of-band production change, or designing partition exclusion rules for dynamic schemas.

---

## Drift Sources

### Common Causes

| Source | Frequency | Risk Level | Example |
|--------|-----------|------------|---------|
| Emergency hotfix | Occasional | High | DBA adds index directly to fix timeout spike |
| Failed migration rollback | Rare | Critical | Partial migration leaves schema in intermediate state |
| Dynamic partitions | Regular | Low (if excluded) | Cron job creates monthly partitions not in VCS |
| Manual DBA intervention | Occasional | High | Column type changed to fix data issue |
| ORM auto-migration | Depends on setup | Medium | ORM applies changes not captured in Atlas migrations |
| Extension updates | Rare | Low | PostgreSQL extension upgrade changes internal schemas |

### Why Drift Is Dangerous

1. **Silent migration failures:** A migration expects column A but production already has column B
2. **Recovery divergence:** Restoring from VCS produces a different schema than production had
3. **CI false confidence:** CI passes because it validates against VCS, not production reality
4. **Cascading drift:** One unreconciled change leads to more ad-hoc fixes to work around it

---

## Catalog-vs-VCS Comparison

The fundamental drift detection operation: compare the production database catalog against the schema state that VCS migrations would produce.

### Using Atlas

```bash
# Compare production schema against the desired state from migrations
atlas schema diff \
  --from "postgres://prod-readonly:password@prod-host:5432/mydb?sslmode=require" \
  --to "file://migrations" \
  --dev-url "postgres://dev:dev@localhost:5432/dev?sslmode=disable"
```

**How it works:**
1. Atlas connects to production (read-only credentials) and reads the catalog
2. Atlas replays all migrations against the dev database to build the expected state
3. Atlas computes the diff between production catalog and expected state
4. Any diff output indicates drift

### Interpreting Results

| Result | Meaning | Action |
|--------|---------|--------|
| Empty diff | No drift | Production matches VCS |
| `CREATE INDEX` | Production missing an index | Migration may not have been applied |
| `DROP INDEX` | Production has an extra index | Out-of-band index addition |
| `ALTER TABLE ADD COLUMN` | Production missing a column | Migration not applied |
| `ALTER TABLE DROP COLUMN` | Production has an extra column | Out-of-band column addition |
| `ALTER TABLE ALTER COLUMN` | Column type/constraint mismatch | Drift in type or nullability |

### Read-Only Safety

Always use read-only credentials for drift detection against production. The comparison is a read operation — it should never have write access.

---

## Automated Drift Checks

### Scheduling

| Check Frequency | Use Case |
|-----------------|----------|
| Every PR (CI) | Compare migration directory integrity (not against production) |
| Hourly | Production drift detection (automated alert) |
| On deploy | Pre-deployment drift verification (hard-block) |
| Daily | Comprehensive drift audit with detailed reporting |

### Atlas Schema Monitoring

Atlas Pro provides Schema Monitoring — a managed service that periodically compares the production catalog against the expected schema and alerts on drift.

**Key features:**
- Scheduled comparisons (configurable interval)
- Alert integration (Slack, PagerDuty, webhook)
- Historical drift timeline
- Exclusion rules for dynamic objects

**Constraint:** Requires Atlas Pro license (v0.38+). For open-source Atlas, implement drift checks via CI cron jobs.

### CI Cron Job Pattern

For teams without Atlas Pro, implement drift detection as a scheduled CI job:

```yaml
# .github/workflows/drift-check.yml
name: Schema Drift Check
on:
  schedule:
    - cron: '0 * * * *'  # Hourly
  workflow_dispatch: {}

jobs:
  drift-check:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: dev
          POSTGRES_USER: dev
          POSTGRES_PASSWORD: dev
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v4
      - uses: ariga/setup-atlas@v0
      - name: Check for drift
        env:
          PROD_URL: ${{ secrets.PROD_READONLY_URL }}
        run: |
          atlas schema diff \
            --from "$PROD_URL" \
            --to "file://migrations" \
            --dev-url "postgres://dev:dev@localhost:5432/dev?sslmode=disable" \
            > drift.txt 2>&1
          if [ -s drift.txt ]; then
            echo "::error::Schema drift detected"
            cat drift.txt
            exit 1
          fi
```

---

## Dynamic Partition Exclusions

Dynamic partitions (created by cron jobs or application logic) are expected to exist in production but not in VCS. These must be excluded from drift detection to avoid false positives.

### Exclusion Strategies

| Strategy | Mechanism | Pros | Cons |
|----------|-----------|------|------|
| Schema-based exclusion | Place dynamic partitions in a separate schema | Clean separation | Requires schema-aware application code |
| Naming convention | Exclude tables matching `*_p20*` pattern | Simple | Relies on consistent naming |
| Atlas exclusion config | `exclude` block in `atlas.hcl` | Native support | Requires Atlas configuration |

### Atlas Exclusion Configuration

```hcl
# atlas.hcl
env "drift-check" {
  src = "file://migrations"
  url = getenv("PROD_URL")
  dev = "docker://postgres/16/dev"

  diff {
    skip {
      # Exclude dynamically created partitions
      drop_table = true
    }
  }

  exclude = [
    "public.*_p20*",        # Monthly partitions (events_p2025_01, etc.)
    "partitions.*",         # Entire partitions schema
    "pg_partman.*",         # pg_partman management tables
  ]
}
```

### Partition Naming Convention

For drift exclusion to work reliably, partitions must follow a consistent naming pattern:

| Parent Table | Partition Pattern | Example |
|-------------|-------------------|---------|
| `events` | `events_p{YYYY}_{MM}` | `events_p2025_03` |
| `audit_logs` | `audit_logs_p{YYYY}_q{Q}` | `audit_logs_p2025_q1` |
| `metrics` | `metrics_p{YYYY}_{MM}_{DD}` | `metrics_p2025_03_10` |

---

## Emergency Hotfix Reconciliation

When a break-glass change is applied directly to production, reconciliation ensures VCS catches up.

### Reconciliation Workflow

1. **Detect:** Hourly drift check fires an alert
2. **Investigate:** Run `atlas schema diff` to identify the exact changes
3. **Classify:** Determine if the change was authorized (break-glass) or unauthorized
4. **Backport:** Create a migration file that reproduces the production change
5. **Validate:** Run the backport migration against the dev database
6. **Verify:** Re-run drift check to confirm production and VCS match

### Backport Migration Pattern

```sql
-- Migration: 20250310120000_backport_hotfix_idx_orders_status.sql
-- Backport of emergency index added during incident INC-1234
-- Original change applied directly to production on 2025-03-09

-- atlas:txmode none

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_orders_status
    ON orders (status)
    WHERE status IN ('pending', 'processing');
```

**Key elements:**
- `IF NOT EXISTS` — idempotent, safe to apply on environments that already have the change
- Comment with incident reference — traceability
- Proper Atlas directives — passes CI like any other migration

### Post-Reconciliation Verification

After the backport PR merges and deploys:

```bash
# Verify drift is resolved
atlas schema diff \
  --from "$PROD_URL" \
  --to "file://migrations" \
  --dev-url "postgres://dev:dev@localhost:5432/dev?sslmode=disable"

# Expected: empty output (no drift)
```

---

## Anti-Patterns

### 1. No Drift Detection

Assuming VCS is always in sync with production. Drift accumulates silently until a migration fails in production.

### 2. Drift Detection Without Exclusions

Running drift checks without excluding dynamic partitions. Every cron-created partition triggers a false alarm, causing alert fatigue and eventual ignore-all behavior.

### 3. Manual Reconciliation Without Migration

Fixing drift by modifying VCS schema definitions directly (editing the desired state) instead of creating a proper migration. This skips the review, lint, and integrity checks that migrations go through.

### 4. Write Credentials for Drift Checks

Using read-write credentials for the drift detection job. A misconfigured `atlas schema apply` (instead of `schema diff`) could modify production.

### 5. Ignoring Drift Alerts

Treating drift alerts as informational rather than actionable. Every drift alert should be investigated and resolved within 24 hours. Persistent unresolved drift degrades trust in the entire migration system.

---

## Examples

### Example 1: Setting Up Hourly Drift Detection

A team wants to detect production drift automatically.

1. Create a read-only PostgreSQL role for drift checks
2. Store the connection string as a CI secret (`PROD_READONLY_URL`)
3. Add the drift-check CI cron job (see "CI Cron Job Pattern" above)
4. Configure partition exclusions in `atlas.hcl`
5. Set up alert routing (Slack channel for drift notifications)
6. Document the escalation path for drift alerts

### Example 2: Investigating a Drift Alert

The hourly drift check fires. The diff output shows:

```sql
ALTER TABLE "orders" ADD COLUMN "priority" integer;
CREATE INDEX "idx_orders_priority" ON "orders" ("priority");
```

**Investigation:**
- Check incident channels — a DBA added `priority` during an incident 6 hours ago
- Verify the change is intentional and correct
- Create a backport migration with `IF NOT EXISTS`
- Merge the backport PR after CI passes
- Verify the next drift check is clean

### Example 3: Partition Exclusion Setup

A team uses pg_partman to create monthly partitions for the `events` table.

**Problem:** Every month, the drift check alerts on new partitions.

**Solution:**
1. Ensure partitions follow the naming convention: `events_p{YYYY}_{MM}`
2. Add `"public.events_p*"` to the Atlas `exclude` list
3. Verify the drift check ignores partitions but still detects real drift
4. Test by introducing a deliberate column change and confirming it triggers an alert
