---
name: create-bundle
description: "Triggers on: create a bundle, name a bundle, add a new bundle, scaffold a bundle folder"
allowed-tools: Read, Bash, Write, Glob
argument-hint: "[description of what the bundle does]"
---

# Create Bundle Skill

You are creating a new bundle in this repository. Follow these steps exactly.

## Step 0: Parse Input

The user's description is in `$ARGUMENTS`. If empty or unclear, ask the user to describe what the bundle will do before proceeding.

## Step 1: Technological Coupling

Determine whether the bundle is strictly coupled to a single vendor, runtime, or tool.

- **Yes** (e.g., wraps AWS APIs, Terraform configs) → include a technology prefix (single lowercase token: `aws`, `terraform`, `postgres`, `react`, etc.)
- **No** (vendor-agnostic patterns) → omit the technology prefix

## Step 2: Select Domain

Choose the bounded context from the controlled vocabulary in `references/naming-grammar.md`:

`api-route` | `database-schema` | `frontend-component` | `release-pipeline` | `incident-response` | `infrastructure` | `observability` | `identity-access`

If no existing term fits, propose a new one with a clear definition and justification. You must add it to `references/naming-grammar.md` after user approval.

## Step 3: Select Purpose

Choose the core purpose from the controlled vocabulary in `references/naming-grammar.md`:

`governance` | `authoring` | `scaffolding` | `auditing` | `orchestration` | `maintenance` | `resolution`

If no existing term fits, propose a new one with justification and add it to the vocabulary file after approval.

## Step 4: Assemble Name

Combine segments in strict order:

```
[technology]-[domain]-[purpose]
```

Rules:
- Strict kebab-case throughout
- Technology prefix only if Step 1 determined strict coupling
- All segments must be lowercase

### Prohibited Terms

Never use any of these in a bundle name: `utils`, `utilities`, `helpers`, `misc`, `tools`, `manager`, `core`, `common`, `base`

If you find yourself reaching for one of these, revisit Steps 2-3 for more precise vocabulary.

## Step 5: Evaluate Against Rubric

Score the proposed name against the four criteria in `references/evaluation-rubric.md`:

1. **Distinctiveness & Collision Risk** (0-3)
2. **Breadth without Vagueness** (0-3)
3. **Semantic Discoverability** (0-3)
4. **Structural Consistency** (0-3)

The name must score **10 or higher out of 12**. If below 10, revise by re-examining domain/purpose terms and re-score. After two failed attempts, present options to the user with trade-off analysis.

## Step 6: Present for Approval

**Always** present the following to the user and wait for explicit approval before creating any files:

```
Proposed bundle name: [name]
Rubric score: [score]/12
  - Distinctiveness & Collision Risk: [n]/3
  - Breadth without Vagueness: [n]/3
  - Semantic Discoverability: [n]/3
  - Structural Consistency: [n]/3
Rationale: [brief explanation of segment choices]
```

Do NOT proceed until the user approves.

## Step 7: Create Bundle

On approval, create the bundle directory and manifest:

1. Create directory `<name>/` at the repository root
2. Create `<name>/README.md` following the template in `references/bundle-readme-template.md`

The `README.md` must include:
- **Title**: `# {Display Name}` — title-cased human-readable name
- **Description**: one-sentence summary from user's input
- **Domain**: the domain term selected in Step 2
- **Risk Level**: `low`, `medium`, or `high` based on blast radius
- **Technologies**: tech tokens, or "None (vendor-agnostic)"
- **Aliases**: 2-3 shorthand alternatives for discoverability
- **Components**: Skills, Agents, and Tools sections (all "None" initially — populated later)

## Component Naming Conventions

When components are eventually added to a bundle, they must follow these conventions:

| Component | Convention | Separator | Example |
|---|---|---|---|
| Tools | `verb_noun` | underscore | `validate_schema` |
| Skills | `gerund-concept` | kebab-case | `reviewing-migrations` |
| Agents | `role_noun` | underscore | `schema_reviewer` |

## References

- [Naming Grammar & Vocabularies](references/naming-grammar.md)
- [Evaluation Rubric](references/evaluation-rubric.md)
- [Bundle README Template](references/bundle-readme-template.md)
- [Worked Example](examples/bundle-example.md)
