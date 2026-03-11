---
name: governing-field-lifecycle
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of API field lifecycle semantics including null versus absence distinction, absent-over-null output rule, null-only-in-PATCH input rule, field behavior annotations for REQUIRED OPTIONAL OUTPUT_ONLY and IMMUTABLE, mass-assignment protection via OUTPUT_ONLY, input-output schema separation with distinct Pydantic models, and OpenAPI 3.1 nullability with required array versus type null. Use when reviewing field nullability, auditing field behavior annotations, enforcing input-output model separation, or implementing field lifecycle rules. Triggered by: field lifecycle, null vs absent, nullability, absent-over-null, OUTPUT_ONLY, IMMUTABLE, field behavior, mass assignment, input output separation, schema separation, field annotation, required fields, optional fields, Pydantic model separation, OpenAPI null.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Field Lifecycle Governance

Comprehensive governance for API field lifecycle semantics, enforcing consistent null/absence behavior, field behavior annotations, and input-output schema separation.

## Purpose

Null and absence are different concepts in API payloads, but most APIs conflate them. When `null` sometimes means "clear this field" and other times means "I didn't send this field," clients cannot reliably construct PATCH requests. When the same Pydantic model handles both creation input and GET response, server-managed fields like `id` and `created_at_time` leak into the creation schema -- creating mass-assignment vulnerabilities and confusing API documentation. This skill encodes lifecycle rules that make field behavior explicit and predictable.

Use this skill when designing field nullability, reviewing field behavior annotations, auditing input-output model separation, or implementing field lifecycle rules.

---

## Null vs Absence Distinction

### Semantic Difference

In JSON payloads, `null` and absence carry different meanings:

| Concept | JSON Representation | Meaning |
|---------|-------------------|---------|
| Null | `{"nickname": null}` | "Clear this field / set to empty" |
| Absent | `{}` (field not present) | "Don't change this field / not applicable" |

This distinction matters most for PATCH operations but applies across all contract types.

### Output Rule: Absent-Over-Null

When the server sends responses, omit fields whose value is null rather than including them as `null`:

| Correct (absent) | Wrong (explicit null) |
|-------------------|---------------------|
| `{"name": "Project A"}` | `{"name": "Project A", "nickname": null, "description": null}` |

**Rationale:**
- Reduces payload size (null fields carry no information)
- Avoids confusion about whether `null` is intentional or a default
- Simplifies client-side parsing (missing key = not applicable)

**Implementation:**
```python
@router.get(
    "/projects/{id}",
    response_model=ProjectResponse,
    response_model_exclude_none=True,  # absent-over-null
)
```

### Input Rule: Null Only in PATCH

- **POST (create):** Clients should not send `null` values. Omit optional fields instead.
- **PUT (replace):** All fields present. Optional fields may be `null` to clear.
- **PATCH (partial update):** `null` explicitly means "clear this field." Absent fields are untouched.

| Method | `null` meaning | Absent meaning |
|--------|---------------|----------------|
| POST | Not allowed (omit instead) | Use server default |
| PUT | Clear / reset to default | Clear / reset to default |
| PATCH | Clear this field | Don't change this field |

Cross-reference: `governing-mutation-semantics` for PATCH deserialization with `exclude_unset=True`.

---

## Field Behavior Annotations

Every field in an API payload has one of four behavior annotations that define how it behaves across create, read, and update operations.

### REQUIRED

The field must be present and non-null in input. The server always includes it in output.

```python
class ProjectCreate(BaseModel):
    name: str  # REQUIRED -- must be provided on creation
```

### OPTIONAL

The field may be omitted from input. The server includes it in output if it has a value (absent-over-null).

```python
class ProjectCreate(BaseModel):
    description: str | None = None  # OPTIONAL -- may be omitted
```

### OUTPUT_ONLY

The field is managed by the server. It appears in responses but must be silently ignored if present in input.

```python
class ProjectResponse(BaseModel):
    id: str                    # OUTPUT_ONLY -- server-generated
    created_at_time: datetime  # OUTPUT_ONLY -- server-managed
    updated_at_time: datetime  # OUTPUT_ONLY -- server-managed
```

**Mass-Assignment Protection:** OUTPUT_ONLY fields must never be writable through input payloads. If a client sends `"id": "fake-id"` in a creation request, the server silently ignores it -- never raises an error (which would leak implementation details) and never accepts the value.

### IMMUTABLE

The field is set once on creation and cannot be changed afterward. Attempts to change it in PUT or PATCH return `400 Bad Request`.

```python
class ProjectCreate(BaseModel):
    slug: str  # IMMUTABLE -- set on creation, cannot change
```

**Implementation:**
```python
# In the update handler:
if update_data.slug != existing.slug:
    raise HTTPException(
        status_code=400,
        detail="Field 'slug' is immutable and cannot be changed after creation."
    )
```

### Annotation Matrix

| Annotation | In Create Input | In Update Input | In Response |
|------------|-----------------|-----------------|-------------|
| REQUIRED | Must be present | Must be present | Always present |
| OPTIONAL | May be omitted | May be omitted | Present if non-null |
| OUTPUT_ONLY | Silently ignored | Silently ignored | Always present |
| IMMUTABLE | Must be present | Must match original (or absent) | Always present |

---

## Input-Output Schema Separation

### Separate Models Per Operation

Never use the same Pydantic model for input and output. Create distinct models for each contract type:

| Contract Type | Model Name | Purpose |
|---------------|------------|---------|
| Create input | `ResourceCreate` | Fields needed to create |
| Replace input | `ResourceUpdate` | All mutable fields for PUT |
| Patch input | `ResourcePatch` | Optional fields for PATCH |
| Response output | `ResourceResponse` | Full resource representation |

### Example: Project Models

```python
class ProjectCreate(BaseModel):
    """Desired state for creating a project."""
    name: str                          # REQUIRED
    slug: str                          # IMMUTABLE
    description: str | None = None     # OPTIONAL

class ProjectUpdate(BaseModel):
    """Desired state for full replacement."""
    name: str                          # REQUIRED
    description: str | None = None     # OPTIONAL
    # slug is IMMUTABLE -- not in update model

class ProjectPatch(BaseModel):
    """Partial mutation fields."""
    name: str | None = None
    description: str | None = None
    # All fields optional -- only provided fields are changed

class ProjectResponse(BaseModel):
    """State representation returned by the server."""
    id: str                            # OUTPUT_ONLY
    name: str                          # REQUIRED
    slug: str                          # IMMUTABLE
    description: str | None = None     # OPTIONAL
    status: str                        # OUTPUT_ONLY
    created_at_time: datetime          # OUTPUT_ONLY
    updated_at_time: datetime          # OUTPUT_ONLY
```

**Why separate models matter:**
- `ProjectCreate` does not include `id` or `created_at_time` -- preventing mass assignment
- `ProjectUpdate` does not include `slug` -- preventing immutable field changes
- `ProjectPatch` makes every field optional -- supporting partial updates
- `ProjectResponse` includes all fields -- giving consumers complete state

Cross-reference: `governing-schema-implementation` for model naming conventions and Pydantic configuration.

---

## OpenAPI 3.1 Nullability

### Two Independent Axes

In OpenAPI 3.1 (JSON Schema 2020-12), "required" and "nullable" are independent concepts:

| Concept | Mechanism | Meaning |
|---------|-----------|---------|
| Required | `required: [field_name]` array | Field must be present in the payload |
| Nullable | `type: ["string", "null"]` | Field's value may be `null` |

### Combinations

| required | nullable | Meaning | Example |
|----------|----------|---------|---------|
| Yes | No | Must be present and non-null | `name: str` |
| Yes | Yes | Must be present, may be null | `nickname: str \| None` (in PUT/PATCH) |
| No | No | May be omitted, non-null if present | `description: str = None` with `exclude_none` |
| No | Yes | May be omitted or null | `nickname: str \| None = None` (in PATCH) |

### Pydantic to OpenAPI Mapping

```python
# Required, non-nullable → required: [name], type: string
name: str

# Required, nullable → required: [nickname], type: [string, null]
nickname: str | None

# Optional, non-nullable → not in required, type: string, default: null
tag: str = None  # Caution: Pydantic treats this as Optional

# Optional, nullable → not in required, type: [string, null], default: null
description: str | None = None
```

---

## Review Dimensions

When reviewing or auditing field lifecycle, evaluate these dimensions:

### 1. Null/Absence Semantics
- Does the API use `response_model_exclude_none=True` for absent-over-null?
- Is `null` reserved for PATCH "clear field" semantics?
- Are POST requests free of `null` values (using omission instead)?

### 2. Field Behavior Annotations
- Are OUTPUT_ONLY fields excluded from input models?
- Are IMMUTABLE fields validated against changes in update handlers?
- Does the API silently ignore OUTPUT_ONLY fields in input (not error)?

### 3. Schema Separation
- Are there distinct Pydantic models for create, update, patch, and response?
- Do input models exclude server-managed fields (`id`, `created_at_time`)?
- Do patch models make all fields optional?

### 4. OpenAPI Nullability
- Does the OpenAPI spec correctly represent required vs nullable for each field?
- Are nullable fields using `type: ["string", "null"]` (not the deprecated `nullable: true`)?
- Does the `required` array match field behavior annotations?

---

## Anti-Patterns

### 1. Single Model for Everything
Using one `Project` model for create, update, and response. This forces `id: Optional[str] = None` in the create schema, which is confusing and allows mass assignment.

### 2. Explicit Nulls in Responses
Returning `{"nickname": null, "avatar_url": null}` instead of omitting null fields. Wastes bandwidth and confuses "intentionally null" with "not applicable."

### 3. Error on OUTPUT_ONLY Input
Returning `400 Bad Request` when a client sends `"id": "..."` in a create payload. This leaks that `id` is significant. Silently ignore it instead.

### 4. Mutable Immutables
Allowing `slug` to be changed via PUT or PATCH after creation. Slugs in URLs become broken links. Enforce immutability with validation.

### 5. Nullable Without Context
Making every field `str | None = None` "just in case." This makes the OpenAPI schema useless -- consumers cannot tell which fields are truly optional.

---

## Examples

### Example 1: Single Model for Input and Output

**Scenario:** The API uses a single `Project` Pydantic model for POST input and GET response. The model includes `id: Optional[str] = None` and `created_at: Optional[datetime] = None`.

**Findings:**
- **Schema Separation: FAIL** -- Same model for input and output exposes server-managed fields in creation schema
- **Field Behavior Annotations: FAIL** -- No OUTPUT_ONLY protection; `id` is writable in creation payload
- **Recommendation:** Split into `ProjectCreate` (without `id`, `created_at`) and `ProjectResponse` (with all fields)

### Example 2: Well-Separated Schemas

**Scenario:** Distinct `ProjectCreate`, `ProjectUpdate`, `ProjectPatch`, and `ProjectResponse` models. `response_model_exclude_none=True` on all GET endpoints. IMMUTABLE fields validated in update handlers. OUTPUT_ONLY fields silently ignored in input.

**Findings:**
- All four dimensions: **PASS**
