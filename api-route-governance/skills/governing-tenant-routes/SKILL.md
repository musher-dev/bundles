---
name: governing-tenant-routes
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of multi-tenant API route structures including path-derived tenant context, dual-validation gateway middleware, global vs tenant-scoped endpoint classification, token structure validation, rate limiting boundaries, and BOLA prevention. Use when designing organization-scoped API routes, reviewing tenant resolution patterns, auditing API gateway middleware, or implementing multi-tenant endpoint conventions. Triggered by: tenant route, org slug, path-derived context, dual validation, BOLA prevention, tenant-scoped endpoint, API gateway, rate limiting, multi-tenant API, organization endpoint, route governance, endpoint classification.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Tenant Route Governance

Comprehensive governance for multi-tenant API route design, review, audit, and implementation.

## Purpose

In a multi-tenant SaaS platform, API routes are the primary surface where tenant context is established and validated. A poorly designed route structure enables Broken Object Level Authorization (BOLA) attacks, destroys edge observability, and creates cross-tenant data contamination risks. This skill encodes the canonical patterns for multi-tenant API route design built on path-derived explicit tenant context with dual-validation middleware.

Use this skill when designing new API routes, reviewing existing multi-tenant endpoint patterns, auditing API gateway middleware, or implementing organization-scoped endpoint conventions.

---

## Design Principles

### 1. Path-Derived Tenant Context

The target tenant must be embedded directly in the URI path, not inferred from the API key or passed in a custom header. The canonical pattern is:

```
/api/v1/organizations/{org_slug}/...
```

This provides defense-in-depth: the tenant is visible in access logs, WAFs can route or rate-limit by path, and caching layers work naturally with URL-based keys.

### 2. Dual-Validation Middleware

The API gateway must verify that the API key's embedded tenant identity matches the URI's requested tenant identity. This two-step verification prevents a key bound to Organization A from accessing Organization B's routes, even if the attacker knows valid resource IDs.

### 3. Global vs Tenant-Scoped Separation

Not all endpoints can be organization-scoped. The API must cleanly separate:
- **Tenant-scoped endpoints:** `/api/v1/organizations/{org_slug}/projects` -- require org-bound API keys or session tokens with org context
- **Global endpoints:** `/api/v1/me` -- accept global session tokens, reject org-bound API keys

Attempting to call a global endpoint with a tenant-bound credential must return `400 Bad Request`.

### 4. Checksum-First Validation

The gateway should validate the token's structural integrity (checksum) before performing any database lookup. This cheap gate rejects malformed tokens at the edge without consuming database connections.

---

## Route Conventions

### Tenant-Scoped Resources

All endpoints that operate on tenant data must include the organization slug in the path:

```
GET    /api/v1/organizations/{org_slug}/projects
POST   /api/v1/organizations/{org_slug}/projects
GET    /api/v1/organizations/{org_slug}/projects/{project_id}
PUT    /api/v1/organizations/{org_slug}/projects/{project_id}
DELETE /api/v1/organizations/{org_slug}/projects/{project_id}
GET    /api/v1/organizations/{org_slug}/members
GET    /api/v1/organizations/{org_slug}/billing
GET    /api/v1/organizations/{org_slug}/api-keys
POST   /api/v1/organizations/{org_slug}/api-keys
```

### Global Endpoints

Endpoints operating on the authenticated user's global identity, not scoped to any tenant:

```
GET    /api/v1/me                    -- User profile
PUT    /api/v1/me                    -- Update profile
GET    /api/v1/me/organizations      -- List user's organizations
POST   /api/v1/auth/token            -- Authentication
POST   /api/v1/auth/refresh          -- Token refresh
```

### Versioning

- Version prefix in path: `/api/v1/...`
- Major version only in the URI
- No version in headers or query parameters

---

## Validation Chain

When an API request arrives, the gateway processes it through this ordered middleware chain:

### Step 1: Token Structure Check
Extract the bearer token from the Authorization header. Parse the token structure `[prefix]_[env]_[entropy]_[checksum]`. Recalculate the CRC32 checksum of the entropy block. If the checksum does not match, reject with `401 Unauthorized` immediately. No database interaction occurs.

### Step 2: Key Lookup
Hash the full token with SHA-256 and query the database (or distributed edge cache) for matching `key_hash`. If not found, reject with `401 Unauthorized`. Check `revoked_at` -- if set, reject with `401 Unauthorized`. Check `expires_at` -- if past, reject with `401 Unauthorized`.

### Step 3: Tenant Match
Extract `{org_slug}` from the URI path. Look up the organization by slug. Compare the organization's ID against the API key's `org_id`. If they do not match, reject with `403 Forbidden`. This prevents cross-tenant access regardless of resource ID manipulation.

### Step 4: RBAC + Scope Check
Resolve the effective principal from the key's polymorphic relation (membership or service account). Retrieve the principal's RBAC role for this organization. Retrieve the key's explicit scopes from `api_key_scopes`. Compute effective permissions as: `principal_role ∩ key_scopes`. Evaluate whether the effective permissions allow the requested operation.

### Step 5: Handler Execution
Pass the validated request to the route handler with the resolved tenant context in a request-scoped object. Never set tenant context in a global variable or thread-local that could leak across concurrent requests.

---

## Review Dimensions

When reviewing or auditing API route designs, evaluate these six dimensions:

### 1. Path-Derived Tenant Context
- Do all tenant-scoped endpoints include `{org_slug}` in the URI path?
- Are there any tenant resources accessible via flat paths like `/api/v1/projects/{id}`?
- Is the org slug a path segment (not a query parameter or header)?

### 2. Dual-Validation Middleware
- Does the gateway validate key's `org_id` against the route's `org_slug`?
- Is the validation performed before the request reaches application logic?
- Does a mismatch return `403 Forbidden` (not `404` or `500`)?

### 3. Endpoint Classification
- Are endpoints explicitly classified as global or tenant-scoped?
- Do global endpoints reject org-bound API keys with `400 Bad Request`?
- Do tenant-scoped endpoints reject global session tokens without org context?

### 4. Token Structure
- Does the token follow `[prefix]_[env]_[entropy]_[checksum]` format?
- Is checksum validated at the gateway before database lookup?
- Is the prefix registered for secret scanning partnerships?

### 5. Rate Limiting
- Is rate limiting applied per-tenant (not just per-IP)?
- Does the path structure enable tenant-level throttling at the edge?
- Are per-key rate limits supported for noisy-neighbor prevention?

### 6. BOLA Prevention
- Does every database query for tenant resources include `org_id` in the WHERE clause?
- Are RLS policies in place to enforce tenant isolation at the database level?
- Is the tenant context passed via request-scoped objects (not global state)?

---

## Anti-Patterns

### 1. Implicit Key-Derived Context

Resolving the tenant from the API key alone without requiring it in the URL path. Routes like `GET /api/v1/projects` rely entirely on application code to filter by the key's hidden tenant ID. If a developer writes a query using only the resource ID, the platform is instantly vulnerable to BOLA. Edge observability is also destroyed -- all tenant traffic is mixed on a single route.

### 2. Header-Derived Tenant Context

Using `X-Tenant-ID` or similar headers instead of path segments. Headers are invisible in access logs, complicate caching (cache keys typically ignore custom headers), and enable cache poisoning across tenants. Spoofing a header is trivial compared to modifying a URL path that the gateway validates.

### 3. Missing Tenant in URI for Tenant Resources

Endpoints like `GET /api/v1/projects/{id}` that serve tenant data without an org_slug path segment. Makes BOLA trivial -- an attacker provides their valid API key and guesses resource IDs belonging to other tenants. The gateway cannot detect the mismatch because the target tenant is never declared.

### 4. Org-Bound Keys on Global Endpoints

Allowing organization-scoped API keys to call `/api/v1/me` or other global endpoints. This conflates the human identity domain with the tenant machine domain. Global endpoints must reject tenant-bound credentials to maintain strict separation.

### 5. Checksum Validation After DB Lookup

Performing the database lookup before validating the token's CRC32 checksum. The checksum is a computationally cheap gate that should reject malformed tokens at the edge. Checking it after the database lookup wastes connection pool resources and leaves the system vulnerable to Layer 7 DDoS via random token flooding.

### 6. Global State Tenant Overrides

Application middleware that sets the tenant context in a global variable or thread-local after resolving the API key. In async/concurrent runtimes, this creates cross-request tenant leakage where Request A's tenant context is overwritten by Request B, causing Request A to read Request B's data. Tenant context must be explicitly passed down the call stack in request-scoped objects.

---

## Implementation Guidance

### Phase 1: Path-Explicit Routes + Gateway Middleware

Build the foundational routing:
- Establish `/api/v1/organizations/{org_slug}/...` as the canonical pattern for all tenant resources
- Implement gateway middleware that extracts org_slug from path and validates against key's org_id
- Classify all existing endpoints as global or tenant-scoped
- Reject mismatched key-to-route tenant with `403 Forbidden`
- Pass tenant context via request-scoped objects, never global state
- Institute code-review policy: every SQL query for tenant data must include `org_id` in WHERE clause

### Phase 2: Per-Tenant Rate Limiting

Add tenant-aware traffic management:
- Implement Token Bucket rate limiting at the gateway keyed by org_slug
- Add per-key rate limits for noisy-neighbor prevention
- Configure WAF rules that can target specific tenant paths
- Add `X-RateLimit-*` response headers per tenant

### Phase 3: Edge Caching and Advanced Routing

Optimize for scale:
- Implement edge caching with tenant-aware cache keys (URL naturally includes org_slug)
- Deploy checksum pre-validation at CDN edge workers
- Cache key metadata in distributed edge cache (Redis) with 60-300s TTL
- Implement pub-sub cache eviction for instant key revocation

---

## Context Resolution Comparison

| Resolution Model | Example | Security | Observability |
|-----------------|---------|----------|---------------|
| Implicit (Key-Derived) | `GET /v1/projects` | Medium (High BOLA risk) | Poor (traffic mixed) |
| Header-Derived | `GET /v1/projects` + `X-Tenant: acme` | Low (spoofing + cache risk) | Poor (CDNs ignore headers) |
| Explicit (Path-Derived) | `GET /v1/orgs/acme/projects` | High (defense-in-depth) | Excellent (native REST) |
| **Hybrid (Recommended)** | `GET /v1/orgs/acme/projects` + key validates | **Maximum (dual-verification)** | **Excellent** |

---

## Examples

### Example 1: API with Flat Resource Paths

**Scenario:** Routes use `GET /api/v1/projects/{id}` without org_slug for tenant data.

**Findings:**
- **Path-Derived Tenant Context: FAIL** -- Tenant resources are accessible without declaring the target tenant in the URI.
- **Dual-Validation: FAIL** -- No org_slug in the path means there is nothing to validate against the key's org_id.
- **BOLA Prevention: FAIL** -- An attacker with a valid key can access any resource by guessing IDs across tenants.
- **Rate Limiting: PARTIAL** -- Cannot implement per-tenant rate limiting because all tenants share the same route paths.
- **Recommendation:** Migrate all tenant-scoped routes to `/api/v1/organizations/{org_slug}/...` pattern. Implement gateway middleware for dual-validation.

### Example 2: Path Tenant But Missing Endpoint Classification

**Scenario:** Routes use `/api/v1/organizations/{org_slug}/projects` correctly, but also allow org-scoped API keys to call `/api/v1/me`.

**Findings:**
- **Path-Derived Tenant Context: PASS** -- Tenant resources are properly scoped.
- **Endpoint Classification: FAIL** -- No separation between global and tenant-scoped endpoints. An org-bound machine credential should not be able to access global user profile data.
- **Recommendation:** Add middleware that detects org-bound API keys on global endpoints and rejects with `400 Bad Request`.

### Example 3: Well-Designed Multi-Tenant Route Structure

**Scenario:** All tenant resources at `/api/v1/organizations/{org_slug}/...`, global endpoints at `/api/v1/me` and `/api/v1/auth/...`, gateway validates key org_id against route org_slug, checksum pre-validation at edge, per-tenant rate limiting.

**Findings:**
- All six dimensions: **PASS**
- **Minor recommendation:** Add `X-API-Key-Expiring: true` response header for keys approaching expiration to enable proactive client rotation.
