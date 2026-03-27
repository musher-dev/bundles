---
name: scaffolding-repo-documentation
version: 1.0.0
user-invocable: true
description: Scaffold a new repository's community health files with starter templates for README, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, SUPPORT, CHANGELOG, issue templates, PR template, CODEOWNERS, GOVERNANCE, and FUNDING.yml. Use when initializing a new repo, bootstrapping community health files, or setting up GitHub discoverability. Triggered by: scaffold repo docs, community health files, initialize repo, repo setup, bootstrap repo, new repo documentation.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
argument-hint: <project-root-path>
---

# Scaffold Repository Documentation

## Purpose

Create the full set of GitHub-recommended community health files for a repository. This is the entry point for the repo-documentation-authoring workflow — run this first, then use the individual writing skills to flesh out each file with substantive content.

## Instructions

Follow these steps when this skill is activated:

1. **Determine the project root**
   - Use the path provided by the user, or default to the current working directory
   - Verify it is a git repository (look for `.git/` directory)
   - If not a git repo, warn the user and ask whether to proceed

2. **Detect the project type**
   - Inspect the repository for signals:
     - `package.json`, `setup.py`, `Cargo.toml`, `go.mod` → Library or Application
     - `docs/`, `examples/`, plugin/extension architecture → Framework
     - Repository name `.github` under an org → Org-level defaults
   - Consult `./references/project-type-matrix.md` to determine which files are Required, Recommended, or Optional
   - Present the detected type and recommended file set to the user for confirmation

3. **Scan for existing files**
   - Search the repo root and `.github/` directory for each community health file
   - Report which files already exist and which are missing
   - For existing files, ask the user whether to skip, overwrite, or merge

4. **Generate stub files**
   - Use the templates from `./references/file-templates.md`
   - Place files according to GitHub's discovery rules:
     - **Root directory**: README.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, SECURITY.md, SUPPORT.md, CHANGELOG.md, LICENSE, GOVERNANCE.md
     - **`.github/` directory**: ISSUE_TEMPLATE/, PULL_REQUEST_TEMPLATE.md, CODEOWNERS, FUNDING.yml
   - Substitute `{project-name}` placeholders with the actual repository name
   - Set date placeholders to today's date

5. **Handle LICENSE**
   - Create a LICENSE placeholder with a comment directing the user to choosealicense.com
   - Never select a license — this is a legal decision for the project owner
   - If a LICENSE file already exists, always skip it

6. **Output summary**
   - List all created files with their paths
   - List any skipped files and the reason
   - Suggest the next writing skill to use (typically `writing-readme` or `writing-contributing-guide`)
   - Note any files marked as "Recommended" or "Optional" that were skipped

## File Placement Rules

| File | Location | GitHub Discovery |
|---|---|---|
| README.md | Root | Rendered on repo homepage |
| CONTRIBUTING.md | Root | Linked from "Contributing" sidebar |
| CODE_OF_CONDUCT.md | Root | Linked from "Code of conduct" sidebar |
| SECURITY.md | Root | Linked from "Security" tab |
| SUPPORT.md | Root | Linked from "Support" sidebar |
| CHANGELOG.md | Root | No automatic GitHub integration |
| LICENSE | Root | Detected and displayed in repo sidebar |
| GOVERNANCE.md | Root | No automatic GitHub integration |
| ISSUE_TEMPLATE/ | .github/ | Populates issue creation form |
| PULL_REQUEST_TEMPLATE.md | .github/ | Pre-fills PR description |
| CODEOWNERS | .github/ | Assigns reviewers automatically |
| FUNDING.yml | .github/ | Enables "Sponsor" button |

## Guidelines

- **DO**: Use the exact templates from references — consistency across projects matters
- **DO**: Create the `.github/` directory if it doesn't exist
- **DO**: Respect existing files — never silently overwrite
- **DON'T**: Fill in substantive content — that's the job of the individual writing skills
- **DON'T**: Choose a license for the user
- **DON'T**: Create files marked as "Optional" for the project type unless the user explicitly requests them

## Edge Cases

- **Monorepo**: Create root-level community health files for the entire repo. Individual packages may need their own README.md but share all other files from the root
- **Org-level `.github` repo**: All files become defaults for every repo in the organization. Skip CHANGELOG (not applicable). Emphasize that CODE_OF_CONDUCT and SECURITY apply org-wide
- **Existing partial documentation**: Report what exists, offer to scaffold only the missing files
- **Non-GitHub hosting** (GitLab, Bitbucket): Templates are still valuable but file discovery paths differ. Note the platform difference in the summary

## Supporting Files

- [File Templates](./references/file-templates.md)
- [Project Type Matrix](./references/project-type-matrix.md)
