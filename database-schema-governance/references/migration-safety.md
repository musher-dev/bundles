# Migration Safety

Atlas provides layered protection: diff policies prevent generating dangerous SQL,
lint policies catch risky patterns in generated migrations, and CI validation blocks
PRs when migration integrity or drift checks fail.

## Safety Layers

### Layer 1: Diff Policies

Configured in the `diff` block of `atlas.hcl`. These prevent Atlas from *generating*
certain operations in the first place.

```hcl
diff {
  skip {
    drop_schema = true    # Never generate DROP SCHEMA
  }
  concurrent_index {
    create = true         # Always use CREATE INDEX CONCURRENTLY
  }
}
```

If you have `skip.drop_schema = true`, Atlas will never produce a `DROP SCHEMA`
statement, even if the models suggest one. This is a hard guardrail.

### Layer 2: Lint Policies

Configured in the `lint` block of `atlas.hcl`. These analyze generated SQL *after*
it's written and flag dangerous patterns.

```hcl
lint {
  destructive {
    error = true          # Block DROP TABLE, DROP COLUMN
  }
  data_depend {
    error = true          # Block NOT NULL without DEFAULT, type changes
  }
  concurrent_index {
    error = true          # Block non-concurrent index on existing tables
  }
}
```

Lint runs as a separate step after `diff`. A migration can be generated but still fail
lint.

### Layer 3: CI Validation

A migration CI gate should run on PRs and enforce:

1. **Drift check** — compare committed migrations against the desired schema and fail
   if they diverge
2. **Integrity check** (`atlas migrate validate`) — verify migration directory integrity
   (checksums, linear history)

## What Gets Blocked

| Scenario | Lint Code | Blocked By | Resolution |
|----------|-----------|------------|------------|
| Drop a table | DS102 | Lint | Use `-- atlas:nolint DS102` with justification |
| Drop a column | DS103 | Lint | Use `-- atlas:nolint DS103` with justification |
| Add NOT NULL column without default | MF102 | Lint | Add a `DEFAULT` value or use multi-stage migration |
| Add unique constraint to populated column | MF101 | Lint | Verify no duplicates exist, then `-- atlas:nolint MF101` |
| Change column type | MF103 | Lint | Verify the cast is safe, then `-- atlas:nolint MF103` |
| Non-concurrent index on existing table | PG101 | Lint | Diff policy auto-generates `CONCURRENTLY`, but verify |
| Concurrent index without `txmode none` | PG102 | Lint | Add `-- atlas:txmode none` at top of migration |
| Rename table | BC101 | Lint | Use expand-contract pattern instead |
| Rename column | BC102 | Lint | Use expand-contract pattern instead |
| Tampered migration file | — | Validate | Run `atlas migrate hash` to regenerate `atlas.sum` |
| Multiple migration heads | — | Validate | Linear history is enforced; resolve before merge |

## CI Pipeline

A typical migration CI workflow enforces checks when schema-relevant files change
(models, migrations, atlas.hcl, load_models.py):

```
+---------------------------------+
|  1. Drift check                 |  Missing migration detection
+---------------------------------+
|  2. atlas migrate validate      |  Checksum integrity, linear history
+---------------------------------+
```

## Anti-Patterns

| Do Not | Instead |
|--------|---------|
| Edit `atlas.sum` manually | Run `atlas migrate hash` to regenerate |
| Use `schema apply` in production | Use `migrate apply` (versioned workflow) |
| Skip the dev database for diffs | Always use `--env local` which includes the dev database |
| Hardcode database URLs in `atlas.hcl` | Use `getenv("DATABASE_URL")` |
| Run `migrate diff` against production | Diff against the dev database; it replays migrations to build current state |
| Ignore lint warnings | Investigate every warning; override with `-- atlas:nolint` only when justified |
| Apply migrations without reviewing SQL | Read every generated file before `migrate apply` |
| Modify applied migrations | Only edit migrations that haven't been applied to any shared environment |
| Suppress lint in `atlas.hcl` globally | Use file-level `-- atlas:nolint` for targeted overrides |
