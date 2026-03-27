---
name: writing-security-policy
version: 1.0.0
user-invocable: false
description: Author SECURITY.md files that enable responsible disclosure with supported versions tables, reporting channels, response timelines, severity classification, and safe harbor provisions. Use when writing security policies, setting up vulnerability reporting, or defining disclosure processes. Triggered by: SECURITY, security policy, vulnerability reporting, responsible disclosure, security contact, supported versions.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Security Policy Authoring

## Purpose

Author `SECURITY.md` files that give security researchers a clear, trusted path to report issues privately. A good security policy protects both the project (by enabling private disclosure before public exposure) and the reporter (by providing safe harbor and response commitments). Without one, researchers may disclose publicly by default, or worse, not report at all.

## Instructions

Follow these steps when authoring or updating a project's security policy:

1. **Read existing file or stub**
   - Look for an existing `SECURITY.md` in the repository root
   - Read it if present to understand current coverage and gaps
   - If no file exists, suggest running the `scaffolding-repo-documentation` skill first

2. **Determine supported versions**
   - Inspect package manifests (`package.json`, `pyproject.toml`, `Cargo.toml`, etc.) for the current version
   - Review release tags, changelogs, or GitHub Releases to identify which versions receive security updates
   - Build a supported versions table with clear "Supported" / "Not Supported" designations

3. **Define the reporting channel**
   - Choose the appropriate channel using the Reporting Channel Decision Matrix below
   - GitHub Security Advisories is recommended for open-source projects (free, private by default)
   - For email channels, confirm the address is monitored and not a personal inbox
   - For bug bounty platforms, confirm budget and program scope are established

4. **Author sections**
   - Work through each section systematically using the framework in `./references/security-policy-framework.md`
   - Set concrete response SLAs by severity level using the Severity Classification table below
   - Every SLA must use specific timeframes, not vague language like "as soon as possible"

5. **Add safe harbor language**
   - Include an explicit statement that good-faith security reporters will not face legal or retaliatory action
   - Reference the `./references/security-policy-framework.md` safe harbor template
   - Adapt the template to match the project's legal context and organizational policies

6. **Write the file**
   - Place `SECURITY.md` in the repository root
   - Verify that the reporting channel is actually reachable and monitored
   - Confirm all SLAs are achievable given the project's maintainer capacity

## Quality Criteria

| Section | Quality Test |
|---|---|
| Supported Versions | Lists specific version ranges? Updated within the last 6 months? |
| Reporting Channel | Provides a concrete contact method? Not just "email us"? |
| Response Timeline | States specific SLAs (e.g., "acknowledge within 48 hours")? |
| Scope | Defines what is in scope for reporting? |
| Safe Harbor | Explicitly states reporters will not face adverse action? |

## Reporting Channel Decision Matrix

| Channel | Best For | Pros | Cons |
|---|---|---|---|
| GitHub Security Advisories | Open-source projects | Free, integrated, private by default | Requires GitHub |
| Dedicated email | Any project | Simple, universally accessible | No tracking, may go to spam |
| Bug bounty platform | Projects with budget | Structured process, incentives | Cost, setup overhead |

## Severity Classification

| Severity | Definition | Response SLA |
|---|---|---|
| Critical | Can be exploited remotely without authentication | Acknowledge: 24h, Fix: 7 days |
| High | Requires some conditions to exploit but has significant impact | Acknowledge: 48h, Fix: 30 days |
| Medium | Limited impact or requires significant preconditions | Acknowledge: 1 week, Fix: 90 days |
| Low | Minimal impact, theoretical, or defense-in-depth | Acknowledge: 2 weeks, Fix: next release |

## Guidelines

- **DO**: Enable GitHub Security Advisories for the repository (free and built-in)
- **DO**: State concrete response timelines with actual numbers
- **DO**: Include safe harbor language to protect good-faith reporters
- **DO**: Keep the supported versions table current with each release
- **DON'T**: Require reporters to use PGP encryption (offer it as an option, not a requirement)
- **DON'T**: Set SLAs you cannot keep -- be honest about volunteer project limitations
- **DON'T**: Require NDAs or legal agreements before accepting a report
- **DON'T**: Leave the reporting email unmonitored

## Edge Cases

- **Volunteer/open-source project**: Use longer SLAs and state explicitly that maintainers are volunteers. Recommend GitHub Security Advisories as the zero-cost option.
- **Enterprise product**: Reference your organization's existing security response team and CVE assignment process.
- **No current maintainer**: Direct reporters to the nearest active fork or upstream project. State that the project is unmaintained.

## Supporting Files

- [Security Policy Framework](./references/security-policy-framework.md)
