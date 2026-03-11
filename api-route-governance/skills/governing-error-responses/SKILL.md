---
name: governing-error-responses
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of API error response payloads following RFC 9457 Problem Details including application/problem+json content type, required fields (type, title, status, detail, instance), errors array extension for field-level validation, and FastAPI exception handler mapping. Use when designing error responses, reviewing error payloads, auditing problem details compliance, or implementing structured error handling. Triggered by: error response, problem details, RFC 9457, application/problem+json, error payload, validation error, field error, exception handler, error format, API error, structured error, error contract.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Error Response Governance

Comprehensive governance for API error response payloads, enforcing RFC 9457 Problem Details compliance across all error endpoints.

## Purpose

Error responses are the most important part of an API's developer experience. When things go wrong -- and they always do -- developers rely on structured, predictable error payloads to diagnose and fix problems. An API that returns unstructured strings, inconsistent shapes, or bare status codes forces consumers into trial-and-error debugging. This skill enforces RFC 9457 Problem Details as the universal error format.

Use this skill when designing error response structures, reviewing error payloads, auditing problem details compliance, or implementing exception handlers.

---

## RFC 9457 Problem Details

### Content-Type

All error responses must use:

```
Content-Type: application/problem+json
```

This signals to clients that the response body follows the Problem Details schema, enabling automated error parsing.

### Required Fields

Every error response must include these five fields:

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `type` | string (URI) | A URI reference identifying the error type | `"https://api.example.com/errors/validation-failed"` |
| `title` | string | A short, human-readable summary | `"Validation Failed"` |
| `status` | integer | The HTTP status code | `422` |
| `detail` | string | A human-readable explanation specific to this occurrence | `"The email field must be a valid email address"` |
| `instance` | string (URI) | A URI reference identifying this specific occurrence | `"/api/v1/projects/a1b2c3d4/members"` |

### Field Rules

**`type`:**
- Must be a URI that, when dereferenced, provides human-readable documentation of the error type
- Use `"about:blank"` only when the `title` alone is sufficient (e.g., 404 Not Found)
- Namespace types under your API domain: `https://api.example.com/errors/{error-type}`
- Types are stable identifiers -- once published, they must not change meaning

**`title`:**
- Must not change between occurrences of the same error type
- Should be short enough for UI display (under 60 characters)
- Is the human label for the `type` URI

**`status`:**
- Must match the HTTP response status code exactly
- Exists for convenience -- clients that lose access to the HTTP status line can still read it from the body

**`detail`:**
- Should be specific to this occurrence (not a generic message)
- May include dynamic values: `"Project 'acme' already exists in organization 'corp'"`
- Must not expose internal implementation details (stack traces, SQL queries, internal paths)

**`instance`:**
- Should identify the specific request or resource that triggered the error
- Use the request URI path or a unique error reference ID
- Useful for support requests: "Error on instance `/api/v1/projects/abc123`"

---

## Validation Errors Extension

### The `errors` Array

For 422 Unprocessable Content responses with multiple field-level validation errors, extend Problem Details with an `errors` array:

```json
{
  "type": "https://api.example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 422,
  "detail": "The request body contains 2 validation errors",
  "instance": "/api/v1/organizations/acme/projects",
  "errors": [
    {
      "field": "name",
      "message": "Project name must be between 1 and 100 characters",
      "code": "string_too_long",
      "value": ""
    },
    {
      "field": "slug",
      "message": "Slug 'my project' contains invalid characters. Use lowercase letters, numbers, and hyphens only",
      "code": "invalid_format",
      "value": "my project"
    }
  ]
}
```

### Error Array Item Schema

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `field` | string | Yes | JSON path to the invalid field (`"name"`, `"address.zip_code"`, `"items[0].quantity"`) |
| `message` | string | Yes | Human-readable description of the validation failure |
| `code` | string | Yes | Machine-readable error code for programmatic handling |
| `value` | any | No | The rejected value (omit for sensitive fields like passwords) |

### Standard Validation Codes

| Code | Meaning | Example |
|------|---------|---------|
| `required` | Field is missing | `"field": "email"` |
| `invalid_format` | Value does not match expected format | Invalid email, UUID, date |
| `string_too_short` | Below minimum length | `"name"` with 0 characters |
| `string_too_long` | Exceeds maximum length | `"bio"` over 500 characters |
| `number_out_of_range` | Outside min/max bounds | Quantity < 0 |
| `invalid_enum` | Value not in allowed set | Status "unknown" |
| `duplicate` | Value already exists | Duplicate email |
| `not_found` | Referenced resource does not exist | Invalid `project_id` |

---

## Error Type Registry

### Standard Error Types

Maintain a registry of error types with stable URIs:

| Type URI Suffix | Status | Title | When |
|----------------|--------|-------|------|
| `/errors/validation-failed` | 422 | Validation Failed | Request body validation errors |
| `/errors/not-found` | 404 | Not Found | Resource does not exist |
| `/errors/conflict` | 409 | Conflict | Duplicate creation, state conflict |
| `/errors/unauthorized` | 401 | Unauthorized | Missing or invalid credentials |
| `/errors/forbidden` | 403 | Forbidden | Insufficient permissions |
| `/errors/rate-limited` | 429 | Rate Limit Exceeded | Too many requests |
| `/errors/internal` | 500 | Internal Server Error | Unhandled server failure |

---

## FastAPI Exception Handler Mapping

### Custom Exception Classes

Map application exceptions to Problem Details responses:

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class ProblemDetailError(Exception):
    def __init__(
        self,
        status: int,
        error_type: str,
        title: str,
        detail: str,
        instance: str | None = None,
        errors: list[dict] | None = None,
    ):
        self.status = status
        self.error_type = error_type
        self.title = title
        self.detail = detail
        self.instance = instance
        self.errors = errors


async def problem_detail_handler(request: Request, exc: ProblemDetailError) -> JSONResponse:
    body = {
        "type": f"https://api.example.com{exc.error_type}",
        "title": exc.title,
        "status": exc.status,
        "detail": exc.detail,
        "instance": exc.instance or str(request.url.path),
    }
    if exc.errors:
        body["errors"] = exc.errors
    return JSONResponse(
        status_code=exc.status,
        content=body,
        media_type="application/problem+json",
    )
```

### FastAPI Integration

```python
from fastapi import FastAPI

app = FastAPI()
app.add_exception_handler(ProblemDetailError, problem_detail_handler)
```

### Overriding Default FastAPI Errors

FastAPI's built-in `RequestValidationError` returns a non-standard format. Override it:

```python
from fastapi.exceptions import RequestValidationError

async def validation_error_handler(request: Request, exc: RequestValidationError) -> JSONResponse:
    errors = []
    for error in exc.errors():
        field = ".".join(str(loc) for loc in error["loc"] if loc != "body")
        errors.append({
            "field": field,
            "message": error["msg"],
            "code": error["type"],
        })
    return JSONResponse(
        status_code=422,
        content={
            "type": "https://api.example.com/errors/validation-failed",
            "title": "Validation Failed",
            "status": 422,
            "detail": f"The request body contains {len(errors)} validation error(s)",
            "instance": str(request.url.path),
            "errors": errors,
        },
        media_type="application/problem+json",
    )

app.add_exception_handler(RequestValidationError, validation_error_handler)
```

---

## Review Dimensions

When reviewing or auditing error responses, evaluate these dimensions:

### 1. Content-Type
- Do error responses use `application/problem+json`?
- Is `Content-Type` set explicitly (not defaulting to `application/json`)?

### 2. Required Fields
- Do all error responses include `type`, `title`, `status`, `detail`, and `instance`?
- Is `status` consistent with the HTTP status code?
- Does `type` use a stable, dereferenceable URI?

### 3. Validation Errors
- Do 422 responses include an `errors` array with field-level details?
- Do error items include `field`, `message`, and `code`?
- Are sensitive values (passwords) omitted from the `value` field?

### 4. Security
- Do error details avoid leaking internal implementation (stack traces, SQL, file paths)?
- Are 500 errors generic to clients while detailed in server logs?
- Do 404 responses avoid confirming resource existence to unauthorized users?

---

## Anti-Patterns

### 1. Unstructured Error Strings
Returning `{ "error": "something went wrong" }` with no type, status, or field details. Clients cannot programmatically distinguish error types or display field-level feedback.

### 2. Inconsistent Error Shapes
Different endpoints returning errors in different formats: `{ "error": "..." }`, `{ "message": "..." }`, `{ "detail": "..." }`. Clients need a single parsing strategy.

### 3. Stack Traces in Production
Returning Python tracebacks or SQL errors to clients. This leaks internal architecture and aids attackers. Log details server-side; return generic Problem Details to clients.

### 4. 200 with Error Body
Returning `200 OK` with `{ "success": false, "error": "..." }`. HTTP status codes exist for this purpose. Clients should not parse the body to determine if the request succeeded.

### 5. Missing Field-Level Errors
Returning `422` with `"detail": "Validation failed"` but no field-level breakdown. The client cannot highlight which form fields need correction.

### 6. Non-Standard FastAPI Validation
Relying on FastAPI's default `RequestValidationError` format, which returns a `detail` array with Pydantic-specific `loc`/`msg`/`type` structure. This format is framework-specific and not RFC 9457 compliant.

---

## Examples

### Example 1: Bare Error String

**Scenario:** API returns `{ "error": "Invalid input" }` with status 400 for all client errors.

**Findings:**
- **Content-Type: FAIL** -- Uses `application/json` instead of `application/problem+json`
- **Required Fields: FAIL** -- Missing `type`, `title`, `status`, `detail`, `instance`
- **Recommendation:** Implement RFC 9457 Problem Details for all error responses with proper Content-Type.

### Example 2: Good Problem Details, Missing Validation Array

**Scenario:** 422 response includes `type`, `title`, `status`, `detail`, `instance` but no `errors` array for field-level details.

**Findings:**
- **Required Fields: PASS** -- All five fields present
- **Validation Errors: FAIL** -- No field-level error breakdown for 422
- **Recommendation:** Add `errors` array with `field`, `message`, and `code` for each validation failure.

### Example 3: Fully Compliant Error Responses

**Scenario:** All errors use `application/problem+json`, include all five required fields, 422 responses have `errors` array, no internal details leaked.

**Findings:**
- All four dimensions: **PASS**
