---
name: testing-contract-conformance
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of contract conformance testing using Schemathesis integration with from_asgi() for FastAPI applications, stateful testing with OpenAPI link-based state machines, test database management for conformance test suites, response schema validation, and coverage measurement against the OpenAPI specification. Use when configuring Schemathesis, auditing contract test coverage, reviewing stateful test strategies, or setting up conformance test infrastructure. Triggered by: contract testing, Schemathesis, conformance testing, schema validation, stateful testing, OpenAPI links, from_asgi, property-based testing, API testing, contract test, spec conformance, response validation.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Contract Conformance Testing

Comprehensive governance for verifying that API implementations conform to their OpenAPI specifications using automated, property-based testing.

## Purpose

A spec and an implementation are two separate artifacts that can silently diverge. The spec says a field is required; the implementation omits it. The spec defines a 201 response; the implementation returns 200. Manual test suites cannot cover the combinatorial space of valid and invalid inputs. This skill enforces automated conformance testing using Schemathesis, which generates test cases directly from the OpenAPI spec and validates that the implementation matches the contract.

**Boundary:** This skill governs conformance testing between spec and implementation. For the spec itself, see `openapi-specification-governance`. For the CI pipeline that orchestrates testing alongside other stages, see `governing-enforcement-pipeline`.

Use this skill when setting up contract conformance testing, auditing test coverage against the spec, or configuring stateful API testing.

---

## Schemathesis Integration

### Core Concept

Schemathesis reads an OpenAPI specification and automatically generates test cases that exercise every documented endpoint, parameter combination, and edge case. It then validates that responses conform to the documented schemas.

### Installation

```bash
# Python package
pip install schemathesis

# Or via Docker
docker pull schemathesis/schemathesis
```

### Basic Usage

```bash
# Test against a running server
st run https://api.example.com/openapi.json

# Test with base URL override
st run openapi.json --base-url http://localhost:8000

# Test specific endpoint
st run openapi.json --endpoint /projects
```

---

## FastAPI Integration with `from_asgi()`

### Direct ASGI Testing (No Server Required)

```python
import schemathesis
from myapp.main import app

schema = schemathesis.from_asgi("/openapi.json", app=app)

@schema.parametrize()
def test_api_conformance(case):
    response = case.call_asgi()
    case.validate_response(response)
```

### Why `from_asgi()` Over Network Testing

| Aspect | `from_asgi()` | Network Testing |
|--------|---------------|-----------------|
| Speed | Fast (in-process) | Slow (HTTP overhead) |
| Server required | No | Yes |
| Database control | Direct access | Requires fixtures |
| Debugging | Standard debugger | Remote debugging |
| CI simplicity | Single process | Requires server lifecycle |

### Configuration with pytest

```python
# conftest.py
import schemathesis
from myapp.main import app

schema = schemathesis.from_asgi("/openapi.json", app=app)

# tests/test_conformance.py
@schema.parametrize()
def test_api(case):
    response = case.call_asgi()
    case.validate_response(response)
```

### Filtering Operations

```python
# Test only specific operations
@schema.parametrize(endpoint="/projects")
def test_projects(case):
    response = case.call_asgi()
    case.validate_response(response)

# Exclude operations
@schema.parametrize(endpoint="/projects", method="GET")
def test_list_projects(case):
    response = case.call_asgi()
    case.validate_response(response)
```

---

## Stateful Testing with OpenAPI Links

### Link-Based State Machines

OpenAPI links define relationships between operations. Schemathesis uses these links to generate stateful test sequences:

```yaml
# In the OpenAPI spec
paths:
  /projects:
    post:
      operationId: createProject
      responses:
        "201":
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ProjectResponse"
          links:
            GetProject:
              operationId: getProject
              parameters:
                project_id: "$response.body#/id"
            UpdateProject:
              operationId: updateProject
              parameters:
                project_id: "$response.body#/id"
  /projects/{project_id}:
    get:
      operationId: getProject
    patch:
      operationId: updateProject
    delete:
      operationId: deleteProject
```

### Enabling Stateful Tests

```bash
# CLI
st run openapi.json --stateful=links

# Python
@schema.parametrize(stateful=schemathesis.Stateful.links)
def test_api_stateful(case):
    response = case.call_asgi()
    case.validate_response(response)
```

### State Machine Behavior

With stateful testing enabled, Schemathesis generates sequences like:

```
1. POST /projects → 201 (create)
2. GET /projects/{id} → 200 (read, using id from step 1)
3. PATCH /projects/{id} → 200 (update, using id from step 1)
4. DELETE /projects/{id} → 204 (delete, using id from step 1)
```

### Stateful Testing Rules

| Rule | Rationale |
|------|-----------|
| Define OpenAPI links for all CRUD relationships | Enables automated state machine generation |
| Link `parameters` must use `$response.body#/` runtime expressions | Connects operation outputs to inputs |
| Link `operationId` must reference existing operations | Broken links silently disable stateful sequences |
| Test sequences should cover create → read → update → delete | Full lifecycle coverage |

---

## Test Database Management

### Isolation Requirements

Conformance tests generate random data and create resources. The test database must be:

| Requirement | Implementation |
|-------------|----------------|
| Isolated from development/production | Dedicated test database or schema |
| Reset between test runs | Truncate tables or recreate schema |
| Seeded with required reference data | Fixtures for foreign key targets |
| Fast to provision | In-memory or containerized |

### Database Strategy for FastAPI

```python
# conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from myapp.database import Base, get_db
from myapp.main import app

TEST_DATABASE_URL = "postgresql://test:test@localhost:5432/test_conformance"

@pytest.fixture(scope="session")
def test_engine():
    engine = create_engine(TEST_DATABASE_URL)
    Base.metadata.create_all(engine)
    yield engine
    Base.metadata.drop_all(engine)

@pytest.fixture(autouse=True)
def test_session(test_engine):
    connection = test_engine.connect()
    transaction = connection.begin()
    session = sessionmaker(bind=connection)()

    def override_get_db():
        yield session

    app.dependency_overrides[get_db] = override_get_db
    yield session
    transaction.rollback()
    connection.close()
```

### Database Rules

| Rule | Rationale |
|------|-----------|
| Use transaction rollback for test isolation | Faster than truncation, guarantees clean state |
| Seed reference data in session-scoped fixtures | Foreign key targets must exist before test cases run |
| Never run conformance tests against production databases | Generated data includes random/edge-case values |
| Use the same database engine as production | SQLite-for-tests hides PostgreSQL-specific behavior |

---

## Response Validation

### What Schemathesis Validates

| Aspect | Validated? | Details |
|--------|-----------|---------|
| Response status codes | Yes | Only documented status codes are acceptable |
| Response body schema | Yes | JSON structure matches the declared schema |
| Required properties | Yes | All required fields must be present |
| Property types | Yes | Field types match schema declarations |
| Content-Type header | Yes | Response media type matches spec |
| Response headers | Partial | Validates declared headers only |

### Custom Validation Hooks

```python
@schema.parametrize()
def test_api(case):
    response = case.call_asgi()
    case.validate_response(response)

    # Custom assertions beyond schema validation
    if response.status_code == 201:
        assert "Location" in response.headers
    if response.status_code == 200 and "items" in response.json():
        assert isinstance(response.json()["items"], list)
```

### Validation Rules

| Rule | Rationale |
|------|-----------|
| All responses must validate against spec schemas | Core conformance requirement |
| Undocumented status codes are failures | Spec must document all possible responses |
| Empty response bodies must match spec declaration | If spec says no body, implementation must not return one |
| Additional properties in responses are acceptable by default | Forward compatibility; spec does not need to enumerate internal fields |

---

## Coverage Measurement

### Measuring Spec Coverage

Track what percentage of the specification is exercised by conformance tests:

| Metric | Target | Measurement |
|--------|--------|-------------|
| Endpoint coverage | 100% | Every path/method combination tested |
| Response code coverage | All documented | Every documented status code triggered |
| Schema coverage | 100% | Every schema property validated |
| Parameter coverage | All required + optional | All parameters exercised at least once |

### Schemathesis Coverage Report

```bash
# Generate coverage report
st run openapi.json --show-trace --dry-run

# Check which operations are tested
st run openapi.json --show-trace 2>&1 | grep -E "^(GET|POST|PUT|PATCH|DELETE)"
```

---

## Review Dimensions

When reviewing or auditing conformance testing, evaluate these dimensions:

### 1. Test Integration
- Is Schemathesis (or equivalent) configured and running in CI?
- Is `from_asgi()` used for FastAPI projects?
- Are conformance tests separate from unit tests?

### 2. Stateful Coverage
- Are OpenAPI links defined for CRUD relationships?
- Is stateful testing enabled?
- Do test sequences cover full resource lifecycles?

### 3. Database Management
- Is the test database isolated from development/production?
- Are tests using transaction rollback for isolation?
- Is the database engine the same as production?

### 4. Validation Completeness
- Are all response schemas validated?
- Are undocumented status codes treated as failures?
- Are custom validation hooks used for business rules?

---

## Anti-Patterns

### 1. Spec-Last Testing
Writing conformance tests against the implementation and then updating the spec to match. Reverses the contract-first flow.

### 2. SQLite-for-Tests
Using SQLite instead of PostgreSQL for conformance tests. Hides database-specific behavior (constraints, types, JSON operators).

### 3. No Stateful Sequences
Testing each endpoint in isolation without state machine sequences. Misses bugs that only appear in multi-step workflows (create then update, create then delete).

### 4. Suppressing Schema Failures
Configuring tests to ignore response schema mismatches. Defeats the purpose of conformance testing.

### 5. Shared Test Database
Running conformance tests against a database shared with manual testing or other CI jobs. Random test data corrupts shared state.

---

## Guardrails

1. **Conformance tests must run against the published OpenAPI spec** -- not a test-specific copy
2. **`from_asgi()` is the default integration method for FastAPI** -- avoids server lifecycle complexity
3. **Stateful testing must be enabled when OpenAPI links are defined** -- exercises real operation sequences
4. **Test database must use the same engine as production** -- no SQLite substitution
5. **Transaction rollback for test isolation** -- faster and more reliable than truncation
6. **Every documented response code should be triggerable** -- dead documentation is misleading documentation
