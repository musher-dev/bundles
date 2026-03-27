# Repo Documentation Authoring

Scaffolds, authors, and audits the essential repository documentation files recommended by GitHub for discoverability, community health, and contributor onboarding.

## Metadata

- **Domain:** repo-documentation
- **Risk Level:** low
- **Technologies:** None (vendor-agnostic)
- **Aliases:** repo-docs, readme-authoring, community-health

## Components

### Skills
- `scaffolding-repo-documentation` — Scaffold starter files for all community health documents
- `writing-readme` — Author README.md with structured sections adapted by project type
- `writing-contributing-guide` — Author CONTRIBUTING.md with dev setup, workflow, and PR process
- `writing-security-policy` — Author SECURITY.md with disclosure process and response SLAs
- `writing-community-policies` — Author CODE_OF_CONDUCT, SUPPORT, GOVERNANCE, and FUNDING.yml
- `writing-github-config` — Author issue templates, PR template, CODEOWNERS, and CHANGELOG
- `auditing-repo-documentation` — Score documentation health across 5 dimensions with remediation roadmap

### Agents
- `repo_docs_author` — Orchestrates the full documentation authoring workflow from scaffolding through writing
- `repo_docs_auditor` — Read-only auditor that scores documentation health and produces prioritized findings

### Tools
- None
