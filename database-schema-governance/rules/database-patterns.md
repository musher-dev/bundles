---
paths:
  - "**/database/**/*.py"
  - "**/models/**/*.py"
  - "**/migrations/**/*.sql"
---

# Database Patterns

These rules apply to all database-related Python code.

## Primary Keys

- **MUST** use UUID (not auto-increment integers)
- Generate with `uuid.uuid4()` or database default

## Timestamps

- `created_at`: Immutable, set on insert, **MUST NOT** be nullable
- `updated_at`: Nullable, set on update only
- `deleted_at`: Nullable, enables soft delete

## Soft Deletes

- **MUST** use `deleted_at` field instead of hard deletes
- Repositories **MUST** filter `WHERE deleted_at IS NULL` by default
- Use explicit methods for queries that include deleted records

## Naming Convention

All constraints, indexes, and keys use a deterministic naming convention. Define this
in your ORM base module to prevent migration drift.

| Type | Template | Example |
|------|----------|---------|
| PK | `pk_%(table_name)s` | `pk_jobs` |
| FK | `fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s` | `fk_jobs_queue_id_queues` |
| UQ | `uq_%(table_name)s_%(column_0_name)s` | `uq_users_email` |
| CK | `ck_%(table_name)s_%(constraint_name)s` | `ck_jobs_status` |
| IX | `ix_%(column_0_N_label)s` | `ix_jobs_workspace_id` |

### CheckConstraint names — no prefix

The `ck` convention adds the `ck_<table>_` prefix automatically. ORM code **MUST** use
bare semantic names:

```python
# Correct — convention produces ck_jobs_status
CheckConstraint("status IN ('queued', 'running')", name="status")

# Wrong — produces ck_jobs_ck_jobs_status (double prefix)
CheckConstraint("status IN ('queued', 'running')", name="ck_jobs_status")
```

## Pagination Indexes

Tables with cursor-based keyset pagination **MUST** have a composite index covering the `ORDER BY` columns plus the `id` tiebreaker. This ensures efficient seek queries without full-table scans.

### Index Pattern

```python
# For ORDER BY created_at DESC, id DESC WHERE deleted_at IS NULL
Index(
    "ix_{table}_ws_created_page",
    "workspace_id", "created_at", "id",
    postgresql_ops={"created_at": "DESC", "id": "DESC"},
    postgresql_where=text("deleted_at IS NULL"),
)
```

### Rules

- **MUST** include all `ORDER BY` columns in the index, in the same order
- **MUST** include `id` as the final column (tiebreaker)
- **MUST** use `postgresql_ops` to match sort direction when not ASC (default)
- **SHOULD** add a partial `WHERE` predicate matching the query filter (e.g., `deleted_at IS NULL`)
- **MUST** prefix with the scoping column (usually `workspace_id`) so the planner can use it as a leading key

### Naming Convention

`ix_{table}_{scope}_{sort_field}_page` — e.g., `ix_jobs_ws_created_page`, `ix_labels_ws_slug_page`.

### Examples

| Sort Pattern | Index Columns | Ops |
|-------------|---------------|-----|
| `ORDER BY created_at DESC, id DESC` | `(workspace_id, created_at, id)` | `{"created_at": "DESC", "id": "DESC"}` |
| `ORDER BY slug ASC, id ASC` | `(workspace_id, slug, id)` | (none — ASC is default) |
| `ORDER BY priority ASC, id ASC` | `(workspace_id, priority, id)` | (none — ASC is default) |
| `ORDER BY received_at DESC, id DESC` | `(source_id, received_at, id)` | `{"received_at": "DESC", "id": "DESC"}` |

## Transaction Safety

Transaction boundaries live in the **presentation layer only** (routers/workers).

| Layer | `flush()` | `commit()` |
|-------|-----------|------------|
| Domain | No | No |
| Application (handlers) | Yes | **Never** |
| Infrastructure (repos) | Yes | **Never** |
| Presentation (routers) | Yes | Yes |

- The session context manager should auto-commit on success and auto-rollback on exception
- Scoped sessions require explicit commit from the caller

## Adding a New Model

1. Create model file in appropriate `models/` directory
2. Export in `__init__.py` — add to both imports AND `__all__`
3. Generate migration: `atlas migrate diff --env local`
4. Review generated SQL in `migrations/`
5. Apply migration: `atlas migrate apply --env local`

## Store Pattern

Stores work with **generic dictionaries**, not domain objects.

```python
# Correct: Store accepts dict
async def save_record(self, record_id: str, data: dict[str, Any]) -> None:
    ...

# Wrong: Store imports domain objects
from domain.job.models import JobState  # FORBIDDEN
```

Domain object conversion happens in the API layer (Adapter Pattern).
