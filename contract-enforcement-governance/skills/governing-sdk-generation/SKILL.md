---
name: governing-sdk-generation
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of SDK generation governance including @hey-api/openapi-ts configuration, prohibiting manual fetch wrappers, tsc --noEmit conformance checks, generated SDK consumption rules, regeneration triggers, and SDK package publishing strategy. Use when configuring SDK generation, auditing manual API wrappers, reviewing generated client usage, or setting up SDK regeneration pipelines. Triggered by: SDK generation, openapi-ts, hey-api, generated client, fetch wrapper, API client, type generation, SDK config, client generation, codegen, generated types, SDK governance.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# SDK Generation Governance

Comprehensive governance for generating, consuming, and maintaining typed API client SDKs from OpenAPI specifications.

## Purpose

Hand-written API clients are a maintenance liability. Every endpoint added to the spec requires a manual client update. Every field renamed requires a search-and-replace across fetch calls. Every type change requires manual type updates that may not match the spec. This skill enforces SDK generation from the OpenAPI spec, prohibits manual fetch wrappers, and establishes consumption rules that keep the frontend in lock-step with the contract.

**Boundary:** This skill governs the SDK generation pipeline and consumption patterns. For the spec that feeds generation, see `openapi-specification-governance`. For the CI pipeline that triggers regeneration, see `governing-enforcement-pipeline`.

Use this skill when configuring SDK generation, auditing for manual fetch wrappers, or establishing SDK consumption patterns.

---

## @hey-api/openapi-ts Configuration

### Installation

```bash
npm install @hey-api/openapi-ts --save-dev
```

### Configuration File

```typescript
// openapi-ts.config.ts
import { defineConfig } from "@hey-api/openapi-ts";

export default defineConfig({
  client: "@hey-api/client-fetch",
  input: "generated/openapi.bundled.json",
  output: {
    path: "generated/sdk",
    format: "prettier",
    lint: "eslint",
  },
  schemas: false,
  services: {
    asClass: true,
  },
  types: {
    enums: "javascript",
    dates: "types+transform",
  },
});
```

### Configuration Rules

| Setting | Recommended Value | Rationale |
|---------|------------------|-----------|
| `input` | Bundled spec path | Single-file input ensures all refs are resolved |
| `output.path` | `generated/sdk` or equivalent | Clear separation from hand-written code |
| `output.format` | `prettier` | Consistent formatting in generated code |
| `services.asClass` | `true` | Produces organized, importable service classes |
| `types.enums` | `javascript` | Runtime-accessible enum values, not just types |
| `types.dates` | `types+transform` | Date strings become Date objects with type safety |
| `schemas` | `false` | Runtime validation schemas add bundle size without value when server validates |

---

## Prohibiting Manual Fetch Wrappers

### What Constitutes a Manual Wrapper

Any hand-written code that directly calls `fetch`, `axios`, or similar HTTP clients to communicate with an API that has an OpenAPI spec and generated SDK:

```typescript
// Anti-pattern: manual fetch wrapper
async function getProjects(): Promise<Project[]> {
  const response = await fetch("/api/v1/projects");
  return response.json();
}

// Anti-pattern: manual axios wrapper
const api = axios.create({ baseURL: "/api/v1" });
export const getProjects = () => api.get<Project[]>("/projects");
```

### Why Manual Wrappers Are Prohibited

| Problem | Impact |
|---------|--------|
| Types drift from spec | Frontend types become stale when spec changes |
| Missing parameters | New required parameters are not added |
| Wrong response shape | Response type assertions may not match reality |
| No breaking change detection | Manual wrappers bypass the contract pipeline |
| Duplication of effort | SDK generation already produces the same code |

### Detection Patterns

Search for manual wrappers using these patterns:

```
# Grep patterns to detect manual API wrappers
fetch\(["\']\/api\/
axios\.(get|post|put|patch|delete)\(
new XMLHttpRequest
\.request\(\{.*url:.*\/api\/
```

### Replacement Process

1. Identify the manual wrapper and its callers
2. Find the equivalent generated SDK function
3. Replace the manual call with the SDK import
4. Remove the manual wrapper
5. Verify types align (no `as` assertions needed)

---

## `tsc --noEmit` Conformance

### Type Checking Generated SDK Consumers

After SDK regeneration, run `tsc --noEmit` on the consuming codebase to verify type compatibility:

```bash
# In the frontend package
tsc --noEmit
```

### What `tsc --noEmit` Catches

| Issue | Example | Detection |
|-------|---------|-----------|
| Removed field access | `project.legacy_field` after field removed from spec | Property does not exist on type |
| Changed field type | `project.id` changed from string to number | Type 'number' is not assignable to type 'string' |
| New required parameter | `createProject()` now requires `region` | Expected 2 arguments, got 1 |
| Renamed enum value | `Status.ACTIVE` renamed to `Status.ENABLED` | Property 'ACTIVE' does not exist on type |

### CI Integration

```yaml
# In the frontend build job
- name: Regenerate SDK
  run: npx openapi-ts

- name: Type check
  run: npx tsc --noEmit

- name: Build
  run: npm run build
```

### Type Conformance Rules

| Rule | Rationale |
|------|-----------|
| `tsc --noEmit` must pass after every SDK regeneration | Catches contract-breaking type changes |
| No `@ts-ignore` or `@ts-expect-error` on SDK imports | Suppressing errors hides contract drift |
| No `as any` or `as unknown` on SDK return types | Type assertions bypass the contract |
| `strict: true` in tsconfig | Enables full type checking coverage |

---

## SDK Consumption Rules

### Import Patterns

```typescript
// Correct: import from generated SDK
import { ProjectsService, ProjectResponse } from "@workspace/contract/sdk";

// Anti-pattern: import from manual wrapper
import { getProjects } from "../api/projects";

// Anti-pattern: re-export with modifications
import { ProjectResponse as _ProjectResponse } from "@workspace/contract/sdk";
export interface ProjectResponse extends _ProjectResponse {
  computedField: string; // Adding fields to generated types
}
```

### Consumption Rules

| Rule | Rationale |
|------|-----------|
| Import types and services directly from the generated SDK | Single source of truth for API types |
| Never extend or modify generated types | Extensions hide contract drift |
| Use SDK service methods, not raw HTTP calls | Services include correct URL, method, and serialization |
| Handle errors using SDK error types | Generated error types match the spec |
| Configure the SDK client once at app initialization | Consistent base URL, auth headers, interceptors |

### Client Configuration

```typescript
// app initialization
import { client } from "@workspace/contract/sdk";

client.setConfig({
  baseUrl: import.meta.env.VITE_API_BASE_URL,
  headers: {
    Authorization: `Bearer ${getToken()}`,
  },
});
```

---

## Regeneration Triggers

### When to Regenerate

| Trigger | Mechanism | Rationale |
|---------|-----------|-----------|
| Spec file changed in PR | CI pipeline | Keep SDK in sync with spec |
| Dependency update (openapi-ts) | Dependabot/Renovate PR | Generator version may change output |
| Manual request | Developer runs script | Ad-hoc sync during development |
| Post-merge to main | CI pipeline | Ensure main branch SDK is current |

### Regeneration Pipeline

```
1. Bundle spec      →  openapi.bundled.json
2. Generate SDK     →  generated/sdk/
3. Type check       →  tsc --noEmit
4. Run tests        →  vitest / jest
5. Commit (if CI)   →  generated files committed
```

### Regeneration Rules

| Rule | Rationale |
|------|-----------|
| Regeneration must be deterministic | Same spec input produces same SDK output |
| Generated files should be committed | Ensures consumers do not need the generator installed |
| Regeneration must trigger type checking | Catches breaking changes immediately |
| Generator version must be pinned | Prevents surprise output changes |

---

## Review Dimensions

When reviewing or auditing SDK generation governance, evaluate these dimensions:

### 1. Generation Configuration
- Is a code generator (openapi-ts or equivalent) configured?
- Is the input the bundled spec (not raw multi-file)?
- Are output settings appropriate (format, lint, types)?

### 2. Manual Wrapper Elimination
- Are there manual fetch/axios wrappers for spec-covered endpoints?
- Have all manual wrappers been replaced with SDK imports?
- Are there grep patterns configured to prevent regression?

### 3. Type Conformance
- Does `tsc --noEmit` pass after regeneration?
- Are there `@ts-ignore` or `as any` suppressions on SDK types?
- Is `strict: true` enabled in tsconfig?

### 4. Consumption Hygiene
- Do consumers import directly from the generated SDK?
- Are there modified or extended generated types?
- Is the SDK client configured at app initialization?

---

## Anti-Patterns

### 1. Manual Fetch Alongside Generated SDK
Having both a generated SDK and hand-written fetch calls for the same API. Consumers use whichever they find first, leading to inconsistent behavior.

### 2. Type Assertion on SDK Types
Using `as any`, `as unknown`, or custom type extensions on generated types. Hides contract drift until runtime.

### 3. Unpinned Generator Version
Using `latest` or `^` for the generator package version. Generator updates may change output format, breaking consumers.

### 4. Generated-But-Not-Committed
Generating SDK in CI but not committing the output. Developers working locally use stale generated code until they remember to regenerate.

### 5. Spec-Specific Generator Config
Hardcoding spec-specific paths or options that break when the spec structure changes. Config should reference stable paths from the contract package.

---

## Guardrails

1. **All API communication must use the generated SDK** -- no manual fetch/axios wrappers for spec-covered endpoints
2. **`tsc --noEmit` must pass after every regeneration** -- type errors indicate contract drift
3. **No type assertions on generated SDK types** -- `as any`, `@ts-ignore`, and type extensions are prohibited
4. **Generator version must be pinned** -- prevent surprise output changes
5. **Generated files must be committed to version control** -- ensures all consumers use the same generated code
6. **SDK regeneration must be triggered by spec changes in CI** -- keeps SDK in sync with the contract
