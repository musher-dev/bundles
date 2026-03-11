---
name: governing-api-versioning
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of API versioning and lifecycle management including Expand and Contract migration strategy, RFC 8594 Deprecation header, Sunset header, URI-based versioning with /v1/ prefix, breaking change classification, and deprecation timelines. Use when designing API versioning strategy, reviewing breaking changes, auditing deprecation processes, or implementing version migration. Triggered by: API versioning, version migration, breaking change, deprecation, sunset header, expand and contract, RFC 8594, /v1/, version lifecycle, API evolution, backward compatibility, deprecation timeline, version strategy.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# API Versioning Governance

Comprehensive governance for API versioning, lifecycle management, and backward-compatible evolution.

## Purpose

API versioning is the mechanism by which an API evolves without breaking existing consumers. Poor versioning -- surprise breaking changes, silent field removals, missing deprecation warnings -- destroys consumer trust and generates support burden. This skill enforces a disciplined versioning strategy using URI-based major versions, Expand-Contract migration, and standards-based deprecation signaling.

Use this skill when designing a versioning strategy, reviewing breaking changes, auditing deprecation processes, or implementing version migrations.

---

## Versioning Strategy

### URI-Based Major Versions

The API version is a path prefix. Only major versions appear in the URI:

```
/api/v1/projects
/api/v2/projects
```

**Rules:**
- Version prefix format: `/api/v{N}/` where N is a positive integer
- Only major version changes increment the URI version
- Minor and patch changes are backward-compatible and do not change the URI
- All endpoints under a version share the same lifecycle

**Why URI-based:**

| Strategy | Pros | Cons | Verdict |
|----------|------|------|---------|
| URI path (`/v1/`) | Visible, cacheable, simple routing | URL changes on major version | **Recommended** |
| Header (`Accept-Version: 1`) | Clean URLs | Invisible, cache complications | Avoid |
| Query param (`?version=1`) | Easy to add | Inconsistent, optional by nature | Avoid |
| Content negotiation (`Accept: application/vnd.api.v1+json`) | HTTP-pure | Complex, poor tooling support | Avoid |

### Version Lifecycle States

| State | Description | Headers | Duration |
|-------|-------------|---------|----------|
| **Active** | Current recommended version | None | Indefinite |
| **Deprecated** | Still functional, successor available | `Deprecation` + `Sunset` | Minimum 6 months |
| **Sunset** | Read-only or rejected | `Sunset` (past date) | 30-day grace period |
| **Retired** | Returns 410 Gone | None | Permanent |

---

## Breaking Change Classification

### What Constitutes a Breaking Change

| Change | Breaking? | Reason |
|--------|-----------|--------|
| Remove a field from response | **Yes** | Clients may depend on it |
| Rename a field | **Yes** | Clients reference the old name |
| Change a field's type | **Yes** | Clients parse the old type |
| Add a required field to request | **Yes** | Existing requests lack it |
| Remove an endpoint | **Yes** | Clients call it |
| Change authentication mechanism | **Yes** | Clients use the old auth |
| Change status code meaning | **Yes** | Client error handling breaks |
| Tighten validation (reject previously valid input) | **Yes** | Existing valid requests fail |

### Non-Breaking Changes (Safe to Deploy)

| Change | Safe? | Reason |
|--------|-------|--------|
| Add optional field to request | **Yes** | Existing requests still valid |
| Add field to response | **Yes** | Clients should ignore unknown fields |
| Add new endpoint | **Yes** | No existing client calls it |
| Loosen validation (accept more input) | **Yes** | Existing valid requests still valid |
| Add new enum value to response | **Caution** | Safe if clients handle unknown values |
| Add new error code | **Yes** | Clients should handle unexpected errors |

---

## Expand and Contract Migration

### The Pattern

Expand-Contract is a two-phase migration that maintains backward compatibility throughout:

```
Phase 1: EXPAND          Phase 2: CONTRACT
├─ Add new field/endpoint    ├─ Remove old field/endpoint
├─ Dual-write to both        ├─ Stop writing to old
├─ Read from old by default  ├─ Read from new only
└─ Announce deprecation       └─ Complete migration
```

### Step-by-Step Process

**Step 1: Expand**
- Add the new field, endpoint, or behavior alongside the old one
- If renaming a field: add the new field, populate it from the old field
- If restructuring: add the new endpoint, keep the old one functional
- Both old and new work simultaneously -- zero consumer breakage

**Step 2: Deprecate**
- Add `Deprecation` header to old field/endpoint responses
- Add `Sunset` header with the planned removal date
- Document the migration path in changelog
- Notify consumers via appropriate channels

**Step 3: Migrate**
- Monitor old field/endpoint usage metrics
- Assist consumers who haven't migrated
- Extend sunset date if adoption is insufficient

**Step 4: Contract**
- Remove the old field/endpoint after sunset date passes
- Return clear error messages if old paths are still called
- Log removal metrics for compliance

### Example: Renaming a Response Field

```
# Step 1: EXPAND - Add new field alongside old
Response: { "user_name": "alice", "username": "alice" }

# Step 2: DEPRECATE - Signal in headers
Deprecation: true
Sunset: Sat, 01 Mar 2025 00:00:00 GMT
Link: <https://docs.example.com/migration/username>; rel="deprecation"

# Step 3: MIGRATE - Monitor usage of "user_name"

# Step 4: CONTRACT - Remove old field
Response: { "username": "alice" }
```

---

## Deprecation Headers

### RFC 8594 Deprecation Header

Signal that a resource is deprecated:

```
Deprecation: true
```

Or with a date indicating when deprecation took effect:

```
Deprecation: Sat, 01 Jan 2025 00:00:00 GMT
```

### Sunset Header

Signal the date when the resource will become unavailable:

```
Sunset: Sat, 01 Jul 2025 00:00:00 GMT
```

### Link Header for Migration Documentation

Point consumers to migration documentation:

```
Link: <https://docs.example.com/api/migration/v1-to-v2>; rel="deprecation"
```

### Combined Header Example

```http
HTTP/1.1 200 OK
Content-Type: application/json
Deprecation: Sat, 01 Jan 2025 00:00:00 GMT
Sunset: Sat, 01 Jul 2025 00:00:00 GMT
Link: <https://docs.example.com/api/migration/v1-to-v2>; rel="deprecation"
```

---

## Deprecation Timeline

### Minimum Timelines

| Consumer Type | Minimum Deprecation Period | Rationale |
|--------------|---------------------------|-----------|
| External/public API | 12 months | Third-party release cycles |
| Partner API | 6 months | Coordinated migration |
| Internal API | 3 months | Team coordination |
| Alpha/beta API | 30 days | Expectation of instability |

### Timeline Process

1. **T-0: Announce deprecation** -- Add Deprecation header, publish changelog, notify consumers
2. **T+30d: First reminder** -- Email/notification to consumers still using deprecated endpoint
3. **T+60d: Usage review** -- Assess migration progress, decide if extension needed
4. **T+90d (internal) / T+180d (partner) / T+365d (public): Sunset** -- Add Sunset header with past date, begin returning 410 Gone
5. **T+sunset+30d: Retire** -- Remove endpoint code entirely

---

## Review Dimensions

When reviewing or auditing API versioning, evaluate these dimensions:

### 1. Version Strategy
- Is versioning URI-based with `/api/v{N}/` prefix?
- Are only major version changes reflected in the URI?
- Are backward-compatible changes deployed without version bumps?

### 2. Breaking Change Management
- Are breaking changes correctly classified?
- Is the Expand-Contract pattern used for migrations?
- Are there any silent breaking changes (field removals without deprecation)?

### 3. Deprecation Signaling
- Do deprecated endpoints include `Deprecation` and `Sunset` headers?
- Is migration documentation linked via the `Link` header?
- Are deprecation timelines appropriate for the consumer type?

### 4. Lifecycle Compliance
- Do deprecated endpoints still function until the sunset date?
- Do sunset endpoints return 410 Gone with a clear message?
- Are retired endpoints fully removed from the codebase?

---

## Anti-Patterns

### 1. Silent Breaking Changes
Removing or renaming response fields without deprecation headers or notice. Consumers discover the break in production.

### 2. Perpetual Beta
Keeping an API in "beta" indefinitely to avoid versioning commitments. If the API has production consumers, it needs versioning discipline.

### 3. Version Explosion
Creating v3, v4, v5 for minor changes instead of using backward-compatible additions. Each version multiplies maintenance burden.

### 4. Deprecation Without Sunset
Adding a `Deprecation` header but never setting a `Sunset` date. Consumers have no urgency to migrate, and the deprecated endpoint lives forever.

### 5. Immediate Removal
Removing an endpoint on the same day it's deprecated. Zero migration window forces emergency changes on consumers.

### 6. Header-Based Versioning
Using `Accept-Version` or custom headers for versioning. This makes version routing invisible in logs, breaks caching, and complicates debugging.

---

## Examples

### Example 1: Field Renamed Without Warning

**Scenario:** Response field `user_name` changed to `username` in a deployment with no deprecation period.

**Findings:**
- **Breaking Change Management: FAIL** -- Field rename is a breaking change deployed without Expand-Contract
- **Deprecation Signaling: FAIL** -- No Deprecation or Sunset headers were ever sent
- **Recommendation:** Revert to dual-writing both fields, add deprecation headers to old field, set 6-month sunset

### Example 2: Proper Expand-Contract in Progress

**Scenario:** New field `username` added alongside `user_name`, Deprecation header present, Sunset set 6 months out, migration docs linked.

**Findings:**
- All four dimensions: **PASS**
- **Note:** Monitor `user_name` usage to confirm migration progress before sunset date

### Example 3: Version 4 with 3 Active Versions

**Scenario:** API has v1, v2, v3, and v4 all active. No Deprecation headers on v1 or v2.

**Findings:**
- **Lifecycle Compliance: FAIL** -- v1 and v2 should be deprecated with Sunset dates
- **Recommendation:** Add Deprecation headers to v1 and v2, set sunset dates, communicate migration plan to consumers
