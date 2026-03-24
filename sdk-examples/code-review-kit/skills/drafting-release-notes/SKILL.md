---
name: drafting-release-notes
description: >
  Generate release notes from git history, pull request descriptions,
  and changelog entries. Use when preparing a release, summarizing
  changes for stakeholders, or updating a CHANGELOG file.
---

# Drafting Release Notes

Generate clear, audience-appropriate release notes by analyzing the changes between two reference points (tags, branches, or commits).

## Process

1. **Gather changes** — collect commits, merged PRs, and closed issues between the two reference points.

2. **Categorize** — group changes into standard sections:
   - **Added** — new features or capabilities
   - **Changed** — modifications to existing behavior
   - **Fixed** — bug fixes
   - **Removed** — deprecated features that were deleted
   - **Security** — vulnerability patches or security improvements
   - **Infrastructure** — CI/CD, build, or dependency changes

3. **Summarize each entry** — write one concise line per change that describes the user-facing impact, not the implementation detail. Reference the PR or issue number.

4. **Highlight breaking changes** — call out anything that requires user action (API changes, config format changes, removed features) in a dedicated section at the top.

5. **Add context** — include upgrade instructions for breaking changes and link to relevant documentation or migration guides.

## Format

Follow [Keep a Changelog](https://keepachangelog.com/) conventions:

```markdown
## [version] - YYYY-MM-DD

### Breaking Changes
- Description of breaking change (#PR)

### Added
- Description of new feature (#PR)

### Fixed
- Description of bug fix (#PR)
```

## Guidelines

- Write for the user of the software, not the developer who made the change
- Omit internal refactors, test-only changes, and CI tweaks unless they affect users
- Use past tense ("Added", "Fixed") for consistency
- Keep entries to one line — link to the PR for details
