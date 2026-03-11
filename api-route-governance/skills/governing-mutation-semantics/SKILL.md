---
name: governing-mutation-semantics
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of API mutation semantics including PUT full replacement with default reset for omitted optionals, JSON Merge Patch RFC 7396 as house standard for PATCH with application/merge-patch+json Content-Type, rationale for rejecting JSON Patch RFC 6902 and Field Masks AIP-161, array mutation via dedicated sub-resource endpoints, and exclude_unset deserialization for PATCH. Use when reviewing mutation endpoints, auditing PATCH implementation, enforcing PUT versus PATCH semantics, or implementing merge patch. Triggered by: mutation semantics, PUT replacement, PATCH semantics, JSON Merge Patch, RFC 7396, merge patch, partial update, exclude_unset, array mutation, sub-resource, PUT vs PATCH, mutation design, update semantics, field mask.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Mutation Semantics Governance

Comprehensive governance for API mutation operations, enforcing consistent PUT replacement semantics, JSON Merge Patch for PATCH, and safe array mutation strategies.

## Purpose

Mutation endpoints are the most error-prone part of any API. When PUT sometimes acts like PATCH, when PATCH uses three different formats, and when array fields are modified through unpredictable mechanisms, clients cannot safely update resources. This skill encodes canonical mutation rules: PUT for full replacement, JSON Merge Patch (RFC 7396) for partial updates, and dedicated sub-resources for array manipulation. The result is mutations that are predictable, reversible, and safe.

Use this skill when designing mutation endpoints, reviewing PATCH implementation, auditing PUT vs PATCH semantics, or implementing merge patch.

---

## PUT: Full Replacement

### Semantics

PUT replaces the entire resource with the provided representation. Every mutable field must be present in the request body.

```
PUT /projects/{id}
Content-Type: application/json

{
  "name": "Renamed Project",
  "description": "Updated description"
}
```

**Rules:**
- All REQUIRED mutable fields must be present
- Omitted OPTIONAL fields reset to their defaults (not left unchanged)
- IMMUTABLE fields must not be in the PUT body (or must match the existing value)
- OUTPUT_ONLY fields are silently ignored
- The response is the complete updated resource (200) or no body (204)

### Default Reset Behavior

The critical distinction between PUT and PATCH: omitted optional fields in PUT reset to defaults.

| Field | PUT Body | Before | After |
|-------|----------|--------|-------|
| `name` | `"New Name"` | `"Old Name"` | `"New Name"` |
| `description` | *(omitted)* | `"Some text"` | `null` (reset) |
| `nickname` | *(omitted)* | `"My Project"` | `null` (reset) |

**Implication:** Clients must send the complete desired state for PUT. If they want to change only `name` and keep everything else, they must include all existing field values. This is why PATCH exists.

Cross-reference: `governing-http-semantics` for PUT idempotency and status code selection.

---

## PATCH: JSON Merge Patch (RFC 7396)

### House Standard

This API uses JSON Merge Patch (RFC 7396) as the standard PATCH format:

```
PATCH /projects/{id}
Content-Type: application/merge-patch+json

{
  "name": "Renamed Project"
}
```

**Merge Patch Rules:**
- Present fields with values: update to the new value
- Present fields with `null`: clear the field (set to null/default)
- Absent fields: leave unchanged
- Content-Type must be `application/merge-patch+json`

| Field in Body | Value | Effect |
|---------------|-------|--------|
| `"name": "New"` | string | Update name to "New" |
| `"description": null` | null | Clear description |
| *(field absent)* | -- | Leave unchanged |

### Why JSON Merge Patch

| Format | RFC | Complexity | Array Support | Null Semantics |
|--------|-----|-----------|---------------|----------------|
| **JSON Merge Patch** | 7396 | Low | None (replace whole array) | `null` = remove |
| JSON Patch | 6902 | High | Full (add/remove/move) | Explicit operations |
| Field Masks | AIP-161 | Medium | Via mask paths | Separate mask parameter |

**JSON Merge Patch wins because:**
- Simplest mental model -- the patch looks like a partial version of the resource
- Lowest implementation complexity -- no operation arrays or mask parameters
- Sufficient for 95% of mutations -- most updates change scalar fields
- Array mutations are better handled through dedicated sub-resources (see below)

### Why Not JSON Patch (RFC 6902)

JSON Patch uses an array of operations:

```json
[
  {"op": "replace", "path": "/name", "value": "New Name"},
  {"op": "remove", "path": "/description"}
]
```

**Rejected because:**
- Operation arrays are harder to construct and validate
- Path syntax (`/name`) differs from JSON field access
- Ordering of operations introduces subtle bugs
- The complexity is not justified when array mutation has a better solution

### Why Not Field Masks (AIP-161)

Field Masks use a separate parameter to specify which fields are being updated:

```
PATCH /projects/{id}?update_mask=name,description
```

**Rejected because:**
- Requires a separate query parameter (not self-contained in the body)
- Mask path syntax adds another convention to learn
- Well-suited for protobuf/gRPC but over-engineered for JSON APIs

---

## PATCH Deserialization

### `exclude_unset=True`

When processing PATCH requests, use Pydantic's `exclude_unset=True` to distinguish between "field not sent" and "field sent as null":

```python
@router.patch("/projects/{id}")
async def patch_project(id: UUID, body: ProjectPatch):
    # Only fields explicitly sent by the client
    update_data = body.model_dump(exclude_unset=True)

    # update_data contains only the fields in the request body
    # Absent fields are NOT in update_data
    # Fields set to None ARE in update_data (meaning "clear")
```

**How `exclude_unset` works:**

```python
class ProjectPatch(BaseModel):
    name: str | None = None
    description: str | None = None

# Client sends: {"name": "New Name"}
body = ProjectPatch(name="New Name")
body.model_dump(exclude_unset=True)
# → {"name": "New Name"}
# description is NOT in the dict (it was not sent)

# Client sends: {"name": "New Name", "description": null}
body = ProjectPatch(name="New Name", description=None)
body.model_dump(exclude_unset=True)
# → {"name": "New Name", "description": None}
# description IS in the dict with None (client wants to clear it)
```

**Rules:**
- Always use `exclude_unset=True` for PATCH deserialization
- Never use `exclude_none=True` for PATCH input (it would make clearing fields impossible)
- The patch model should have all fields as `Optional` with `None` default

Cross-reference: `governing-field-lifecycle` for null vs absence semantics.

---

## Array Mutation

### Dedicated Sub-Resource Endpoints

Arrays within resources must not be mutated through PATCH. JSON Merge Patch replaces arrays entirely, which is dangerous for large or ordered collections.

**Problem with PATCH for arrays:**

```json
// Original: members = ["alice", "bob", "charlie"]
// Client wants to add "dave"
// Must send the entire array:
PATCH /projects/{id}
{"members": ["alice", "bob", "charlie", "dave"]}
// Race condition: another client removed "charlie" concurrently
```

**Solution: Dedicated sub-resource endpoints:**

```
# List members
GET    /projects/{id}/members

# Add a member
POST   /projects/{id}/members
{"user_id": "dave-uuid"}

# Remove a member
DELETE /projects/{id}/members/{member_id}

# Replace all members (rare, explicit)
PUT    /projects/{id}/members
{"items": ["alice-uuid", "bob-uuid"]}
```

**Rules:**
- Never PATCH arrays within a parent resource
- Create sub-resource endpoints for array-like relationships
- Use POST to add items, DELETE to remove items
- Use PUT on the sub-resource collection only for full replacement
- Cross-reference: `governing-route-naming` for sub-resource path design

---

## Review Dimensions

When reviewing or auditing mutation semantics, evaluate these dimensions:

### 1. PUT Semantics
- Does PUT replace the entire resource (not partial update)?
- Do omitted optional fields reset to defaults?
- Are IMMUTABLE fields excluded or validated?
- Is the response the complete updated resource?

### 2. PATCH Implementation
- Is the Content-Type `application/merge-patch+json`?
- Does the handler use `exclude_unset=True`?
- Does `null` in the body clear the field?
- Are absent fields left unchanged?

### 3. Array Mutation Strategy
- Are arrays mutated through dedicated sub-resources (not PATCH)?
- Do sub-resources follow standard CRUD patterns (POST to add, DELETE to remove)?
- Is full array replacement available via PUT on the sub-collection?

### 4. Content-Type Enforcement
- Does the server validate `Content-Type` on PATCH requests?
- Does it reject `application/json` for PATCH (must be `application/merge-patch+json`)?
- Cross-reference: `governing-http-semantics` for Content-Type negotiation

---

## Anti-Patterns

### 1. PUT That Acts Like PATCH
Leaving unchanged fields intact when they are omitted from PUT. This makes PUT and PATCH behave identically, defeating the purpose of having both.

### 2. PATCH Without exclude_unset
Using `body.model_dump()` (without `exclude_unset=True`) for PATCH. This treats absent fields as explicitly sent with their default value, overwriting existing data.

### 3. Array PATCH
Accepting `{"tags": ["new", "list"]}` in a merge patch to replace the entire tags array. Concurrent clients lose each other's additions. Use sub-resource endpoints instead.

### 4. Wrong Content-Type for PATCH
Accepting `application/json` for PATCH requests. Without `application/merge-patch+json`, the null semantics are ambiguous.

### 5. Partial PUT
Accepting a PUT body with missing required fields and only updating the provided fields. This is PATCH behavior under a PUT method, violating RFC 9110.

---

## Examples

### Example 1: PUT Acting as PATCH

**Scenario:** `PUT /projects/{id}` with `{"name": "New Name"}` only updates the name and leaves all other fields unchanged.

**Findings:**
- **PUT Semantics: FAIL** -- PUT should replace the entire resource; omitted fields should reset
- **Recommendation:** Change behavior so omitted optional fields reset to defaults. If partial update is desired, use PATCH with merge patch.

### Example 2: PATCH Without Merge Patch Content-Type

**Scenario:** `PATCH /projects/{id}` accepts `Content-Type: application/json` and uses `body.model_dump()` without `exclude_unset`.

**Findings:**
- **PATCH Implementation: FAIL** -- Wrong Content-Type and no `exclude_unset=True`
- **Content-Type Enforcement: FAIL** -- Should require `application/merge-patch+json`
- **Recommendation:** Require `application/merge-patch+json`, use `model_dump(exclude_unset=True)`

### Example 3: Well-Implemented Mutations

**Scenario:** PUT replaces fully with default reset. PATCH uses `application/merge-patch+json` with `exclude_unset=True`. Arrays managed through sub-resource endpoints.

**Findings:**
- All four dimensions: **PASS**
