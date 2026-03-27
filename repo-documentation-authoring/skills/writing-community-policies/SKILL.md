---
name: writing-community-policies
version: 1.0.0
user-invocable: false
description: Author community governance documents including CODE_OF_CONDUCT.md based on Contributor Covenant 2.1, SUPPORT.md with support channel routing, GOVERNANCE.md with decision-making models and maintainer paths, and FUNDING.yml for sponsorship configuration. Use when establishing community standards, defining support channels, or documenting project governance. Triggered by: code of conduct, community standards, support channels, governance model, funding, sponsor button, maintainer path, contributor covenant.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Community Policy Authoring

## Purpose

Author the community governance documents that set expectations for how people interact with and contribute to a project. This skill covers four related files that share a common concern — defining the social contract between a project and its community. Each document reinforces the others: the code of conduct sets behavioral standards, the support guide routes people to the right channels, the governance model defines decision-making authority, and the funding configuration enables financial sustainability.

## Instructions

Follow these steps when authoring or updating community policy documents:

1. **Read existing files or stubs**
   - Check for existing `CODE_OF_CONDUCT.md`, `SUPPORT.md`, `GOVERNANCE.md`, and `.github/FUNDING.yml`
   - Identify which of the four documents are needed — not every project requires all four
   - Review any existing content to preserve project-specific customizations

2. **Gather project context**
   - Determine project size: solo maintainer, small team (2-5), or large community (6+)
   - Identify governance model: BDFL (benevolent dictator for life), consensus, or voting
   - Catalog support channels currently in use: GitHub Issues, Discussions, Discord, Slack, mailing list
   - Assess funding status: actively seeking sponsors, already funded, or not applicable

3. **Author each applicable file**
   - Work through each document using the sub-frameworks below
   - Use templates from `./references/community-policy-templates.md` as the structural starting point
   - Fill in all bracketed placeholders with project-specific values

4. **Customize for project context**
   - Solo maintainer projects need different governance than large foundations
   - Adjust response time expectations to match actual maintainer availability
   - Scale enforcement processes to the size of the community
   - Match support channel complexity to the number of active contributors

5. **Cross-reference between documents**
   - `CODE_OF_CONDUCT.md` should reference enforcement contacts that are reachable
   - `SUPPORT.md` should reference issue templates if they exist
   - `GOVERNANCE.md` should reference `CONTRIBUTING.md` for contribution mechanics
   - `FUNDING.yml` platforms should match links mentioned elsewhere in the repository

6. **Write the files**
   - Place `CODE_OF_CONDUCT.md`, `SUPPORT.md`, and `GOVERNANCE.md` at the repository root
   - Place `FUNDING.yml` inside the `.github/` directory
   - Verify that all contact methods, links, and channel references are valid

## Sub-Frameworks

### CODE_OF_CONDUCT.md — Contributor Covenant 2.1

Based on the [Contributor Covenant](https://www.contributor-covenant.org/) version 2.1:

- **Sections**: Pledge, Standards, Enforcement Responsibilities, Scope, Enforcement Guidelines, Attribution
- **Enforcement Ladder** (four levels):
  1. **Correction** — private written warning with clarity on the violation
  2. **Warning** — formal warning with consequences for continued behavior
  3. **Temporary Restriction** — temporary ban from community interaction
  4. **Permanent Restriction** — permanent ban from all community spaces
- **Quality test**: Does it name a specific enforcement contact? Does it define concrete consequences?

### SUPPORT.md — Support Channel Routing

A channel matrix that routes users to the correct venue based on their need:

- **Channel matrix** mapping issue type to channel (bugs to Issues, questions to Discussions, real-time chat to Discord/Slack)
- **Triage labels** and expected response cadence per channel
- **Quality test**: Can a user determine the right channel in under 10 seconds?

### GOVERNANCE.md — Decision-Making and Roles

Define how decisions are made and how contributors advance:

- **Three governance models**:
  - Benevolent Dictator (solo/small) — one person has final say
  - Consensus (medium) — maintainers discuss until agreement
  - Voting (large/foundation) — formal votes with quorum rules
- **Role definitions**: User, Contributor, Committer, Maintainer
- **Path to maintainership** with concrete criteria (e.g., "10+ merged PRs and 3 months of consistent review activity")
- **Quality test**: Could a contributor determine how to become a maintainer?

### FUNDING.yml — GitHub Sponsors Configuration

Configure the Sponsor button on the repository:

- **Platform options**: `github`, `open_collective`, `patreon`, `custom`
- Only include platforms that are actively configured and accepting contributions
- **Quality test**: Does the Sponsor button appear correctly on the repository?

## Quality Criteria

| Document | Quality Test |
|---|---|
| CODE_OF_CONDUCT | Names enforcement contact? Defines concrete consequences? Based on recognized standard? |
| SUPPORT | Routes to specific channels? Sets response expectations? |
| GOVERNANCE | Defines decision-making model? Documents path to maintainership? |
| FUNDING.yml | Only lists active platforms? Valid YAML syntax? |

## Guidelines

- **DO**: Base CODE_OF_CONDUCT on the Contributor Covenant — it is the most widely recognized standard
- **DO**: Be honest about response times — "no guaranteed SLA" is better than a broken promise
- **DO**: Define concrete criteria for becoming a maintainer (not "when we feel you're ready")
- **DO**: Keep FUNDING.yml minimal — only list platforms you actively use
- **DON'T**: Write a code of conduct without an enforcement process
- **DON'T**: Create GOVERNANCE.md for a solo project unless you plan to grow the team
- **DON'T**: Promise commercial support in SUPPORT.md if none exists
- **DON'T**: Copy the Contributor Covenant verbatim without filling in the contact details

## Edge Cases

- **Solo maintainer**: Use BDFL governance model, set realistic response expectations in SUPPORT, skip FUNDING.yml unless actively seeking sponsors
- **Large open-source foundation**: Formal voting governance, multiple enforcement contacts for CODE_OF_CONDUCT, tiered support channels
- **Corporate open-source**: May need to reference corporate policies alongside community standards
- **Inactive/archived project**: State clearly that the project is no longer actively maintained in SUPPORT and GOVERNANCE

## Supporting Files

- [Community Policy Templates](./references/community-policy-templates.md)
