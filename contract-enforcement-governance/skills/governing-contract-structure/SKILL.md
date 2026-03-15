---
name: governing-contract-structure
version: 1.0.0
user-invocable: false
description: Guide design, review, audit, and implementation of monorepo contract structure including workspace configuration for Turborepo and Nx, packages/contract/ directory layout, artifact classification (editable spec, generated outputs, implementation source), cross-package dependency rules, and build orchestration for contract-dependent packages. Use when designing monorepo contract layout, auditing artifact separation, reviewing workspace config, or structuring contract packages. Triggered by: monorepo contract, packages/contract, artifact separation, workspace config, Turborepo, Nx, contract layout, editable spec, generated artifacts, contract directory, contract package, repo structure.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Contract Structure Governance

Comprehensive governance for monorepo contract artifact organization -- workspace configuration, directory layout, artifact classification, and build orchestration.

## Purpose

Contract artifacts span three distinct lifecycle categories: editable specifications authored by humans, generated outputs produced by tooling, and implementation source that consumes the contract. Mixing these categories in a single directory creates confusion about what can be edited, what will be overwritten, and what depends on what. This skill enforces clean separation of contract artifacts across the repository, ensuring every file has a clear owner and lifecycle.

**Boundary:** This skill governs where contract artifacts live in the repository and how they relate to each other. For how the spec itself is split across files within the spec directory, see `openapi-specification-governance/governing-spec-structure`.

Use this skill when designing monorepo layout for contract artifacts, auditing artifact separation, or configuring workspace dependencies.

---

## Artifact Classification

### Three Artifact Categories

Every file in the contract pipeline belongs to exactly one category:

| Category | Owner | Edited By | Lifecycle | Example Files |
|----------|-------|-----------|-----------|---------------|
| **Editable** | Humans | Developers via PR | Changed intentionally | `openapi/openapi.yaml`, `openapi/paths/*.yaml`, `openapi/components/**/*.yaml` |
| **Generated** | Tooling | CLI commands, CI pipelines | Overwritten on regeneration | `sdk/`, `openapi.bundled.json`, client types |
| **Implementation** | Humans | Developers via PR | Imports from generated | `backend/src/`, `frontend/src/` |

### Classification Rules

| Rule | Rationale |
|------|-----------|
| Never hand-edit generated files | Edits are overwritten on next generation cycle |
| Generated files should be gitignored or committed as read-only | Prevents accidental manual edits |
| Implementation code imports from generated packages, never from editable spec | Ensures implementation consumes the same artifact as CI validation |
| Editable spec is the sole input to the generation pipeline | Single source of truth |

---

## Directory Layout

### Recommended `packages/contract/` Structure

```
packages/contract/
  openapi/                    # Editable: spec source files
    openapi.yaml              # Root spec file (entry point)
    paths/                    # Path items split by resource
      projects.yaml
      organizations.yaml
    components/
      schemas/                # Domain schemas
        project.yaml
        organization.yaml
      parameters/             # Shared parameters
        pagination.yaml
      responses/              # Shared error responses
        errors.yaml
  generated/                  # Generated: tooling output
    openapi.bundled.json      # Bundled single-file spec
    sdk/                      # Generated SDK client
      index.ts
      types.ts
      services/
  redocly.yaml                # Bundling config
  package.json                # Package metadata and scripts
  tsconfig.json               # TypeScript config for generated SDK
```

### Layout Rules

| Rule | Rationale |
|------|-----------|
| `openapi/` contains only editable spec files | Clear boundary for human-authored content |
| `generated/` contains only tooling output | Everything inside is overwritten by scripts |
| Root config files (`redocly.yaml`, `package.json`) live at package root | Standard package conventions |
| No implementation code inside `packages/contract/` | Contract package is a dependency, not an application |

---

## Workspace Configuration

### Turborepo

```json
// turbo.json
{
  "tasks": {
    "contract#bundle": {
      "inputs": ["packages/contract/openapi/**"],
      "outputs": ["packages/contract/generated/openapi.bundled.json"]
    },
    "contract#generate-sdk": {
      "dependsOn": ["contract#bundle"],
      "inputs": ["packages/contract/generated/openapi.bundled.json"],
      "outputs": ["packages/contract/generated/sdk/**"]
    },
    "backend#build": {
      "dependsOn": ["contract#bundle"]
    },
    "frontend#build": {
      "dependsOn": ["contract#generate-sdk"]
    }
  }
}
```

### Nx

```json
// packages/contract/project.json
{
  "targets": {
    "bundle": {
      "executor": "nx:run-commands",
      "options": {
        "command": "redocly bundle openapi/openapi.yaml -o generated/openapi.bundled.json"
      },
      "inputs": ["{projectRoot}/openapi/**"],
      "outputs": ["{projectRoot}/generated/openapi.bundled.json"]
    },
    "generate-sdk": {
      "executor": "nx:run-commands",
      "dependsOn": ["bundle"],
      "options": {
        "command": "openapi-ts"
      },
      "inputs": ["{projectRoot}/generated/openapi.bundled.json"],
      "outputs": ["{projectRoot}/generated/sdk/**"]
    }
  }
}
```

### Workspace Rules

| Rule | Rationale |
|------|-----------|
| `bundle` task depends only on editable spec inputs | Caching invalidates when spec changes |
| `generate-sdk` depends on `bundle` | SDK generation consumes the bundled output |
| Backend build depends on `bundle` | Backend validates against bundled spec |
| Frontend build depends on `generate-sdk` | Frontend imports generated SDK types |
| Task outputs are declared explicitly | Enables correct caching and invalidation |

---

## Cross-Package Dependencies

### Dependency Direction

```
packages/contract/          (upstream -- no dependencies on app packages)
    ↓ bundle
packages/contract/generated (intermediate artifact)
    ↓                  ↓
backend/               frontend/
(validates against     (imports SDK from
 bundled spec)          generated/)
```

### Dependency Rules

| Rule | Rationale |
|------|-----------|
| Contract package has zero dependencies on application packages | Contract is infrastructure, not application code |
| Application packages depend on contract, never the reverse | Unidirectional dependency flow |
| Frontend imports only from `generated/sdk/`, never from `openapi/` | Frontend consumes typed SDK, not raw spec |
| Backend references `generated/openapi.bundled.json` for validation | Single bundled file for runtime schema validation |

---

## Package Scripts

### Recommended `package.json` Scripts

```json
{
  "name": "@workspace/contract",
  "scripts": {
    "lint": "redocly lint openapi/openapi.yaml",
    "bundle": "redocly bundle openapi/openapi.yaml -o generated/openapi.bundled.json",
    "generate-sdk": "openapi-ts",
    "diff": "oasdiff breaking generated/openapi.bundled.json <previous-version>",
    "validate": "npm run lint && npm run bundle && npm run diff",
    "clean": "rm -rf generated/"
  }
}
```

### Script Rules

| Rule | Rationale |
|------|-----------|
| `lint` runs before `bundle` | Catches spec errors before generating artifacts |
| `bundle` produces a single file from multi-file spec | Consumers need one file, authors need many |
| `generate-sdk` runs after `bundle` | SDK generation requires bundled input |
| `diff` compares against previous version | Breaking change detection before merge |
| `clean` removes only `generated/` | Never deletes editable spec files |

---

## Review Dimensions

When reviewing or auditing contract structure, evaluate these dimensions:

### 1. Artifact Separation
- Are editable, generated, and implementation files in distinct directories?
- Are generated files clearly marked (gitignore, README, naming convention)?
- Is there any hand-edited code inside a generated directory?

### 2. Workspace Configuration
- Are build tasks declared with correct dependency ordering?
- Are inputs and outputs declared for caching?
- Does the frontend build depend on SDK generation?

### 3. Dependency Direction
- Does the contract package have zero application dependencies?
- Do application packages import from correct contract subdirectories?
- Is the dependency graph acyclic?

### 4. Script Completeness
- Does the contract package have lint, bundle, generate, diff, and clean scripts?
- Are scripts ordered to match the pipeline stages?
- Is there a single `validate` script that runs the full pipeline?

---

## Anti-Patterns

### 1. Mixed Artifact Directory
Editable spec files and generated SDK output in the same directory. Developers cannot tell which files are safe to edit.

### 2. Implementation Inside Contract Package
Backend route handlers or frontend components placed inside `packages/contract/`. The contract package should contain only spec and generated artifacts.

### 3. Direct Spec Import
Frontend code importing from `openapi/openapi.yaml` instead of the generated SDK. Bypasses type generation and validation.

### 4. Missing Build Dependencies
Frontend build task that does not depend on `generate-sdk`. Stale SDK types cause silent type mismatches.

### 5. Undeclared Task Outputs
Workspace tasks without explicit output declarations. Build caching cannot function correctly without output manifests.

### 6. Bidirectional Dependencies
Contract package importing from application packages. Creates circular dependency and couples contract to implementation.

---

## Guardrails

1. **Editable and generated artifacts must be in separate directories** -- prevents accidental edits to generated files
2. **Contract package must have zero application dependencies** -- contract is infrastructure, not application code
3. **Build tasks must declare dependency ordering** -- bundle before generate, generate before application build
4. **Frontend must import from generated SDK, never from raw spec** -- ensures type safety through the generation pipeline
5. **Every workspace task must declare inputs and outputs** -- enables correct caching and invalidation
6. **Generated files must be reproducible from editable spec alone** -- `clean && validate` must succeed from a fresh state
