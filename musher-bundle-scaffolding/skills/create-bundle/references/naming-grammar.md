# Naming Grammar Reference

## Grammar Rule

```
[technology]-[domain]-[purpose]
```

All segments use **strict kebab-case**. The technology prefix is optional.

## Technology Prefix Decision Guide

Include the technology prefix **only** when the bundle is strictly coupled to a single vendor or runtime:

| Condition | Include Tech Prefix? | Example |
|---|---|---|
| Logic only works with one vendor SDK/API | Yes | `aws-identity-access-auditing` |
| Wraps a specific tool's CLI or config format | Yes | `terraform-infrastructure-scaffolding` |
| Vendor-agnostic patterns or practices | No | `database-schema-governance` |
| Multi-cloud / polyglot by design | No | `release-pipeline-orchestration` |

When included, the technology prefix is a single lowercase token (e.g., `aws`, `terraform`, `postgres`, `react`).

## Domain Vocabulary

| Term | Definition |
|---|---|
| `api-route` | HTTP API endpoint design, routing, and contract management |
| `database-schema` | Database structure, migrations, and data modeling |
| `frontend-component` | UI component architecture, patterns, and rendering |
| `release-pipeline` | CI/CD workflows, deployment stages, and release gating |
| `incident-response` | Alert handling, runbooks, and post-incident processes |
| `infrastructure` | Cloud resources, IaC templates, and environment provisioning |
| `observability` | Logging, metrics, tracing, and monitoring configuration |
| `identity-access` | Authentication, authorization, roles, and permissions |
| `repo-documentation` | Essential repository documentation files recommended by GitHub for discoverability, community health, and contributor onboarding |
| `agent-asset` | Agent file design and structure — skills, hooks, rules, and related assets |
| `bundle` | Bundle directory structure, manifests, and packaging of reusable components |
| `developer-docs` | Developer-facing technical documentation including API references, user guides, tutorials, and usage documentation |
| `developer-environment` | Local development workspace configuration, containerized environments, task runners, and toolchain setup |
| `marketing-site` | Marketing website creation, content production, design system enforcement, and conversion-focused page authoring |
| `product-profile` | Canonical product documentation encompassing machine-readable metadata, strategic positioning, roadmaps, and technical specifications for a product or product line |
| `project-shaping` | Upstream product development: problem framing, solution design, risk assessment, and scope definition prior to build commitment |

## Purpose Vocabulary

| Term | Definition |
|---|---|
| `governance` | Policy enforcement, compliance checks, and standards adherence |
| `authoring` | Content or code creation workflows and templates |
| `scaffolding` | Project or component bootstrapping and initial structure generation |
| `auditing` | Inspection, reporting, and drift detection against a known baseline |
| `orchestration` | Multi-step workflow coordination and sequencing |
| `maintenance` | Upkeep tasks: upgrades, deprecation handling, and cleanup |
| `resolution` | Troubleshooting, root-cause analysis, and remediation |

## Extensibility Process

If no existing term fits, the skill may propose a new term under these conditions:

1. Document why no current term is adequate
2. Provide a concise definition following the style above
3. Add the new term to the appropriate vocabulary table in this file
4. Note the addition in the bundle's `README.md` description for traceability
