---
name: writing-github-config
version: 1.0.0
user-invocable: false
description: Author GitHub configuration files including YAML-based issue templates (bug report, feature request), pull request templates, CODEOWNERS for automatic reviewer assignment, and CHANGELOG following Keep a Changelog 1.1.0 format. Use when setting up issue forms, defining PR checklists, configuring code ownership, or establishing changelog conventions. Triggered by: issue template, PR template, pull request template, CODEOWNERS, code owners, CHANGELOG, keep a changelog, issue form, bug report template, feature request template.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# GitHub Configuration Authoring

## Purpose

Author the structural configuration files that shape how contributors interact with a repository's issue tracker, pull request workflow, and release history. These files follow rigid format specifications — YAML schemas for issue templates, markdown checklists for PR templates, glob-based ownership rules for CODEOWNERS, and the Keep a Changelog standard for CHANGELOG.

## Instructions

Follow these steps when authoring or updating GitHub configuration files:

1. **Read existing files**
   - Check `.github/ISSUE_TEMPLATE/` for existing templates (both `.md` and `.yml`)
   - Check `.github/PULL_REQUEST_TEMPLATE.md` and `.github/CODEOWNERS`
   - Check `CHANGELOG.md` at the repository root
   - Flag any legacy markdown issue templates for migration to YAML forms

2. **Identify which config files are needed**
   - Determine project type (library, application, monorepo) and existing workflow
   - Map the project's contribution patterns to the appropriate set of config files
   - Prioritize: issue templates and PR template first, CODEOWNERS and CHANGELOG second

3. **Author each file using the sub-frameworks below**
   - Apply the format-specific rules in the sub-framework sections
   - Use the templates from `./references/github-config-templates.md` as starting points
   - Customize templates to match the project's domain, team structure, and workflow

4. **Validate YAML syntax for issue templates**
   - Verify all required keys are present: `name`, `description`, `body`
   - Confirm each form field has a valid `type` and required `attributes`
   - Check that `labels` reference labels that exist in the repository

5. **Detect team structure for CODEOWNERS**
   - Review git history to identify directory ownership patterns
   - Check for GitHub team references in the organization
   - Map directories to owners, preferring teams over individuals

6. **Write the files**
   - Place issue templates in `.github/ISSUE_TEMPLATE/`
   - Place PR template at `.github/PULL_REQUEST_TEMPLATE.md`
   - Place CODEOWNERS at `.github/CODEOWNERS`
   - For CHANGELOG, preserve existing entries and add the proper header format if missing

## Sub-Frameworks

### Issue Templates (`.github/ISSUE_TEMPLATE/*.yml`)

- Use GitHub's YAML form schema — not legacy markdown templates
- Standard templates: Bug Report (`bug-report.yml`), Feature Request (`feature-request.yml`)
- Each template needs: `name`, `description`, `labels`, `body` with form fields
- Form field types: `markdown`, `textarea`, `input`, `dropdown`, `checkboxes`
- Include a `config.yml` for blank issue settings and contact links
- Quality test: Do the forms render correctly on GitHub? Are required fields marked?

### PR Template (`.github/PULL_REQUEST_TEMPLATE.md`)

- Checklist-based format with HTML comments for instructions
- Sections: Description, Type of Change (checkboxes), Testing, Checklist, Related Issues
- Keep it short — long templates discourage PRs
- Quality test: Can a contributor complete the template in under 2 minutes?

### CODEOWNERS (`.github/CODEOWNERS`)

- Glob-based ownership rules, last matching pattern wins
- Structure: default owner first, then directory-specific, then file-type-specific
- Map ownership to teams (preferred) or individuals
- Quality test: Does every actively-maintained directory have an owner? No stale team references?

### CHANGELOG (`CHANGELOG.md`)

- Keep a Changelog 1.1.0 format
- Section types: Added, Changed, Deprecated, Removed, Fixed, Security
- Always maintain an `[Unreleased]` section at the top
- Link version headers to comparison URLs
- Align with Semantic Versioning
- Quality test: Does every release have a date? Are entries categorized correctly?

## Quality Criteria

| File | Quality Test |
|---|---|
| Issue Templates | YAML syntax valid? Required fields marked? Labels predefined? Renders correctly? |
| PR Template | Completable in under 2 minutes? Includes testing checklist? Links related issues? |
| CODEOWNERS | Every directory covered? Teams exist in org? No stale references? |
| CHANGELOG | Follows Keep a Changelog format? [Unreleased] section present? Dates in ISO 8601? |

## Guidelines

- **DO**: Use YAML issue forms over legacy markdown templates — they provide structured input and validation
- **DO**: Keep PR templates concise — a 5-item checklist is better than a 20-field form
- **DO**: Map CODEOWNERS to teams rather than individuals (survives team changes)
- **DO**: Include an `[Unreleased]` section in CHANGELOG for in-progress changes
- **DON'T**: Auto-generate CHANGELOG from git commits — changelogs are for humans, git log is for developers
- **DON'T**: Set every file as owned by a single person in CODEOWNERS (creates bottlenecks)
- **DON'T**: Use markdown issue templates when YAML forms are available
- **DON'T**: Omit the `config.yml` in ISSUE_TEMPLATE/ — it controls blank issue behavior

## Edge Cases

- **Legacy markdown issue templates**: Offer to migrate to YAML forms. Preserve any custom fields or instructions during conversion.
- **Monorepo CODEOWNERS**: Map each package directory to its owning team. Use wildcard patterns for shared infrastructure.
- **Project without releases yet**: Create CHANGELOG with only the `[Unreleased]` section header.
- **CODEOWNERS with no GitHub teams**: Use individual handles as a starting point, recommend creating teams for scalability.

## Supporting Files

- [GitHub Config Templates](./references/github-config-templates.md)
