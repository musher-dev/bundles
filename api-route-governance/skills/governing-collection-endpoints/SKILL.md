---
name: governing-collection-endpoints
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of API collection endpoint standards including cursor-based pagination, page size limits, filter and sort query parameters, response envelope with data and meta and links fields, and offset pagination anti-pattern. Use when designing list endpoints, reviewing pagination, auditing collection responses, or implementing filtering and sorting. Triggered by: collection endpoint, pagination, cursor-based, page size, filter, sort, response envelope, list endpoint, offset pagination, cursor pagination, data meta links, query parameter, paginated response.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Collection Endpoint Governance

Comprehensive governance for API collection (list) endpoint design, enforcing consistent pagination, filtering, sorting, and response envelope standards.

## Purpose

Collection endpoints are the most frequently called and most performance-sensitive routes in any API. A poorly designed list endpoint -- using offset pagination, returning unbounded results, or providing no filtering -- creates cascading problems: database strain from `OFFSET` scans, client memory exhaustion from oversized payloads, and developer frustration from inconsistent response shapes. This skill enforces collection endpoint standards that scale predictably.

Use this skill when designing list endpoints, reviewing pagination strategies, auditing collection response shapes, or implementing filtering and sorting.

---

## Response Envelope

### Standard Collection Response Shape

All collection endpoints must return a consistent envelope:

```json
{
  "data": [
    { "id": "a1b2c3", "name": "Project Alpha", "created_at": "2025-01-15T10:00:00Z" },
    { "id": "d4e5f6", "name": "Project Beta", "created_at": "2025-01-14T09:00:00Z" }
  ],
  "meta": {
    "page_size": 20,
    "has_next": true,
    "has_previous": false,
    "total_count": 142
  },
  "links": {
    "next": "/api/v1/projects?cursor=eyJpZCI6ImQ0ZTVmNiJ9&page_size=20",
    "previous": null,
    "self": "/api/v1/projects?page_size=20"
  }
}
```

### Envelope Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `data` | array | Yes | The array of resource objects |
| `meta` | object | Yes | Pagination metadata |
| `links` | object | Yes | Navigation URLs for pagination |

### `meta` Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `page_size` | integer | Yes | Number of items per page |
| `has_next` | boolean | Yes | Whether more items exist after this page |
| `has_previous` | boolean | Yes | Whether items exist before this page |
| `total_count` | integer | Optional | Total number of items matching the query |

**`total_count` note:** Computing total count requires a separate `COUNT(*)` query that can be expensive on large tables. Make it optional -- include when cheap (cached, materialized), omit when expensive. Document when it's absent.

### `links` Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `self` | string | Yes | URL of the current page |
| `next` | string or null | Yes | URL of the next page (null if no more pages) |
| `previous` | string or null | Yes | URL of the previous page (null if first page) |

---

## Cursor-Based Pagination

### Why Cursors Over Offsets

| Aspect | Cursor-Based | Offset-Based |
|--------|-------------|--------------|
| Performance | O(1) -- seeks to cursor position | O(n) -- scans and discards offset rows |
| Consistency | Stable under concurrent inserts/deletes | Rows shift, causing duplicates or skips |
| Deep pages | Same speed as first page | Progressively slower |
| Implementation | Slightly more complex | Simple but dangerous |

### Cursor Design

A cursor is an opaque, base64-encoded pointer to a position in the result set:

```python
import base64
import json

def encode_cursor(last_item: dict) -> str:
    payload = {"id": str(last_item["id"]), "created_at": last_item["created_at"].isoformat()}
    return base64.urlsafe_b64encode(json.dumps(payload).encode()).decode()

def decode_cursor(cursor: str) -> dict:
    return json.loads(base64.urlsafe_b64decode(cursor.encode()).decode())
```

**Rules:**
- Cursors are opaque to clients -- never document the internal format
- Cursors encode the sort key(s) and a tiebreaker (usually `id`)
- Cursors must be URL-safe (use base64url encoding)
- Invalid cursors return 400 Bad Request, not 500

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `cursor` | string | None | Opaque cursor from a previous response |
| `page_size` | integer | 20 | Number of items per page |

**Page size constraints:**
- Minimum: 1
- Maximum: 100
- Default: 20
- Values outside bounds are clamped (not rejected)

### SQL Pattern

```sql
-- First page (no cursor)
SELECT * FROM projects
WHERE org_id = $1
ORDER BY created_at DESC, id DESC
LIMIT $2 + 1;  -- Fetch one extra to determine has_next

-- Subsequent pages (with cursor)
SELECT * FROM projects
WHERE org_id = $1
  AND (created_at, id) < ($3, $4)  -- Cursor values
ORDER BY created_at DESC, id DESC
LIMIT $2 + 1;
```

**The +1 trick:** Fetch `page_size + 1` rows. If you get `page_size + 1` results, `has_next` is true and you drop the extra row. If you get `page_size` or fewer, `has_next` is false.

---

## Filtering

### Query Parameter Conventions

Filters are query parameters using the field name directly:

```
GET /projects?status=active&org_id=a1b2c3
```

### Filter Operators

For advanced filtering, use field-operator suffixes:

| Operator | Pattern | Example | SQL |
|----------|---------|---------|-----|
| Equality (default) | `?field=value` | `?status=active` | `WHERE status = 'active'` |
| Not equal | `?field__ne=value` | `?status__ne=archived` | `WHERE status != 'archived'` |
| Greater than | `?field__gt=value` | `?created_at__gt=2025-01-01` | `WHERE created_at > '2025-01-01'` |
| Less than | `?field__lt=value` | `?created_at__lt=2025-06-01` | `WHERE created_at < '2025-06-01'` |
| In list | `?field__in=a,b,c` | `?status__in=active,pending` | `WHERE status IN ('active', 'pending')` |

### Filter Rules

- Only allow filtering on indexed columns (reject unindexed filters with 400)
- Validate filter values against field types (reject `?created_at=banana`)
- Document which fields are filterable per endpoint
- Use allowlists, not blocklists, for filterable fields

---

## Sorting

### Query Parameter Convention

```
GET /projects?sort_by=created_at&sort_order=desc
```

| Parameter | Values | Default |
|-----------|--------|---------|
| `sort_by` | Any sortable field name | `created_at` |
| `sort_order` | `asc`, `desc` | `desc` |

### Multi-Field Sort

For complex sorting, use comma-separated fields with optional direction prefix:

```
GET /projects?sort=-created_at,name
```

Where `-` prefix means descending, no prefix means ascending.

### Sort Rules

- Only allow sorting on indexed columns
- Document which fields are sortable per endpoint
- Default sort must be deterministic (include a tiebreaker like `id`)
- Sorting must align with cursor pagination (cursor encodes sort key values)

---

## Review Dimensions

When reviewing or auditing collection endpoints, evaluate these dimensions:

### 1. Response Envelope
- Do collection responses use the `data`/`meta`/`links` envelope?
- Is the envelope shape consistent across all collection endpoints?
- Are bare arrays returned from any list endpoint?

### 2. Pagination Strategy
- Is cursor-based pagination used (not offset)?
- Are cursors opaque to clients?
- Is `page_size` bounded with min/max/default?
- Does the response include `has_next` and navigation links?

### 3. Filtering
- Are filter parameters documented per endpoint?
- Is filtering restricted to indexed columns?
- Are filter values validated against field types?

### 4. Sorting
- Is a default sort order defined and deterministic?
- Are sortable fields documented?
- Does sorting align with cursor pagination keys?

---

## Anti-Patterns

### 1. Offset Pagination

Using `?page=5&per_page=20` which translates to `OFFSET 80 LIMIT 20`. The database must scan and discard 80 rows before returning 20.

**Problems:**
- Page 1000 scans 20,000 rows to return 20
- Concurrent inserts shift rows, causing duplicates or missed items
- Encourages deep pagination that strains the database

**Exception:** Offset pagination is acceptable for admin dashboards with small, static datasets where users need to jump to arbitrary pages.

### 2. Unbounded Responses

List endpoints that return all matching items with no pagination. A collection with 100,000 items returns a 50MB response that crashes mobile clients and times out proxies.

### 3. Bare Array Responses

Returning `[{...}, {...}]` instead of `{ "data": [{...}, {...}], "meta": {...} }`. Bare arrays cannot include pagination metadata and are vulnerable to JSON array root injection attacks.

### 4. Inconsistent Envelopes

Some endpoints return `{ "results": [...] }`, others return `{ "data": [...] }`, others return `{ "items": [...] }`. Pick one shape and enforce it everywhere.

### 5. Total Count on Every Request

Computing `COUNT(*)` for every paginated response. On large tables, this adds a full table scan to every list call. Make total count optional or cached.

### 6. Filterable But Unindexed

Allowing `?email=alice@example.com` as a filter when `email` has no index. This triggers a sequential scan on every filtered request.

---

## Examples

### Example 1: Offset-Based Pagination

**Scenario:** `GET /projects?page=3&per_page=50` returns items 101-150 using SQL `OFFSET 100 LIMIT 50`.

**Findings:**
- **Pagination Strategy: FAIL** -- Offset pagination degrades on deep pages and is inconsistent under concurrent writes
- **Recommendation:** Migrate to cursor-based pagination using `cursor` and `page_size` parameters

### Example 2: Bare Array Response

**Scenario:** `GET /projects` returns `[{"id": "..."}, ...]` with no envelope, metadata, or pagination links.

**Findings:**
- **Response Envelope: FAIL** -- No `data`/`meta`/`links` structure
- **Pagination Strategy: FAIL** -- No pagination at all (unbounded response)
- **Recommendation:** Wrap in standard envelope, add cursor-based pagination with max page_size of 100

### Example 3: Well-Designed Collection Endpoint

**Scenario:** Returns `{ "data": [...], "meta": { "page_size": 20, "has_next": true }, "links": { "next": "...", "self": "..." } }` with cursor pagination, indexed filters, and deterministic sort.

**Findings:**
- All four dimensions: **PASS**
