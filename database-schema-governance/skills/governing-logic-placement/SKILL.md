---
name: governing-logic-placement
version: 1.0.0
user-invocable: false
description: >-
  Decide whether business logic belongs in the database layer (constraints, triggers,
  RLS) or application layer (workflows, dynamic rules) using a structured decision
  matrix. Covers triggers, stored procedures, CHECK constraints, EXCLUDE constraints,
  partial unique indexes, deferrable constraints, composite CHECK constraints, TOCTOU
  race conditions, and hybrid patterns for audit trails. Use when debating logic
  placement or reviewing schemas with embedded business rules. Triggered by: logic
  placement, database layer, application layer, trigger, stored procedure, CHECK
  constraint, RLS, business logic, TOCTOU, EXCLUDE constraint, partial unique,
  deferrable constraint.
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Logic Placement Governance

Decision framework for placing business logic in the database layer, application layer, or a hybrid of both.

## Purpose

The boundary between database logic and application logic is the most consequential architectural decision in a data-driven system. Logic placed in the wrong layer creates either fragile data (rules that can be bypassed) or rigid schemas (business rules baked into DDL that require migrations to change). This skill provides the decision matrix and guidelines for making placement decisions consistently.

Use this skill when deciding where to enforce a business rule, reviewing schemas that contain triggers or stored procedures, or investigating data integrity issues that suggest a placement mismatch.

---

## Decision Matrix

| Logic Type | Place In | Rationale |
|------------|----------|-----------|
| Referential integrity | Database (FK constraints) | Cannot be bypassed by any client; enforced at storage level |
| Domain value constraints | Database (CHECK constraints) | Invariants that must hold regardless of application version |
| Uniqueness guarantees | Database (UNIQUE constraints/indexes) | Only the database can guarantee uniqueness under concurrency |
| Row-level security | Database (RLS policies) | Tenant isolation must not depend on application correctness |
| Concurrency control | Database (advisory locks, serializable isolation) | Application-level locks cannot span database transactions |
| Audit trail capture | Hybrid (trigger + application context) | Trigger ensures no write escapes logging; application provides actor context |
| Workflow orchestration | Application | State machines, approval flows, multi-step processes change frequently |
| Dynamic business rules | Application | Rules that change without deployment (feature flags, config-driven) |
| Notification/messaging | Application | Side effects that depend on external services |
| Complex calculations | Application | Business formulas that change with product requirements |
| Data transformation | Application | ETL, format conversion, enrichment logic |

---

## Database Layer: When and How

### CHECK Constraints

Use CHECK constraints for invariants that must hold regardless of which application, migration script, or manual SQL session touches the data.

**Appropriate:**
```sql
-- Value must be positive
ALTER TABLE invoices ADD CONSTRAINT check_invoices_amount_positive
    CHECK (amount_cents > 0);

-- Enum-like restriction
ALTER TABLE api_keys ADD CONSTRAINT check_api_keys_principal_type
    CHECK (principal_type IN ('membership', 'service_account'));

-- Temporal ordering
ALTER TABLE subscriptions ADD CONSTRAINT check_subscriptions_date_order
    CHECK (ends_at > starts_at);
```

**Not appropriate:** Business rules that change with product iterations (e.g., "maximum 5 projects per organization" — this will change).

### Foreign Key Constraints

Every relationship between tables must be enforced by a FK constraint. No exceptions.

**Cascade rules:**
| Scenario | ON DELETE | Rationale |
|----------|-----------|-----------|
| Child cannot exist without parent | `CASCADE` | Delete children when parent is removed |
| Child has independent lifecycle | `RESTRICT` (default) | Prevent orphaning; require explicit cleanup |
| Soft-delete pattern | `NO ACTION` + application logic | Parent soft-deletes don't cascade |
| Nullable reference | `SET NULL` | Optional relationship; absence is valid |

### Row-Level Security (RLS)

RLS enforces tenant isolation at the database level, making cross-tenant data access structurally impossible even if the application has a bug.

```sql
ALTER TABLE api_keys ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON api_keys
    USING (org_id = current_setting('app.current_org_id')::bigint);
```

**When to use RLS:**
- Multi-tenant systems where tenant data must never leak
- Compliance requirements (SOC2, HIPAA) that mandate defense-in-depth
- Systems where multiple application services access the same database

**When NOT to use RLS:**
- Single-tenant applications
- Read replicas serving analytics (where cross-tenant queries are legitimate)
- Performance-critical paths where RLS overhead is measured and unacceptable

---

## Application Layer: When and How

### Workflow Logic

Multi-step processes with conditional branching, human approvals, or external service calls belong in the application.

**Examples:**
- User registration flow (create user → send verification email → activate on confirm)
- Payment processing (validate → charge → update ledger → send receipt)
- Approval workflows (submit → review → approve/reject → execute)

**Why not database:** Triggers cannot call external services, cannot implement timeouts, and create invisible control flow that is difficult to debug and test.

### Dynamic Business Rules

Rules that change without a database migration belong in the application, driven by configuration or feature flags.

**Examples:**
- Rate limits (may change per plan tier)
- Validation rules (minimum password length, max upload size)
- Feature access (which plans can use which features)

**Why not database:** CHECK constraints and triggers require DDL changes (migrations) to modify. Application-layer rules can change with a configuration update or deployment.

---

## Hybrid Patterns

### Audit Trails

Audit logging is the canonical hybrid pattern. The database trigger ensures no write escapes logging; the application provides context (who did it, why).

**Pattern:**
1. Application sets session variables with actor context before each transaction:
   ```sql
   SET LOCAL app.current_user_id = '123';
   SET LOCAL app.request_id = 'req_abc';
   ```
2. Database trigger fires on INSERT/UPDATE/DELETE, reading session variables:
   ```sql
   CREATE FUNCTION audit_trigger() RETURNS TRIGGER AS $$
   BEGIN
       INSERT INTO audit_logs (table_name, operation, row_id, actor_id, request_id, old_data, new_data)
       VALUES (TG_TABLE_NAME, TG_OP, NEW.id,
               current_setting('app.current_user_id', true),
               current_setting('app.request_id', true),
               row_to_json(OLD), row_to_json(NEW));
       RETURN NEW;
   END;
   $$ LANGUAGE plpgsql;
   ```

**Why hybrid:** A pure application approach lets writes bypass logging (direct SQL, migration scripts, admin tools). A pure database approach cannot capture application-level context (who initiated the request, from which endpoint).

### Updated-At Timestamps

Database triggers for `updated_at` ensure the timestamp is always accurate, regardless of whether the application remembers to set it.

```sql
CREATE FUNCTION set_updated_at() RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_set_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

---

## TOCTOU Race Conditions

**Time-Of-Check-Time-Of-Use (TOCTOU)** is the most common argument for database-layer logic. The pattern:

1. Application reads data (CHECK)
2. Application makes a decision based on the read
3. Application writes based on the decision (USE)
4. Between steps 1 and 3, another transaction modifies the data

**Example:** "A user can have at most 5 API keys."

**Vulnerable application-only approach:**
```python
# Thread A and Thread B execute simultaneously
count = db.query("SELECT COUNT(*) FROM api_keys WHERE user_id = ?", user_id)  # Both read 4
if count < 5:
    db.execute("INSERT INTO api_keys ...")  # Both insert → user has 6 keys
```

**Database-enforced solution:**
```sql
-- Option 1: Serializable isolation
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- (repeat the check-then-insert; PostgreSQL will abort one transaction on conflict)

-- Option 2: Advisory lock
SELECT pg_advisory_xact_lock(hashtext('api_keys_' || user_id::text));
-- (then check and insert within the lock)

-- Option 3: Exclusion constraint (when applicable)
-- (only works for range/overlap constraints, not count limits)
```

**Rule of thumb:** If a business rule depends on the current state of other rows in the same table, it is vulnerable to TOCTOU and must be enforced at the database level or within a serialized transaction.

---

## Trigger Guidelines

| Guideline | Rule |
|-----------|------|
| Complexity | Triggers should be < 20 lines; complex logic belongs in the application |
| Side effects | Triggers must NEVER call external services, send emails, or write to files |
| Visibility | Every trigger must have a COMMENT ON explaining its purpose |
| Testing | Triggers must be testable via SQL; if they're hard to test, they're too complex |
| Cascading | Avoid triggers that fire other triggers (cascading triggers are nearly impossible to reason about) |
| Performance | Triggers add latency to every affected DML statement; measure before adding |

### Stored Procedure Constraints

Stored procedures are appropriate only for:
- **Atomic multi-statement operations** that must succeed or fail as a unit
- **Performance-critical operations** where round-trip latency is the bottleneck
- **Database-internal utilities** (maintenance scripts, data migration helpers)

Stored procedures are NOT appropriate for:
- Business logic that changes with product requirements
- Logic that requires external service calls
- Complex control flow (if/else chains, loops with conditional breaks)

---

## EXCLUDE Constraints

EXCLUDE constraints prevent overlapping or conflicting values using operator-based exclusion. They are the database-native solution for temporal and range overlap prevention.

### Syntax and Requirements

```sql
-- Requires the btree_gist extension
CREATE EXTENSION IF NOT EXISTS btree_gist;

-- Prevent overlapping bookings for the same room
ALTER TABLE bookings ADD CONSTRAINT excl_bookings_no_overlap
    EXCLUDE USING GIST (
        room_id WITH =,
        tstzrange(starts_at, ends_at) WITH &&
    );
```

### Requirements

| Requirement | Details |
|-------------|---------|
| GiST index | EXCLUDE constraints require a GiST index (created automatically) |
| `btree_gist` extension | Required for combining equality (`=`) with range operators (`&&`) |
| Range types | Use `tstzrange`, `daterange`, `int4range`, etc. for overlap detection |

### Use Cases

| Scenario | EXCLUDE Definition |
|----------|-------------------|
| Room/resource booking | `EXCLUDE (room_id WITH =, tstzrange(start, end) WITH &&)` |
| Employee scheduling | `EXCLUDE (employee_id WITH =, tstzrange(shift_start, shift_end) WITH &&)` |
| IP address allocation | `EXCLUDE (network WITH &&)` using `inet`/`cidr` ranges |
| Price validity periods | `EXCLUDE (product_id WITH =, daterange(valid_from, valid_to) WITH &&)` |

**Decision rule:** If the business rule is "no two rows with the same X can have overlapping Y," use an EXCLUDE constraint. Application-level checks are vulnerable to TOCTOU race conditions.

---

## Partial Unique Indexes

Partial unique indexes enforce uniqueness only on rows matching a predicate. They solve the common problem of maintaining uniqueness while supporting soft deletes.

### Soft-Delete-Compatible Uniqueness

```sql
-- Standard UNIQUE constraint fails with soft deletes:
-- Two rows with same email, one deleted — constraint violation

-- Partial unique index: uniqueness only among active records
CREATE UNIQUE INDEX uq_users_email_active ON users (email)
    WHERE deleted_at IS NULL;
```

### Conditional Uniqueness Patterns

| Pattern | Index | Use Case |
|---------|-------|----------|
| Active record uniqueness | `UNIQUE (email) WHERE deleted_at IS NULL` | Soft-delete-compatible unique constraints |
| Status-scoped uniqueness | `UNIQUE (org_id, slug) WHERE status = 'published'` | Multiple drafts allowed, only one published |
| Non-null uniqueness | `UNIQUE (external_id) WHERE external_id IS NOT NULL` | Optional external ID that must be unique when present |

**Decision rule:** When uniqueness should apply only to a subset of rows, use a partial unique index instead of adding application-level checks. The database enforces it under concurrency.

---

## Deferrable Constraints

Deferrable constraints delay enforcement until transaction commit, enabling operations that would otherwise violate constraints mid-transaction.

### Syntax

```sql
-- Create a deferrable FK constraint
ALTER TABLE employees ADD CONSTRAINT fk_employees_manager_id
    FOREIGN KEY (manager_id) REFERENCES employees(id)
    DEFERRABLE INITIALLY DEFERRED;

-- Or defer at runtime:
ALTER TABLE employees ADD CONSTRAINT fk_employees_manager_id
    FOREIGN KEY (manager_id) REFERENCES employees(id)
    DEFERRABLE INITIALLY IMMEDIATE;

-- Defer within a transaction:
SET CONSTRAINTS fk_employees_manager_id DEFERRED;
```

### INITIALLY DEFERRED vs INITIALLY IMMEDIATE

| Mode | Behavior | Use When |
|------|----------|----------|
| `INITIALLY IMMEDIATE` | Checked after each statement; can be deferred per-transaction | Default safe behavior; opt-in deferral when needed |
| `INITIALLY DEFERRED` | Checked at commit; always deferred | Circular references or bulk operations that always need deferral |

### Use Cases

| Scenario | Why Deferrable |
|----------|---------------|
| Circular FK insertion | Employee A manages Employee B, B manages A — both need to exist before FKs validate |
| Bulk loading with FK dependencies | Insert parents and children in arbitrary order within a transaction |
| Tree/graph restructuring | Moving nodes in a hierarchy where parent references temporarily violate constraints |

**Decision rule:** Use `DEFERRABLE INITIALLY IMMEDIATE` for FKs that occasionally need deferred validation. Use `INITIALLY DEFERRED` only for constraints that are always validated at commit (e.g., self-referential FKs).

---

## Composite CHECK Constraints

CHECK constraints can enforce multi-column invariants — relationships between columns that must always hold.

### Beyond Temporal Ordering

```sql
-- Temporal ordering (common)
CHECK (ends_at > starts_at)

-- Conditional requirement: if status is 'completed', completed_at must be set
CHECK (status != 'completed' OR completed_at IS NOT NULL)

-- Mutual exclusivity: exactly one of two FKs must be set
CHECK (
    (user_id IS NOT NULL AND service_account_id IS NULL) OR
    (user_id IS NULL AND service_account_id IS NOT NULL)
)

-- Range validation: min must be less than max
CHECK (min_quantity < max_quantity)

-- Percentage columns must sum to 100
CHECK (split_pct_a + split_pct_b + split_pct_c = 100)
```

### Naming Convention

```sql
-- Pattern: ck_{table}_{description}
CONSTRAINT ck_subscriptions_date_order CHECK (ends_at > starts_at)
CONSTRAINT ck_api_keys_principal_type CHECK (...)
CONSTRAINT ck_splits_sum_100 CHECK (split_pct_a + split_pct_b + split_pct_c = 100)
```

**Decision rule:** If an invariant involves multiple columns in the same row and must hold regardless of which application or script modifies the data, use a composite CHECK constraint.

---

## Anti-Patterns

### 1. God Trigger

A trigger that contains the entire business logic for a table — validation, transformation, notification, logging. Impossible to test, debug, or modify independently.

### 2. Application-Only Uniqueness

Checking uniqueness in the application (`SELECT COUNT(*) ... INSERT`) without a database UNIQUE constraint. Guaranteed to produce duplicates under concurrent load.

### 3. Trigger-Based Workflows

Using triggers to orchestrate multi-step processes (insert in table A triggers insert in table B which triggers update in table C). Creates invisible, untestable control flow.

### 4. RLS as Only Security Layer

Relying exclusively on RLS without application-level authorization checks. RLS policies depend on session variables being set correctly — a missing `SET LOCAL` bypasses all protection.

---

## Examples

### Example 1: Where to Put "Max 5 API Keys Per User"

**Analysis:** This rule depends on the count of existing rows and is vulnerable to TOCTOU under concurrent requests.

**Recommendation:** Database layer. Use an advisory lock or serializable transaction:
```sql
-- Within the application's transaction:
SELECT pg_advisory_xact_lock(hashtext('user_api_keys_' || $user_id::text));
SELECT COUNT(*) FROM api_keys WHERE user_id = $user_id AND revoked_at IS NULL;
-- If count < 5, proceed with INSERT
```

### Example 2: Where to Put "Send Email on Password Change"

**Analysis:** Email sending is an external side effect that can fail independently of the database transaction.

**Recommendation:** Application layer. The database records the password change; the application sends the email after commit. If the email fails, the password change should still succeed.

### Example 3: Where to Put "Tenant Data Isolation"

**Analysis:** Cross-tenant data access is a security boundary that must hold even if the application has bugs.

**Recommendation:** Hybrid. RLS policies enforce isolation at the database level. Application middleware sets the `app.current_org_id` session variable. Application authorization checks provide the first line of defense. RLS is the safety net.
