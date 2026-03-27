# Community Policy Templates

Templates for the four community governance documents. Each template provides the framework-specific structure with bracketed placeholders for project-specific values.

---

## CODE_OF_CONDUCT.md — Contributor Covenant 2.1

Adapted from the [Contributor Covenant](https://www.contributor-covenant.org/), version 2.1.

```markdown
# Contributor Covenant Code of Conduct

## Our Pledge

We as members, contributors, and leaders pledge to make participation in our
community a respectful, inclusive, and positive experience for everyone.

## Our Standards

Examples of behavior that contributes to a positive environment for our community:

- Demonstrating empathy and kindness toward other people
- Being respectful of differing opinions, viewpoints, and experiences
- Giving and gracefully accepting constructive feedback
- Accepting responsibility and apologizing to those affected by our mistakes,
  and learning from the experience
- Focusing on what is best not just for us as individuals, but for the overall
  community

Examples of unacceptable behavior:

- The use of language or imagery with inappropriate connotations
- Trolling, insulting or derogatory comments, and personal attacks
- Publishing others' private information without their explicit permission
- Other conduct which could reasonably be considered inappropriate
  in a professional setting

## Enforcement Responsibilities

Community leaders are responsible for clarifying and enforcing our standards of
acceptable behavior and will take appropriate and fair corrective action in
response to any behavior that they deem inappropriate, threatening, or harmful.

## Scope

This Code of Conduct applies within all community spaces, and also applies when
an individual is officially representing the community in public spaces.

## Enforcement

Instances of unacceptable behavior may be reported to the community leaders
responsible for enforcement at **[INSERT CONTACT EMAIL]**.

All complaints will be reviewed and investigated promptly and fairly. All
community leaders are obligated to respect the privacy and security of the
reporter of any incident.

## Enforcement Guidelines

Community leaders will follow these Community Impact Guidelines in determining
the consequences for any action they deem in violation of this Code of Conduct:

### 1. Correction

**Community Impact**: Use of inappropriate language or other behavior deemed
unprofessional.

**Consequence**: A private, written warning from community leaders, providing
clarity around the nature of the violation and an explanation of why the
behavior was inappropriate. A public apology may be requested.

### 2. Warning

**Community Impact**: A violation through a single incident or series of actions.

**Consequence**: A warning with consequences for continued behavior. No
interaction with the people involved, including unsolicited interaction with
those enforcing the Code of Conduct, for a specified period of time.

### 3. Temporary Restriction

**Community Impact**: A serious violation of community standards, including
sustained inappropriate behavior.

**Consequence**: A temporary restriction from any sort of interaction or public
communication with the community for a specified period of time.

### 4. Permanent Restriction

**Community Impact**: Demonstrating a pattern of violation of community
standards.

**Consequence**: A permanent restriction from any sort of public interaction
within the community.

## Attribution

This Code of Conduct is adapted from the [Contributor Covenant][homepage],
version 2.1, available at
[https://www.contributor-covenant.org/version/2/1/code_of_conduct.html][v2.1].

[homepage]: https://www.contributor-covenant.org
[v2.1]: https://www.contributor-covenant.org/version/2/1/code_of_conduct.html
```

### Customization Checklist

| Placeholder | What to Fill In |
|---|---|
| `[INSERT CONTACT EMAIL]` | Email or contact method for reporting (must be monitored) |

### When to Customize Beyond the Template

- **Large projects**: Add multiple enforcement contacts or a committee
- **Corporate projects**: Reference internal HR or ethics process as an escalation path
- **Events**: Add event-specific scope (applies at conferences, meetups, etc.)

---

## SUPPORT.md Template

```markdown
# Support

## Getting Help

| Need | Channel | Response Time |
|---|---|---|
| Bug reports | [GitHub Issues](../../issues) | Triaged within [X days] |
| Feature requests | [GitHub Issues](../../issues) | Reviewed within [X days] |
| Questions | [GitHub Discussions](../../discussions) | Community-driven |
| Real-time chat | [Discord/Slack invite link] | No guaranteed response time |
| Security issues | See [SECURITY.md](SECURITY.md) | See security policy |

## Before You Ask

1. **Search existing issues** — your question may already be answered
2. **Check the documentation** — especially the README and any docs/ directory
3. **Try the latest version** — the issue may already be fixed

## Reporting a Bug

Use the [Bug Report template](../../issues/new?template=bug-report.yml) and include:
- What you expected to happen
- What actually happened
- Steps to reproduce
- Your environment (OS, runtime version, package version)

## Commercial Support

<!-- Remove this section if not applicable -->
For commercial support, contact [INSERT CONTACT] or visit [INSERT URL].

## Maintainer Availability

<!-- Be honest about availability -->
This project is maintained by [volunteers / a small team / {company}].
Response times may vary. We appreciate your patience.
```

### Customization Checklist

| Placeholder | What to Fill In |
|---|---|
| `[X days]` | Realistic triage cadence (e.g., "5 business days") |
| `[Discord/Slack invite link]` | Invite URL, or remove row if no chat exists |
| Commercial support section | Remove entirely if not applicable |
| Maintainer availability | Describe honest availability level |

---

## Governance Model Decision Tree

Choose the governance model based on project characteristics:

| Factor | BDFL | Consensus | Voting |
|---|---|---|---|
| Maintainer count | 1 | 2-5 | 6+ |
| Decision speed | Fast | Moderate | Slow |
| Contributor base | Small | Medium | Large |
| Formality | Informal | Semi-formal | Formal |
| Best for | Personal projects, early-stage | Established team projects | Foundations, large communities |
| Risk | Bus factor of 1 | Decision paralysis | Bureaucratic overhead |

### Decision Rules

1. **Solo maintainer or personal project**: Use BDFL. You can always upgrade later.
2. **Small team with trust**: Use Consensus. Document what "consensus" means (unanimity vs majority).
3. **Large community or foundation**: Use Voting. Define quorum, voting period, and tie-breaking.
4. **Transitioning**: Moving from BDFL to Consensus is natural as the team grows. Document the transition criteria in the governance file.

---

## GOVERNANCE.md Template

```markdown
# Governance

## Overview

This document describes the governance model for [PROJECT NAME]. It defines
roles, decision-making processes, and how contributors can advance to
maintainers.

## Roles

| Role | Description | Privileges |
|---|---|---|
| **User** | Anyone who uses the project | Open issues, participate in discussions |
| **Contributor** | Has submitted at least one merged pull request | Listed in contributors, can be assigned issues |
| **Maintainer** | Trusted contributor with repository access | Merge PRs, manage issues, publish releases |

### Current Maintainers

<!-- List current maintainers with their GitHub handles -->
- @[maintainer-1]
- @[maintainer-2]

## Decision-Making

<!-- Choose one model and delete the others -->

### Model: BDFL (Benevolent Dictator for Life)

[MAINTAINER NAME] has final decision-making authority. Decisions are made after
consulting with contributors when practical, but the BDFL has the final say.

### Model: Consensus

Maintainers discuss proposals in [GitHub Discussions / issues / meetings].
Decisions are made when all maintainers agree. If consensus cannot be reached
within [TIME PERIOD], the proposal is tabled or put to a vote.

### Model: Voting

Decisions that affect project direction require a formal vote among maintainers.
- **Quorum**: [N] of [M] maintainers must participate
- **Voting period**: [N] days from proposal
- **Approval threshold**: Simple majority / two-thirds majority
- **Tie-breaking**: [BDFL / longest-serving maintainer / random]

## Becoming a Maintainer

Contributors may be nominated for maintainership when they meet these criteria:

1. **Track record**: [N]+ merged pull requests over [N]+ months
2. **Review activity**: Regularly reviews others' pull requests
3. **Community engagement**: Helps triage issues and answer questions
4. **Trust**: Demonstrates good judgment and alignment with project values

### Nomination Process

1. An existing maintainer nominates the contributor
2. Other maintainers discuss and vote (or reach consensus)
3. If approved, the new maintainer is granted repository access and added to
   the maintainers list above

## Conflict Resolution

1. **Direct discussion**: Parties attempt to resolve the disagreement directly
2. **Mediation**: A neutral maintainer facilitates discussion
3. **Decision authority**: [BDFL decides / maintainers vote]
4. **Code of Conduct**: Behavioral issues are handled per the
   [Code of Conduct](CODE_OF_CONDUCT.md)

## Project Lifecycle

| Status | Meaning |
|---|---|
| **Active** | Actively developed and maintained |
| **Maintenance** | Accepting bug fixes, not new features |
| **Archived** | Read-only, no longer maintained |
```

### Customization Checklist

| Placeholder | What to Fill In |
|---|---|
| `[PROJECT NAME]` | Repository or project name |
| `[maintainer-N]` | GitHub handles of current maintainers |
| Decision-making model | Keep one model section, delete the others |
| Maintainership criteria | Set concrete numbers for PRs and time period |
| Conflict resolution | Match to your chosen decision model |

---

## FUNDING.yml Syntax Reference

GitHub's FUNDING.yml supports these platform keys:

```yaml
# Individual GitHub Sponsors
github: [username]
# Or multiple accounts:
github: [username1, username2]

# Open Collective
open_collective: project-name

# Ko-fi
ko_fi: username

# Tidelift
tidelift: platform-name/package-name

# Community Bridge
community_bridge: project-name

# Liberapay
liberapay: username

# IssueHunt
issuehunt: username

# LFX Crowdfunding (formerly Community Bridge)
lfx_crowdfunding: project-name

# Polar
polar: username

# Buy Me a Coffee
buy_me_a_coffee: username

# Patreon
patreon: username

# Custom URLs (up to 4)
custom: ["https://example.com/donate", "https://other.com/sponsor"]
```

### Guidelines for FUNDING.yml

1. Only include platforms you have actively configured and are accepting contributions through
2. Test that the Sponsor button appears on your repository after adding the file
3. Prefer established platforms (GitHub Sponsors, Open Collective) over custom URLs for trust
4. The file must be in `.github/FUNDING.yml` — GitHub will not detect it in other locations
