# File Templates

Starter templates for every community health file. Each template provides the structural skeleton with placeholder sections. Substantive content is added later by the individual writing skills.

Substitute `{project-name}` with the actual repository name and `{date}` with today's date in ISO 8601 format.

---

## README.md

```markdown
# {project-name}

> One-sentence description of what this project does and who it is for.

## Quick Start

<!-- Copy-pasteable commands to get running in under 5 minutes -->

## Installation

<!-- Package manager install commands -->

## Usage

<!-- 2-3 most common use cases with code examples -->

## Configuration

<!-- Key configuration options or link to full docs -->

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

<!-- See LICENSE file for details -->
```

---

## CONTRIBUTING.md

```markdown
# Contributing to {project-name}

Thank you for your interest in contributing. This guide explains how to get involved.

## Ways to Contribute

- Report issues
- Suggest features
- Submit pull requests
- Improve documentation

## Prerequisites

<!-- Required tools, versions, and accounts -->

## Development Setup

<!-- Step-by-step local setup instructions -->

## Workflow

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Make your changes
4. Run tests
5. Submit a pull request

## Coding Standards

<!-- Style guide, linting, formatting rules -->

## Testing

<!-- How to run tests, what to test, coverage expectations -->

## Pull Request Process

<!-- PR checklist, review expectations, merge criteria -->

## Recognition

<!-- How contributors are credited -->
```

---

## CODE_OF_CONDUCT.md

```markdown
# Code of Conduct

## Our Pledge

We as members, contributors, and leaders pledge to make participation in our community a positive experience for everyone.

## Our Standards

Examples of behavior that contributes to a positive environment:

- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive feedback
- Focusing on what is best for the community

Examples of unacceptable behavior:

- Personal insults or derogatory comments
- Public or private intimidation
- Publishing others' private information without permission
- Other conduct that is inappropriate in a professional setting

## Enforcement Responsibilities

Community leaders are responsible for clarifying and enforcing standards of acceptable behavior and will take appropriate action in response to any behavior that they deem inappropriate.

## Scope

This Code of Conduct applies within all community spaces and when an individual is officially representing the community in public spaces.

## Enforcement

Instances of unacceptable behavior may be reported to the project team at [INSERT CONTACT METHOD]. All reports will be reviewed and investigated promptly and fairly.

## Enforcement Guidelines

1. **Correction** — A private written warning with clarity around the nature of the issue
2. **Warning** — A formal warning with consequences for continued behavior
3. **Temporary Restriction** — A temporary restriction from community interaction
4. **Permanent Restriction** — A permanent restriction from community interaction

## Attribution

This Code of Conduct is adapted from the [Contributor Covenant](https://www.contributor-covenant.org), version 2.1.
```

---

## SECURITY.md

```markdown
# Security Policy

## Supported Versions

| Version | Supported |
|---|---|
| x.x.x | Yes |
| < x.x.x | No |

<!-- Update with actual version support information -->

## Reporting a Vulnerability

If you discover a security issue, please report it responsibly.

**Do not open a public issue.**

Instead:
1. Email [INSERT SECURITY EMAIL] with a description of the issue
2. Include steps to reproduce the issue if possible
3. Allow a reasonable period for us to address the issue before public disclosure

Alternatively, use [GitHub Security Advisories](https://docs.github.com/en/code-security/security-advisories) to report privately.

## Response Timeline

- **Acknowledgment**: Within 48 hours of receipt
- **Assessment**: Within 1 week of acknowledgment
- **Resolution**: Depends on severity and complexity
- **Disclosure**: Coordinated with the reporter

## Scope

This policy applies to the {project-name} codebase and its official distributions.
```

---

## SUPPORT.md

```markdown
# Support

## How to Get Help

| Channel | Use For |
|---|---|
| [GitHub Issues](../../issues) | Bug reports and confirmed problems |
| [GitHub Discussions](../../discussions) | Questions, ideas, and general discussion |
| <!-- Chat/Forum link --> | Real-time conversation |

## Before Asking

1. Search existing issues and discussions
2. Check the documentation
3. Try the latest version

## Reporting a Bug

Use the [Bug Report issue template](../../issues/new?template=bug-report.yml) and include:
- What you expected to happen
- What actually happened
- Steps to reproduce
- Environment details (OS, runtime version, etc.)
```

---

## CHANGELOG.md

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

### Changed

### Deprecated

### Removed

### Fixed

### Security
```

---

## GOVERNANCE.md

```markdown
# Governance

## Overview

This document describes the governance model for {project-name}.

## Roles

### Users
Anyone who uses {project-name}.

### Contributors
Anyone who has submitted a merged pull request.

### Maintainers
Contributors with write access to the repository. Responsible for reviewing and merging pull requests, triaging issues, and making releases.

<!-- List current maintainers -->

## Decision-Making

<!-- Describe decision-making model: consensus, voting, or BDFL -->

## Becoming a Maintainer

<!-- Describe the path from contributor to maintainer -->

## Conflict Resolution

<!-- Describe how disagreements are resolved -->
```

---

## LICENSE (placeholder)

```
This project has not yet selected a license.

Visit https://choosealicense.com to choose an appropriate license for your project.

IMPORTANT: Without a license, default copyright laws apply, meaning that no one
may reproduce, distribute, or create derivative works from this project.
```

---

## .github/ISSUE_TEMPLATE/bug-report.yml

```yaml
name: Bug Report
description: Report a bug or unexpected behavior
labels: ["bug", "triage"]
body:
  - type: markdown
    attributes:
      value: |
        Thank you for reporting a bug. Please fill out the sections below.
  - type: textarea
    id: description
    attributes:
      label: Description
      description: A clear description of the bug
    validations:
      required: true
  - type: textarea
    id: reproduction
    attributes:
      label: Steps to Reproduce
      description: Steps to reproduce the behavior
      placeholder: |
        1. Go to '...'
        2. Run '...'
        3. See error
    validations:
      required: true
  - type: textarea
    id: expected
    attributes:
      label: Expected Behavior
      description: What you expected to happen
    validations:
      required: true
  - type: textarea
    id: actual
    attributes:
      label: Actual Behavior
      description: What actually happened
    validations:
      required: true
  - type: textarea
    id: environment
    attributes:
      label: Environment
      description: Relevant environment details
      placeholder: |
        - OS: [e.g., macOS 14.0, Ubuntu 22.04]
        - Runtime: [e.g., Node.js 20, Python 3.12]
        - Version: [e.g., 1.2.3]
    validations:
      required: false
```

---

## .github/ISSUE_TEMPLATE/feature-request.yml

```yaml
name: Feature Request
description: Suggest a new feature or improvement
labels: ["enhancement"]
body:
  - type: markdown
    attributes:
      value: |
        Thank you for suggesting a feature. Please fill out the sections below.
  - type: textarea
    id: problem
    attributes:
      label: Problem Statement
      description: What problem does this feature solve?
      placeholder: I'm always frustrated when...
    validations:
      required: true
  - type: textarea
    id: solution
    attributes:
      label: Proposed Solution
      description: How would you like this to work?
    validations:
      required: true
  - type: textarea
    id: alternatives
    attributes:
      label: Alternatives Considered
      description: What alternatives have you considered?
    validations:
      required: false
  - type: textarea
    id: context
    attributes:
      label: Additional Context
      description: Any other context, screenshots, or examples
    validations:
      required: false
```

---

## .github/ISSUE_TEMPLATE/config.yml

```yaml
blank_issues_enabled: true
contact_links:
  - name: Questions and Discussion
    url: https://github.com/{owner}/{project-name}/discussions
    about: Ask questions and discuss ideas here
```

---

## .github/PULL_REQUEST_TEMPLATE.md

```markdown
## Description

<!-- What does this PR do? Why is it needed? -->

## Type of Change

- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update
- [ ] Refactoring
- [ ] Other (describe below)

## Testing

- [ ] Existing tests pass
- [ ] New tests added (if applicable)
- [ ] Manual testing performed

## Checklist

- [ ] My code follows the project's coding standards
- [ ] I have updated documentation as needed
- [ ] I have added tests that prove my fix or feature works
- [ ] All new and existing tests pass

## Related Issues

<!-- Link related issues: Closes #123, Fixes #456 -->
```

---

## .github/CODEOWNERS

```
# Code Owners
# https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners
#
# These owners will be automatically requested for review
# when someone opens a pull request that modifies matching files.
#
# Order matters: the last matching pattern takes precedence.

# Default owner for everything in the repo
# * @org/team-name

# Documentation
# /docs/ @org/docs-team
# README.md @org/docs-team

# CI/CD
# .github/ @org/platform-team
```

---

## .github/FUNDING.yml

```yaml
# Funding platforms
# https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/displaying-a-sponsor-button-in-your-repository

# github: [username]
# open_collective: project-name
# patreon: username
# custom: ["https://example.com/donate"]
```
