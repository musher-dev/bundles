---
name: governing-schema-implementation
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of Pydantic V2 schema implementation for API payloads including response_model enforcement and mass-assignment security, response_model_exclude_none for absent-over-null compliance, JsonSchemaMode for validation versus serialization schemas, polymorphism via Discriminator and OpenAPI oneOf, Field examples for example validation, alias_generator with to_camel and populate_by_name, and model naming conventions for ResourceCreate ResourceUpdate ResourcePatch and ResourceResponse. Use when reviewing Pydantic models, auditing schema configuration, enforcing model naming conventions, or implementing response model patterns. Triggered by: schema implementation, Pydantic model, response_model, exclude_none, JsonSchemaMode, Discriminator, oneOf, alias_generator, model naming, ResourceCreate, ResourceUpdate, ResourcePatch, ResourceResponse, mass assignment, Pydantic V2, schema config, model config.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Schema Implementation Governance

Comprehensive governance for Pydantic V2 schema implementation in API applications, enforcing consistent model configuration, naming conventions, and security patterns.

## Purpose

Pydantic models are the bridge between Python code and the OpenAPI spec. Misconfigured models produce incorrect schemas, leak internal fields, create mass-assignment vulnerabilities, and generate confusing API documentation. This skill encodes canonical Pydantic V2 configuration patterns that ensure models are secure, accurate, and produce clean OpenAPI specifications.

Use this skill when reviewing Pydantic models, auditing schema configuration, enforcing model naming conventions, or implementing response model patterns.

---

## Response Model Enforcement

### `response_model` on Every Endpoint

Every endpoint must declare a `response_model` to ensure the OpenAPI spec accurately describes the response shape and to activate Pydantic's output serialization:

```python
@router.get("/projects/{id}", response_model=ProjectResponse)
async def get_project(id: UUID) -> ProjectResponse:
    project = await project_repo.get(id)
    return project  # Pydantic serializes through ProjectResponse
```

**What `response_model` does:**
- Generates the response schema in the OpenAPI spec
- Filters the return value through the model (output serialization)
- Strips fields not declared in the model (mass-assignment protection)
- Applies serialization rules (alias generation, exclude_none)

### Mass-Assignment Security

Without `response_model`, the endpoint returns whatever the handler returns -- potentially including internal fields:

```python
# DANGEROUS: No response_model -- leaks internal fields
@router.get("/projects/{id}")
async def get_project(id: UUID):
    return await project_repo.get(id)  # May include password_hash, internal_notes, etc.

# SAFE: response_model filters output
@router.get("/projects/{id}", response_model=ProjectResponse)
async def get_project(id: UUID) -> ProjectResponse:
    return await project_repo.get(id)  # Only ProjectResponse fields are returned
```

**Rules:**
- Every endpoint must have `response_model`
- The response model must be a dedicated output model (not the ORM/database model)
- Never return ORM objects directly without response_model filtering

Cross-reference: `governing-openapi-contracts` for response model declarations in the OpenAPI spec.

---

## Absent-Over-Null Compliance

### `response_model_exclude_none=True`

Configure endpoints to omit null fields from responses, enforcing the absent-over-null rule:

```python
@router.get(
    "/projects/{id}",
    response_model=ProjectResponse,
    response_model_exclude_none=True,
)
async def get_project(id: UUID) -> ProjectResponse:
    ...
```

**Effect:**

```python
# If project.description is None:

# Without exclude_none:
{"id": "...", "name": "Project", "description": null}

# With exclude_none:
{"id": "...", "name": "Project"}
# description is absent -- cleaner and more compact
```

**Rules:**
- Apply `response_model_exclude_none=True` on all GET endpoints
- Apply on POST/PUT responses that return the created/updated resource
- Do not apply on PATCH input processing (null has meaning in PATCH)

Cross-reference: `governing-field-lifecycle` for the full absent-over-null semantics.

---

## JsonSchemaMode: Validation vs Serialization

### Two Schema Modes

Pydantic V2 generates different JSON schemas depending on the mode:

| Mode | Purpose | When Used |
|------|---------|-----------|
| `validation` | Schema for validating input | Request body schemas in OpenAPI |
| `serialization` | Schema for describing output | Response body schemas in OpenAPI |

**Why this matters:** A field with `alias_generator=to_camel` has different valid keys in input (both `snake_case` and `camelCase` with `populate_by_name=True`) versus output (only `camelCase`).

```python
# Generate validation schema (for request bodies)
schema = ProjectCreate.model_json_schema(mode="validation")

# Generate serialization schema (for response bodies)
schema = ProjectResponse.model_json_schema(mode="serialization")
```

**Rules:**
- FastAPI handles mode selection automatically when configured correctly
- Custom schema generation must specify the correct mode
- Test both modes when using alias generators or custom serializers

---

## Polymorphism via Discriminator

### Tagged Unions with `Discriminator`

When an endpoint accepts or returns multiple payload types, use Pydantic's `Discriminator` to produce clean OpenAPI `oneOf` schemas:

```python
from pydantic import BaseModel, Discriminator, Tag
from typing import Annotated, Union

class EmailNotification(BaseModel):
    type: Literal["email"] = "email"
    recipient_email: str
    subject: str

class SlackNotification(BaseModel):
    type: Literal["slack"] = "slack"
    channel_id: str
    message: str

Notification = Annotated[
    Union[
        Annotated[EmailNotification, Tag("email")],
        Annotated[SlackNotification, Tag("slack")],
    ],
    Discriminator("type"),
]
```

**OpenAPI output:**
```yaml
oneOf:
  - $ref: '#/components/schemas/EmailNotification'
  - $ref: '#/components/schemas/SlackNotification'
discriminator:
  propertyName: type
  mapping:
    email: '#/components/schemas/EmailNotification'
    slack: '#/components/schemas/SlackNotification'
```

**Rules:**
- Use a `type` field as the discriminator (string literal)
- Each variant must have a `Literal` default for the discriminator field
- Prefer `Discriminator` over bare `Union` for cleaner OpenAPI output
- Cross-reference: `governing-openapi-contracts` for `oneOf` representation in specs

---

## Field Examples

### `Field(examples=[...])`

Provide example values in Pydantic fields for OpenAPI documentation and validation:

```python
class ProjectCreate(BaseModel):
    name: str = Field(
        ...,
        min_length=1,
        max_length=100,
        examples=["My Awesome Project"],
    )
    description: str | None = Field(
        default=None,
        max_length=1000,
        examples=["A project for building awesome things"],
    )
```

**Rules:**
- Use `examples` (plural, list) not `example` (deprecated in Pydantic V2)
- Examples must be valid values that pass the field's own validators
- Include examples for fields with non-obvious formats (UUIDs, dates, enums)
- Examples appear in the OpenAPI spec and Swagger UI "Try it out" feature

---

## Alias Generator Configuration

### camelCase Wire Format

When the API wire format requires camelCase (JavaScript ecosystem integration), configure `alias_generator`:

```python
from pydantic import BaseModel, ConfigDict
from pydantic.alias_generators import to_camel

class ProjectResponse(BaseModel):
    model_config = ConfigDict(
        alias_generator=to_camel,
        populate_by_name=True,
    )

    project_name: str          # Serialized as "projectName"
    created_at_time: datetime  # Serialized as "createdAtTime"
    owner_id: str              # Serialized as "ownerId"
```

**Configuration options:**

| Setting | Purpose |
|---------|---------|
| `alias_generator=to_camel` | Convert snake_case field names to camelCase in JSON |
| `populate_by_name=True` | Accept both snake_case and camelCase in input |

**Rules:**
- Apply alias_generator on a base model that all API models inherit from
- `populate_by_name=True` is required to allow Python code to use snake_case
- Test that the OpenAPI spec uses the aliased names (camelCase)
- The Python codebase always uses snake_case -- aliases are wire format only

Cross-reference: `governing-field-serialization` for field naming conventions.

---

## Model Naming Conventions

### Standard Naming Pattern

API Pydantic models follow the `Resource{Operation}` naming convention:

| Model | Convention | Purpose | Example |
|-------|-----------|---------|---------|
| Create input | `ResourceCreate` | Fields for POST creation | `ProjectCreate` |
| Full update input | `ResourceUpdate` | Fields for PUT replacement | `ProjectUpdate` |
| Partial update input | `ResourcePatch` | Optional fields for PATCH | `ProjectPatch` |
| Response output | `ResourceResponse` | Complete resource for GET | `ProjectResponse` |

**Rules:**
- `Resource` is the singular noun matching the API resource name
- Suffixes are `Create`, `Update`, `Patch`, `Response` (not `In`, `Out`, `DTO`, `Schema`)
- Each model is a separate class (not generated or inherited in ways that obscure the schema)
- Command models use descriptive names: `TransferProjectCommand`, `ArchiveProjectCommand`
- Collection response models (if needed): `ProjectListResponse`

### Model Inheritance Strategy

Use composition over deep inheritance for API models:

```python
# Shared fields as a mixin (not a base model)
class ProjectFields:
    name: str
    description: str | None = None

# Each model is explicit about its fields
class ProjectCreate(BaseModel, ProjectFields):
    slug: str  # IMMUTABLE -- only in create

class ProjectResponse(BaseModel, ProjectFields):
    id: str
    slug: str
    status: str
    created_at_time: datetime
    updated_at_time: datetime
```

**Rules:**
- Avoid deep inheritance chains that make schemas hard to follow
- Each model must be self-documenting -- a developer reading the model should understand all its fields
- Shared fields can use mixins, but each final model must be explicit

Cross-reference: `governing-field-lifecycle` for field behavior annotations (REQUIRED, OPTIONAL, OUTPUT_ONLY, IMMUTABLE).

---

## Review Dimensions

When reviewing or auditing schema implementation, evaluate these dimensions:

### 1. Response Model Security
- Does every endpoint declare a `response_model`?
- Is the response model a dedicated output model (not ORM)?
- Are internal fields excluded from response models?

### 2. Serialization Configuration
- Is `response_model_exclude_none=True` applied on GET endpoints?
- Are alias generators consistent across all models?
- Is `populate_by_name=True` set when using aliases?

### 3. Schema Accuracy
- Does the OpenAPI spec match the actual response shape?
- Are polymorphic types using `Discriminator` (producing `oneOf`)?
- Do `Field(examples=[...])` values pass validation?

### 4. Model Naming
- Do models follow `ResourceCreate`/`ResourceUpdate`/`ResourcePatch`/`ResourceResponse` convention?
- Are command models descriptively named?
- Is each model a separate class (not a dynamically generated variant)?

---

## Anti-Patterns

### 1. Missing response_model
Endpoints without `response_model` return unfiltered data, potentially including sensitive fields. Always declare the response model.

### 2. ORM Model as Response
Using the SQLAlchemy/database model as the response model. This couples the API schema to the database schema and risks leaking columns.

### 3. Shared Input/Output Model
Using one model for both request and response. This requires `Optional` on server-managed fields, creating confusing schemas.

### 4. exclude_none on PATCH Input
Applying `exclude_none` when processing PATCH input. This makes it impossible to clear fields by sending `null`.

### 5. Deep Inheritance Chains
`BaseModel` → `TimestampMixin` → `TenantMixin` → `ProjectBase` → `ProjectResponse`. Each layer hides fields, making the final schema opaque.

### 6. Non-Standard Naming
Using `ProjectDTO`, `ProjectSchema`, `ProjectIn`, `ProjectOut` instead of `ProjectCreate`, `ProjectResponse`. Non-standard suffixes confuse developers familiar with the convention.

---

## Examples

### Example 1: No Response Model

**Scenario:** Endpoints return ORM objects directly without `response_model`. The `/users/{id}` endpoint returns `password_hash` in the response.

**Findings:**
- **Response Model Security: FAIL** -- No response_model, internal fields leaked
- **Recommendation:** Create `UserResponse` model excluding sensitive fields; add `response_model=UserResponse` to all user endpoints

### Example 2: Well-Configured Schema Implementation

**Scenario:** Every endpoint has `response_model` with a dedicated response class. `response_model_exclude_none=True` on all GETs. Models follow `ResourceCreate`/`ResourceResponse` naming. Alias generator applied consistently. Discriminator used for polymorphic types.

**Findings:**
- All four dimensions: **PASS**
