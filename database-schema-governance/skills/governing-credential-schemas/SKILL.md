---
name: governing-credential-schemas
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of multi-tenant API credential database schemas including polymorphic principal patterns, cryptographic hash storage, lifecycle management, scope tables, audit trails, and tenant isolation. Use when designing api_keys tables, reviewing credential storage schemas, auditing multi-tenant authentication models, or implementing API key systems. Triggered by: api key schema, credential table, polymorphic principal, hash storage, key lifecycle, tenant isolation, RLS, audit log schema, scope table, multi-tenant credentials, api key design, credential migration.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Credential Schema Governance

Comprehensive governance for multi-tenant API credential database schema design, review, audit, and implementation.

## Purpose

API credential schemas are the security foundation of multi-tenant SaaS platforms. A poorly designed credential table can leak data across tenant boundaries, store secrets in plaintext, or create operational fragility when employees leave. This skill encodes the canonical patterns for credential schema design built on a polymorphic principal model where every API key belongs to exactly one organization.

Use this skill when designing new credential tables, reviewing existing api_keys schemas, auditing multi-tenant authentication data models, or implementing API key storage systems.

---

## Design Principles

### 1. Every Key Belongs to One Tenant

API keys must be structurally bound to the tenant boundary, not the global user boundary. The `org_id` foreign key on the api_keys table is the fundamental enforcement mechanism. A key without a tenant binding is a cross-tenant data leak waiting to happen.

### 2. Polymorphic Principals

The credential owner is not always a human. The schema must support both human-driven automation (via Membership) and machine-to-machine integrations (via Service Account) through a single polymorphic relation. This uses a `principal_type` enum and `principal_id` foreign key pattern.

### 3. Hash-Only Storage

Plaintext API keys must never be stored. The key is displayed once at creation, then immediately hashed with SHA-256. The database stores only the hash, a prefix for identification, the last four characters for display, and a checksum for gateway-level pre-validation.

### 4. Soft-Delete Revocation

Revoked keys must never be hard-deleted. The `revoked_at` timestamp preserves the forensic audit trail while instantly invalidating access. Deleting rows destroys evidence needed for breach investigation.

### 5. Scopes as Normalized Relations

Key scopes belong in a dedicated join table, not a JSON column. Normalized scopes enable FK constraints, efficient queries, and individual scope grant/revocation tracking.

---

## Reference Schema

### api_keys Table

| Column | Type | Constraints | Purpose |
|--------|------|-------------|---------|
| `id` | UUID | PK | Primary identifier (UUIDv7/TSID recommended) |
| `org_id` | UUID | FK → organizations, NOT NULL | Tenant boundary enforcement |
| `principal_type` | VARCHAR | NOT NULL, CHECK IN ('membership', 'service_account') | Polymorphic discriminator |
| `principal_id` | UUID | NOT NULL | FK to memberships or service_accounts based on type |
| `key_prefix` | VARCHAR(16) | NOT NULL | Environment indicator (e.g., 'live', 'test') |
| `key_hash` | VARCHAR(255) | NOT NULL, UNIQUE | SHA-256 hash for authentication lookup |
| `last_four` | CHAR(4) | NOT NULL | UI display and developer identification |
| `checksum` | VARCHAR(8) | NOT NULL | CRC32/Base62 for gateway pre-validation |
| `name` | VARCHAR(255) | NOT NULL | User-supplied friendly name |
| `expires_at` | TIMESTAMPTZ | | Expiration time (strongly encourage non-null) |
| `last_used_at` | TIMESTAMPTZ | | Async-updated usage tracking for stale key detection |
| `revoked_at` | TIMESTAMPTZ | | Soft-delete revocation timestamp |
| `created_at` | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | Immutable creation timestamp |
| `updated_at` | TIMESTAMPTZ | NOT NULL | Set on every change |

**Required indexes:**
- UNIQUE on `key_hash` (authentication lookup)
- INDEX on `org_id` (tenant-scoped queries)
- INDEX on `principal_type, principal_id` (principal-scoped queries)
- PARTIAL INDEX on `revoked_at IS NULL` (active keys only)
- INDEX on `expires_at` WHERE `revoked_at IS NULL` (expiration sweeps)

### api_key_scopes Table

| Column | Type | Constraints | Purpose |
|--------|------|-------------|---------|
| `id` | UUID | PK | Primary identifier |
| `key_id` | UUID | FK → api_keys, NOT NULL | Parent key reference |
| `scope_name` | VARCHAR(128) | NOT NULL | Capability in resource:action format |
| `created_at` | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | Grant timestamp |

**Required constraints:**
- UNIQUE on `(key_id, scope_name)` (prevent duplicate scope grants)
- INDEX on `key_id` (scope lookup by key)

**Scope naming convention:** `resource:action` format (e.g., `billing:read`, `projects:write`, `users:admin`).

### audit_logs Table

| Column | Type | Constraints | Purpose |
|--------|------|-------------|---------|
| `id` | UUID | PK | Primary identifier |
| `org_id` | UUID | FK → organizations, NOT NULL | Tenant-partitioned audit trail |
| `actor_type` | VARCHAR | NOT NULL | Polymorphic actor discriminator |
| `actor_id` | UUID | NOT NULL | Human or machine actor reference |
| `api_key_id` | UUID | FK → api_keys | Specific credential used (nullable for session-based actions) |
| `action` | VARCHAR(255) | NOT NULL | Operation performed |
| `ip_address` | INET | | Client IP address |
| `metadata` | JSONB | | Additional context (request details, resource IDs) |
| `created_at` | TIMESTAMPTZ | NOT NULL, DEFAULT NOW() | Immutable event timestamp |

**Required indexes:**
- INDEX on `org_id, created_at` (tenant-scoped chronological queries)
- INDEX on `actor_type, actor_id` (actor activity lookups)
- INDEX on `api_key_id` WHERE `api_key_id IS NOT NULL` (key-specific audit trail)

---

## Entity Relationships

```
organizations
  ├── 1:N → memberships (user + org junction with RBAC role)
  ├── 1:N → service_accounts (machine identity with RBAC role)
  └── 1:N → api_keys (tenant boundary)
              ├── polymorphic → memberships | service_accounts (via principal_type + principal_id)
              └── 1:N → api_key_scopes (capability grants)
```

- An Organization has many Memberships and many Service Accounts
- Both Memberships and Service Accounts have many API Keys
- An API Key always belongs to exactly one Organization (enforced by `org_id` FK)
- An API Key has many Scopes (capability restrictions)

---

## Review Dimensions

When reviewing or auditing a credential schema, evaluate these six dimensions:

### 1. Polymorphic Principal Design
- Does `principal_type` use a constrained enum (not free-text)?
- Does the schema support both membership and service account principals?
- Is the polymorphic FK pattern consistent (type + id columns)?
- Can the principal type enum be extended without schema migration?

### 2. Cryptographic Storage
- Is the key stored as a SHA-256 hash, never plaintext?
- Does the schema include `key_prefix` and `last_four` for identification without decryption?
- Is there a `checksum` column for gateway-level pre-validation?
- Is `key_hash` indexed with a UNIQUE constraint?

### 3. Lifecycle Management
- Does the schema include `expires_at` for key expiration?
- Is `last_used_at` present for stale key detection?
- Is `revoked_at` used for soft-delete (not hard-delete)?
- Do standard audit timestamps (`created_at`, `updated_at`) exist?

### 4. Scope Model
- Are scopes in a separate normalized table (not a JSON column)?
- Does the scope table have a composite unique constraint on `(key_id, scope_name)`?
- Do scope names follow the `resource:action` convention?
- Can scopes be individually granted and revoked?

### 5. Audit Trail Design
- Is the audit log partitioned by `org_id`?
- Does the audit log capture `api_key_id` to identify the specific credential?
- Is the audit log insert-only (immutable)?
- Are actor references polymorphic (supporting both human and machine actors)?

### 6. Tenant Isolation
- Does `api_keys` have an `org_id` FK to organizations (not just `user_id`)?
- Are RLS policies in place or planned for tenant isolation?
- Do all credential-related queries include `org_id` in WHERE clauses?
- Is the `org_id` FK NOT NULL (preventing orphaned keys)?

---

## Anti-Patterns

### 1. Storing Plaintext Secrets

Keeping the raw API key in the database instead of a SHA-256 hash. A database compromise immediately exposes every active credential. The key must be displayed once at creation and never be recoverable from storage.

### 2. Global User-Scoped Keys

An api_keys table with `user_id` but no `org_id`, making credentials span all tenant boundaries the user belongs to. A single leaked key exposes every organization. Every key must be structurally bound to exactly one tenant via `org_id` FK.

### 3. Hard-Deleting Revoked Keys

Using DELETE instead of setting `revoked_at`. Destroys the forensic audit trail and makes breach investigation impossible. Revoked keys must remain as historical records with `revoked_at` timestamp.

### 4. Scopes as JSON Array

Storing scopes in a JSONB column on api_keys instead of a normalized join table. Prevents FK constraints, efficient scope queries, and individual scope grant/revocation tracking. Makes it impossible to query "all keys with billing:read scope" efficiently.

### 5. Missing Checksum Column

Omitting the token checksum that enables gateway-level validation before database lookup. Forces every incoming request to hit the database for key verification, making the system vulnerable to brute-force attacks consuming database connection pools.

### 6. Organization-Scoped Anonymous Keys

Keys owned by the organization entity itself rather than a specific principal. Audit logs show "an API key did this" without attributing the action to a human or named machine, making incident response impossible.

---

## Implementation Guidance

### Phase 1: Membership Keys + Hash Storage

Build the foundational schema:
- Create `api_keys` table with `principal_type` defaulting to `'membership'`
- Implement structured token generation: `[prefix]_[env]_[entropy]_[checksum]`
- Store only SHA-256 hashes with prefix, last_four, and checksum columns
- Enforce `org_id` FK on all credential records
- Create the `audit_logs` table with `api_key_id` tracking
- Implement gateway checksum pre-validation

### Phase 2: Service Accounts

Extend the polymorphic model:
- Create `service_accounts` table within the organization boundary
- Expand `principal_type` enum to include `'service_account'`
- Build lifecycle management UI for machine identities
- Zero schema migration required for api_keys table

### Phase 3: Granular Scopes

Add fine-grained capability control:
- Populate `api_key_scopes` table
- Implement scope intersection at authorization: effective permissions = principal RBAC ∩ key scopes
- Build scope selection UI for key creation
- A key's scope can never exceed its principal's authority

---

## Token Anatomy

```
[prefix]_[env]_[entropy]_[checksum]
  │        │       │          │
  │        │       │          └── CRC32/Base62 for gateway pre-validation
  │        │       └── 256-bit CSPRNG entropy (hex or base62 encoded)
  │        └── Environment: live | test | staging
  └── Platform prefix for secret scanning (e.g., acme)
```

**Example:** `acme_live_9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08_7xB2`

**Storage rules:**
- Display plaintext to user exactly once at creation
- Immediately hash with SHA-256
- Store: `key_hash`, `key_prefix`, `last_four`, `checksum`
- Never log the plaintext token

---

## Rotation and Expiration

Support rolling key transitions for zero-downtime rotation:

1. Administrator generates a new key
2. Sets `expires_at` on the old key (e.g., 72 hours in the future)
3. Both keys remain concurrently valid during the transition window
4. API responses include `X-API-Key-Expiring: true` header for proactive client alerts
5. Old key naturally expires; `revoked_at` is not set (expiration is distinct from revocation)

For immediate revocation (compromised key):
1. Set `revoked_at` on the key record
2. Evict cached key hash via pub-sub mechanism
3. Key is neutralized globally within the cache TTL (60-300 seconds max)

---

## Examples

### Example 1: Schema Missing Tenant Binding

**Scenario:** An api_keys table has `user_id` FK but no `org_id` column.

**Findings:**
- **Tenant Isolation: FAIL** -- Keys inherit all tenant access from the global user. A leaked key exposes every organization the user belongs to.
- **Polymorphic Principal: PARTIAL** -- No principal_type column; all keys are implicitly user-scoped with no path to service accounts.
- **Recommendation:** Add `org_id` FK as NOT NULL, add `principal_type` enum, migrate existing keys to membership-scoped with explicit org binding.

### Example 2: Credential Schema with Inline Scopes

**Scenario:** An api_keys table has a `scopes jsonb DEFAULT '[]'` column instead of a join table.

**Findings:**
- **Scope Model: FAIL** -- JSON array prevents FK constraints, cannot efficiently query "all keys with billing:read scope", and individual scope grant/revocation requires full array rewrite.
- **Audit Trail: PARTIAL** -- No granular scope change tracking. The audit log cannot distinguish "scope added" from "key updated."
- **Recommendation:** Create `api_key_scopes` join table, migrate existing JSON scopes, drop the JSON column.

### Example 3: Well-Designed Credential Schema

**Scenario:** An api_keys table with `org_id` FK, `principal_type` enum, `key_hash` (SHA-256), `last_four`, `checksum`, `revoked_at`, `expires_at`, and a separate api_key_scopes table.

**Findings:**
- All six dimensions: **PASS**
- **Minor recommendation:** Add partial index on `expires_at WHERE revoked_at IS NULL` for efficient expiration sweep queries.
