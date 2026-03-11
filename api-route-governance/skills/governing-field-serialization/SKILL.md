---
name: governing-field-serialization
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of API field serialization formats including RFC 3339 timestamps with _time suffix, ISO 8601 durations with _duration suffix, money as embedded objects with currency_code units and nanos, opaque string identifiers, UPPER_SNAKE_CASE enum strings, snake_case field naming with camelCase alias escape hatch, base64url binary encoding, and plural naming for array fields. Use when reviewing field formats, auditing serialization conventions, enforcing type representations, or implementing field naming rules. Triggered by: field serialization, timestamp format, RFC 3339, ISO 8601, money field, currency, enum format, snake_case fields, camelCase alias, base64url, binary encoding, field naming, type representation, serialization format, duration format.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Field Serialization Governance

Comprehensive governance for API field serialization formats, enforcing consistent type representations, naming conventions, and encoding standards across all payloads.

## Purpose

JSON has only six data types, but APIs must represent timestamps, durations, money, enums, binary data, and identifiers. When each developer invents their own serialization format -- Unix timestamps vs ISO strings, float dollars vs integer cents, numeric enums vs string enums -- interoperability collapses. This skill encodes canonical serialization formats that eliminate ambiguity and ensure every field type is represented consistently.

Use this skill when designing field formats, reviewing serialization conventions, auditing type representations, or implementing field naming rules.

---

## Timestamp Serialization

### RFC 3339 with `_time` Suffix

All timestamp fields use RFC 3339 format (a profile of ISO 8601) and carry the `_time` suffix:

| Field Name | Format | Example |
|------------|--------|---------|
| `created_at_time` | RFC 3339 | `"2024-01-15T09:30:00Z"` |
| `updated_at_time` | RFC 3339 | `"2024-01-16T14:20:00Z"` |
| `expires_at_time` | RFC 3339 | `"2024-12-31T23:59:59Z"` |
| `scheduled_start_time` | RFC 3339 | `"2024-03-01T08:00:00-05:00"` |

**Rules:**
- Always include timezone offset (prefer `Z` for UTC)
- Use `_time` suffix to distinguish from date-only or duration fields
- Never use Unix timestamps (integers) -- they are ambiguous (seconds vs milliseconds) and not human-readable
- Never use custom date formats (`MM/DD/YYYY`, `DD-Mon-YY`)
- Pydantic: Use `datetime` type with `AwareDatetime` annotation

### Date-Only Fields

When only a date is needed (no time component), use ISO 8601 date format with `_date` suffix:

| Field Name | Format | Example |
|------------|--------|---------|
| `birth_date` | ISO 8601 | `"1990-05-20"` |
| `effective_date` | ISO 8601 | `"2024-01-01"` |

---

## Duration Serialization

### ISO 8601 with `_duration` Suffix

Duration fields use ISO 8601 duration format and carry the `_duration` suffix:

| Field Name | Format | Example | Meaning |
|------------|--------|---------|---------|
| `timeout_duration` | ISO 8601 | `"PT30S"` | 30 seconds |
| `retention_duration` | ISO 8601 | `"P90D"` | 90 days |
| `trial_duration` | ISO 8601 | `"P14D"` | 14 days |
| `cooldown_duration` | ISO 8601 | `"PT5M"` | 5 minutes |

**Rules:**
- Use `_duration` suffix to distinguish from timestamps
- Never use bare integers (ambiguous: seconds? minutes? days?)
- Pydantic: Use `timedelta` type with custom serializer to ISO 8601

---

## Money Serialization

### Embedded Object, Never Floats

Monetary values are represented as embedded objects with three fields. Never use floating-point numbers for money.

```json
{
  "price": {
    "currency_code": "USD",
    "units": 19,
    "nanos": 990000000
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `currency_code` | string | ISO 4217 three-letter code |
| `units` | integer | Whole units of the currency |
| `nanos` | integer | Nano-units (10^-9) of the currency |

**Rules:**
- Never use `float` or `Decimal` serialized as JSON number -- IEEE 754 cannot represent `0.1` exactly
- `units` and `nanos` must have the same sign (both positive or both negative)
- `nanos` range: -999,999,999 to 999,999,999
- The actual value is `units + nanos / 1,000,000,000`
- Alternative (simpler): use `amount_cents` as integer (minor units) with `currency_code` if nano precision is unnecessary

### Simplified Minor-Unit Alternative

When nano precision is not needed (most e-commerce):

```json
{
  "price": {
    "currency_code": "USD",
    "amount_minor": 1999
  }
}
```

Where `amount_minor` is the amount in the smallest currency unit (cents for USD, yen for JPY).

---

## Identifier Serialization

### Always Opaque Strings

All identifiers are serialized as JSON strings, even if they are integers internally:

| Correct | Wrong | Why |
|---------|-------|-----|
| `"id": "12345"` | `"id": 12345` | Integer IDs exceed JavaScript's `Number.MAX_SAFE_INTEGER` at scale |
| `"id": "a1b2c3d4-..."` | `"id": "a1b2c3d4-..."` | UUIDs are already strings |

**Rules:**
- All `_id` fields are `string` type in JSON Schema
- Clients must treat IDs as opaque -- never parse, decompose, or assume format
- Pydantic: Use `str` type even for integer primary keys; serialize with `str()` in the response model
- Cross-reference: `governing-route-naming` for identifier design in URI paths

---

## Enum Serialization

### UPPER_SNAKE_CASE Strings

Enums are serialized as `UPPER_SNAKE_CASE` strings, never as numeric codes:

| Correct | Wrong | Why |
|---------|-------|-----|
| `"status": "ACTIVE"` | `"status": 1` | Numeric enums require a lookup table |
| `"role": "BILLING_ADMIN"` | `"role": "billingAdmin"` | Inconsistent with field casing convention |
| `"priority": "HIGH"` | `"priority": 3` | Number ordering is implicit and fragile |

**Rules:**
- Use `UPPER_SNAKE_CASE` for enum values (distinct from `snake_case` field names)
- Define enums as `str, Enum` (StrEnum) in Pydantic for automatic string serialization
- Always include an `UNKNOWN` or `UNSPECIFIED` value for forward compatibility
- Document all valid values in the OpenAPI schema using `enum` constraint
- Never use boolean fields when the domain has more than two states -- use an enum

### Pydantic Implementation

```python
from enum import StrEnum

class ProjectStatus(StrEnum):
    ACTIVE = "ACTIVE"
    ARCHIVED = "ARCHIVED"
    SUSPENDED = "SUSPENDED"
```

---

## Field Naming Conventions

### Default: snake_case

All JSON field names use `snake_case` by default:

```json
{
  "project_name": "My Project",
  "created_at_time": "2024-01-15T09:30:00Z",
  "owner_id": "a1b2c3d4-..."
}
```

**Rules:**
- No prepositions in field names (`start_time`, not `time_of_start`)
- Array fields use plural names (`member_ids`, not `member_id_list`)
- Boolean fields use `is_` or `has_` prefix (`is_active`, `has_members`)
- Abbreviations are lowercase (`url`, `html`, `api`), not `URL`, `HTML`, `API`

### camelCase Escape Hatch

When integrating with ecosystems that require camelCase (JavaScript-heavy frontends, legacy clients), use Pydantic's `alias_generator`:

```python
from pydantic import ConfigDict
from pydantic.alias_generators import to_camel

class ProjectResponse(BaseModel):
    model_config = ConfigDict(
        alias_generator=to_camel,
        populate_by_name=True,
    )

    project_name: str
    created_at_time: datetime
```

This produces `{"projectName": "...", "createdAtTime": "..."}` in responses while keeping Python code in snake_case.

**Rules:**
- `populate_by_name=True` allows both `snake_case` and `camelCase` in input
- The alias is for wire format only -- Python code always uses snake_case
- Document the casing convention in API documentation
- Cross-reference: `governing-schema-implementation` for full alias configuration

---

## Binary Data Serialization

### base64url Encoding

Binary data in JSON payloads uses base64url encoding (RFC 4648 Section 5):

```json
{
  "avatar": "aHR0cHM6Ly9leGFtcGxlLmNvbS9hdmF0YXIucG5n",
  "signature": "MEUCIQC7..."
}
```

**Rules:**
- Use base64url (URL-safe alphabet: `A-Z`, `a-z`, `0-9`, `-`, `_`)
- No padding characters (`=`) unless required by the consumer
- For large binary data (>1MB), use a presigned URL instead of inline encoding
- Document the encoding in the field description

---

## Review Dimensions

When reviewing or auditing field serialization, evaluate these dimensions:

### 1. Timestamp Formats
- Do all timestamp fields use RFC 3339 format?
- Do timestamp field names carry the `_time` suffix?
- Are timezone offsets always included?
- Are there any Unix timestamps or custom date formats?

### 2. Type Representations
- Are monetary values objects (not floats)?
- Are durations ISO 8601 strings (not bare integers)?
- Are identifiers strings (not integers)?
- Are enums `UPPER_SNAKE_CASE` strings (not numbers)?

### 3. Naming Consistency
- Are all field names `snake_case`?
- Do array fields use plural names?
- Are there any prepositions in field names?
- If camelCase is used, is it via `alias_generator` (not manual)?

### 4. Encoding Standards
- Is binary data base64url-encoded?
- Are large binaries served via presigned URLs instead of inline?
- Is the encoding documented in field descriptions?

---

## Anti-Patterns

### 1. Unix Timestamps
Using `1705312200` instead of `"2024-01-15T09:30:00Z"`. Ambiguous (seconds vs milliseconds), not human-readable, and lacks timezone information.

### 2. Floating-Point Money
Using `"price": 19.99`. IEEE 754 cannot represent `0.1` exactly. `0.1 + 0.2 = 0.30000000000000004` in most languages.

### 3. Numeric Enums
Using `"status": 1` instead of `"status": "ACTIVE"`. Requires a lookup table, breaks when values are reordered, and is meaningless in API responses.

### 4. Integer IDs in JSON
Using `"id": 9007199254740993` (exceeds `Number.MAX_SAFE_INTEGER`). JavaScript clients silently lose precision. Always serialize as strings.

### 5. Mixed Field Casing
Using `camelCase` for some fields and `snake_case` for others in the same payload. Use `alias_generator` for consistent casing.

### 6. Bare Integer Durations
Using `"timeout": 30` without units. Is it seconds? Minutes? Milliseconds? Use ISO 8601: `"timeout_duration": "PT30S"`.

---

## Examples

### Example 1: API with Mixed Serialization Formats

**Scenario:** Timestamps use Unix epoch in some fields and ISO strings in others. Money is a float. Enum status is numeric.

**Findings:**
- **Timestamp Formats: FAIL** -- Inconsistent timestamp formats; some use Unix epoch
- **Type Representations: FAIL** -- Money as float, enum as number
- **Recommendation:** Standardize all timestamps to RFC 3339, money to embedded objects, enums to `UPPER_SNAKE_CASE` strings

### Example 2: Well-Serialized API

**Scenario:** All timestamps RFC 3339 with `_time` suffix, money as `{currency_code, units, nanos}`, enums as `UPPER_SNAKE_CASE`, IDs as strings, fields in `snake_case`, binary as base64url.

**Findings:**
- All four dimensions: **PASS**
