# Worked Example: "Audit AWS IAM policies for compliance"

## Input

```
/create-bundle audits AWS IAM policies for compliance
```

## Step 1: Technological Coupling

The description explicitly references **AWS IAM** — a vendor-specific service. The bundle's logic will depend on AWS APIs and IAM-specific concepts.

**Decision:** Include `aws` technology prefix.

## Step 2: Bounded Context (Domain)

IAM policies deal with authentication, authorization, roles, and permissions.

**Domain:** `identity-access`

## Step 3: Core Purpose

The primary action is auditing — inspecting existing policies against a compliance baseline and reporting drift.

**Purpose:** `auditing`

## Step 4: Assemble Name

```
aws-identity-access-auditing
```

- Format: `[technology]-[domain]-[purpose]`
- All segments: strict kebab-case
- No prohibited terms

## Step 5: Evaluate Against Rubric

| Criterion | Score | Rationale |
|---|---|---|
| Distinctiveness & Collision Risk | 3 | Unique combination; `aws` prefix + `identity-access` + `auditing` is specific |
| Breadth without Vagueness | 3 | Covers IAM policy auditing without overreaching into provisioning or authoring |
| Semantic Discoverability | 2 | "identity-access" requires slight domain familiarity; most devs would still find it |
| Structural Consistency | 3 | Uses established vocabulary in correct grammar order |
| **Total** | **11/12** | **Pass** (threshold: 10) |

## Proposed Name

```
aws-identity-access-auditing
```

*Presented to user for approval before proceeding.*

## Resulting Output

```
aws-identity-access-auditing/
└── README.md
```

### README.md

```markdown
# AWS Identity Access Auditing

Audits AWS IAM policies for compliance against organizational standards and security baselines.

## Metadata

- **Domain:** identity-access
- **Risk Level:** medium
- **Technologies:** aws
- **Aliases:** iam-audit, aws-iam-compliance

## Components

### Skills
- None

### Agents
- None

### Tools
- None
```
