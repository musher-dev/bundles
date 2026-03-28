# Repo Documentation Authoring

Scaffold and maintain the community health files that GitHub recommends — README, CONTRIBUTING, SECURITY, CODE_OF_CONDUCT, issue templates, PR templates, CODEOWNERS, and more. This bundle audits what you have, scores documentation health across five dimensions, and generates what's missing.

## Quick Start

Install via the [Mush CLI](https://github.com/musher-dev/mush):

```sh
mush bundle install musher-dev/repo-documentation-authoring
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
Audit our repo documentation and tell me what community health files are missing or incomplete
```

## What's Inside

The bundle ships two agents and seven skills covering the full repository documentation lifecycle.

The **repo_docs_auditor** agent runs a read-only assessment of your documentation health across five dimensions: Presence (do the files exist?), Completeness (are sections filled in?), Currency (are they up to date?), Discoverability (can contributors find them?), and Quality (do they meet standards?). It produces a severity-prioritized findings report with a remediation roadmap.

The **repo_docs_author** agent orchestrates the full authoring workflow — scaffolding starter files, then writing each document with content adapted to your project type (library, CLI tool, web app, monorepo, etc.).

Individual writing skills handle each document type: README with structured sections, CONTRIBUTING with dev setup and PR process, SECURITY with disclosure process and response SLAs, CODE_OF_CONDUCT based on Contributor Covenant 2.1, SUPPORT with channel routing, GOVERNANCE with decision-making models, FUNDING.yml for sponsorship, issue templates (bug report, feature request), PR templates, CODEOWNERS for automatic reviewer assignment, and CHANGELOG following Keep a Changelog 1.1.0.

## Usage Examples

**Audit documentation health**
```
Score our repository's documentation health — check presence, completeness, currency, and quality of all community health files
```
The repo_docs_auditor produces a 5-dimension score with prioritized remediation steps.

**Scaffold all community health files**
```
Scaffold starter files for all the missing community health documents in this repo
```
Creates template files for every recommended document that doesn't exist yet.

**Write a CONTRIBUTING guide**
```
Write a CONTRIBUTING.md for this project — include dev setup, branching workflow, coding standards, and PR process
```
Generates a guide tailored to your project's tech stack and workflow.

**Write a security policy**
```
Write a SECURITY.md with responsible disclosure process, supported versions, response SLAs, and safe harbor provisions
```
Produces a security policy following industry best practices for vulnerability reporting.

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `scaffolding-repo-documentation` | Scaffold starter files for all community health documents |
| `writing-readme` | README.md with structured sections adapted by project type |
| `writing-contributing-guide` | CONTRIBUTING.md — dev setup, workflow, coding standards, PR process |
| `writing-security-policy` | SECURITY.md — disclosure process, response SLAs, severity classification, safe harbor |
| `writing-community-policies` | CODE_OF_CONDUCT, SUPPORT, GOVERNANCE, FUNDING.yml |
| `writing-github-config` | Issue templates, PR template, CODEOWNERS, CHANGELOG |
| `auditing-repo-documentation` | 5-dimension documentation health scoring with remediation roadmap |

### Agents

| Agent | Role |
|-------|------|
| `repo_docs_author` | Orchestrates full authoring workflow from scaffolding through writing |
| `repo_docs_auditor` | Read-only health scoring with severity-prioritized findings |

</details>

---

**Domain:** repo-documentation · **Technologies:** Harness-agnostic · **Aliases:** repo-docs, readme-authoring, community-health
