---
name: governing-http-semantics
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of HTTP method semantics following RFC 9110 including method safety for GET and HEAD, idempotency guarantees for PUT and DELETE, precise status code selection distinguishing 201 from 202 and 401 from 403 and 422, and Content-Type negotiation. Use when reviewing HTTP method usage, auditing status codes, enforcing REST semantics, or implementing correct HTTP behavior. Triggered by: HTTP semantics, RFC 9110, method safety, idempotency, idempotent, status code, 201, 202, 401, 403, 422, content negotiation, safe method, GET safety, PUT idempotency, HTTP method, REST semantics.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# HTTP Semantics Governance

Comprehensive governance for HTTP method usage and status code selection, enforcing RFC 9110 compliance across API endpoints.

## Purpose

HTTP methods and status codes are a shared contract between every client and server on the internet. When an API misuses them -- making GET endpoints produce side effects, returning 200 for everything, conflating 401 and 403 -- clients cannot rely on standard HTTP behavior. Caches break, retry logic misfires, and error handling becomes guesswork. This skill enforces RFC 9110 semantics so APIs behave predictably.

Use this skill when designing new endpoints, reviewing HTTP method selection, auditing status code usage, or implementing correct REST semantics.

---

## Method Semantics

### Safety and Idempotency Matrix

| Method | Safe | Idempotent | Request Body | Typical Use |
|--------|------|------------|-------------|-------------|
| GET | Yes | Yes | No | Retrieve resource |
| HEAD | Yes | Yes | No | Check resource existence/metadata |
| OPTIONS | Yes | Yes | No | Discover allowed methods |
| POST | No | No | Yes | Create resource, trigger action |
| PUT | No | Yes | Yes | Full resource replacement |
| PATCH | No | No | Yes | Partial resource update |
| DELETE | No | Yes | No | Remove resource |

### Safety (GET, HEAD, OPTIONS)

Safe methods must not produce side effects on the server. A GET request must never create, update, or delete data.

**Violations to detect:**
- `GET /reports/{id}` that marks the report as "viewed" (write side effect)
- `GET /exports/{id}` that triggers an export job (action side effect)
- `DELETE /cache` exposed as `GET /clear-cache` (destructive action on safe method)

**Acceptable server-side work on safe methods:**
- Logging the request (operational, not business state)
- Incrementing analytics counters (observability, not resource state)
- Updating `last_accessed_at` on a cache entry (infrastructure, not business)

### Idempotency (PUT, DELETE)

Idempotent methods produce the same result when called multiple times with the same input. The server state after N identical calls must be the same as after 1 call.

**PUT must be idempotent:**
- `PUT /projects/{id}` with the same body called twice results in the same resource state
- PUT replaces the entire resource -- it is not a partial update
- If a field is omitted from the PUT body, it should be set to its default or null

**DELETE must be idempotent:**
- `DELETE /projects/{id}` called twice: first call deletes, second call returns 404 (or 204)
- Both calls leave the server in the same state: the resource is gone
- Never return an error on the second DELETE -- the desired state is achieved

**POST is not idempotent:**
- `POST /projects` called twice creates two projects
- For idempotent creation, use `PUT /projects/{client-generated-id}` or an `Idempotency-Key` header

### PATCH Semantics

PATCH applies a partial update. Use JSON Merge Patch (RFC 7396) or JSON Patch (RFC 6902):

| Format | Content-Type | Behavior |
|--------|-------------|----------|
| JSON Merge Patch | `application/merge-patch+json` | Null means "remove field" |
| JSON Patch | `application/json-patch+json` | Array of operations (add, remove, replace) |

**Rule:** If PATCH is supported, document which patch format the API accepts. Do not accept `application/json` for PATCH -- it creates ambiguity about null semantics.

---

## Status Code Precision

### Success Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 OK | Request succeeded, response body present | GET returning data, PUT/PATCH returning updated resource |
| 201 Created | Resource created, response body present | POST that creates a resource (include `Location` header) |
| 202 Accepted | Request accepted for async processing | POST triggering a long-running job (include status URL) |
| 204 No Content | Request succeeded, no response body | DELETE, or PUT/PATCH when not returning the resource |

**201 vs 202 decision:**
- 201: The resource exists right now. The client can GET it immediately.
- 202: The work is queued. The resource may not exist yet. Provide a status polling URL.

### Client Error Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 400 Bad Request | Malformed request syntax | Unparseable JSON, missing required header |
| 401 Unauthorized | No valid credentials | Missing or expired token, invalid API key |
| 403 Forbidden | Valid credentials, insufficient permissions | Authenticated but lacks required role/scope |
| 404 Not Found | Resource does not exist | ID not found, or intentional masking of 403 |
| 405 Method Not Allowed | HTTP method not supported on this route | PUT on a read-only resource |
| 409 Conflict | Request conflicts with current state | Duplicate creation, concurrent edit conflict |
| 422 Unprocessable Content | Well-formed but semantically invalid | Validation errors (invalid email format, out-of-range value) |
| 429 Too Many Requests | Rate limit exceeded | Include `Retry-After` header |

**401 vs 403 decision:**
- 401: "I don't know who you are." The client should authenticate (or re-authenticate).
- 403: "I know who you are, but you can't do this." Re-authenticating won't help.

**400 vs 422 decision:**
- 400: The request is syntactically broken (malformed JSON, wrong Content-Type).
- 422: The request is syntactically valid but semantically wrong (email format invalid, date in the past).

### Server Error Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 500 Internal Server Error | Unhandled server failure | Bug, unrecoverable exception |
| 502 Bad Gateway | Upstream service returned invalid response | Proxy/gateway received bad response |
| 503 Service Unavailable | Server temporarily unable to handle request | Maintenance, overload (include `Retry-After`) |
| 504 Gateway Timeout | Upstream service did not respond in time | Proxy/gateway timeout |

---

## Content-Type Negotiation

### Request Content-Type

Endpoints that accept a request body must validate the `Content-Type` header:

- JSON endpoints require `application/json` (or `application/merge-patch+json` for PATCH)
- Return `415 Unsupported Media Type` for unexpected Content-Types
- Never silently parse a request body without checking Content-Type

### Response Content-Type

- Always include `Content-Type: application/json` (or `application/problem+json` for errors)
- Support `Accept` header negotiation when multiple formats are available
- Return `406 Not Acceptable` when the requested format is not supported

---

## Review Dimensions

When reviewing or auditing HTTP semantics, evaluate these dimensions:

### 1. Method Safety
- Do any GET or HEAD endpoints produce side effects?
- Are all read-only operations using GET (not POST)?
- Are safe methods guaranteed to be non-destructive?

### 2. Idempotency
- Is PUT used for full replacement (not partial updates)?
- Does DELETE handle already-deleted resources gracefully?
- Are non-idempotent creation operations on POST (not PUT)?

### 3. Status Code Precision
- Are 201 and 202 distinguished correctly (sync vs async creation)?
- Are 401 and 403 used for the correct scenarios?
- Are 400 and 422 distinguished (syntax vs semantic errors)?
- Do 429 responses include `Retry-After`?

### 4. Content Negotiation
- Do endpoints validate request `Content-Type`?
- Do error responses use `application/problem+json`?
- Is `415 Unsupported Media Type` returned for wrong Content-Types?

---

## Anti-Patterns

### 1. GET with Side Effects
Using `GET /emails/{id}/mark-read` to update read status. GET must be safe. Use `PATCH /emails/{id}` with `{ "is_read": true }` or `POST /emails/{id}:mark-read`.

### 2. 200 for Everything
Returning `200 OK` for creations (should be 201), deletions (should be 204), and errors (should be 4xx/5xx). Generic 200 defeats client-side status code routing.

### 3. POST for Reads
Using `POST /search` with a query body instead of `GET /resources?q=term`. POST should only be used for search when query parameters exceed URL length limits or contain sensitive data.

### 4. 401/403 Confusion
Returning 401 when the user is authenticated but lacks permissions (should be 403), or returning 403 when no credentials are provided (should be 401).

### 5. 500 for Validation Errors
Returning 500 when the client sends invalid data. Client errors are 4xx -- only unhandled server failures are 5xx. A validation error is a 422.

### 6. Non-Idempotent DELETE
Returning `410 Gone` or `500` on the second DELETE of the same resource. The resource is already deleted -- the desired state is achieved. Return 404 or 204.

---

## Examples

### Example 1: API Using GET for Mutations

**Scenario:** `GET /api/v1/reports/{id}/generate` triggers report generation as a side effect.

**Findings:**
- **Method Safety: FAIL** -- GET triggers a write operation (report generation). GET must be safe per RFC 9110.
- **Recommendation:** Change to `POST /api/v1/reports/{id}:generate` returning 202 Accepted with a status polling URL.

### Example 2: Incorrect Status Code Usage

**Scenario:** POST to create a resource returns 200 with the created resource. DELETE returns `{ "success": true }` with 200.

**Findings:**
- **Status Code Precision: FAIL** -- Creation should return 201 Created with `Location` header. Deletion should return 204 No Content.
- **Recommendation:** Use 201 for successful POST creation, 204 for successful DELETE.

### Example 3: Correct HTTP Semantics

**Scenario:** GET is read-only, POST returns 201 with Location, PUT is idempotent full replacement, DELETE returns 204, PATCH uses merge-patch content type, 422 for validation errors, 401/403 correctly distinguished.

**Findings:**
- All four dimensions: **PASS**
