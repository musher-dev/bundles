# README Section-by-Section Framework

A README is the first file most visitors read. It must pass the 30-Second Test: within one screenful of content, a reader should know what this project is, who it is for, and how to start using it. This framework defines each section's purpose, contents, quality bar, and common mistakes.

## The 30-Second Test

Before diving into individual sections, every README must satisfy three questions above the fold:

1. **What is this?** -- A single sentence or short paragraph naming the problem and the solution.
2. **Who is it for?** -- The target audience stated explicitly, not implied.
3. **How do I start?** -- A visible path to first usage, either inline or as a prominent link.

If a visitor has to scroll past badges, logos, or organizational boilerplate before finding these answers, the README fails. Move decoration below the essentials.

## Section-by-Section Guide

### 1. Title and Badges

**Purpose:** Identify the project and communicate status at a glance.

**What to include:**
- Project name as an H1 heading, matching the package or repository name exactly
- A concise tagline on the line immediately below the title (optional but recommended)
- Badges in a single row: build status, latest version, license, downloads (in that order)
- Link each badge to its source (CI dashboard, package registry, license file)

**Quality test:** Does the title match what a user would type in a package manager search? Do badges link to live, accurate sources? Are there fewer than 5 badges?

**Common mistakes:**
- Badge overload -- rows of badges push the description below the fold
- Stale badges pointing to broken CI or outdated versions
- Using the GitHub repo name when the package name differs
- Decorative badges (code style, "made with love") that add no decision-relevant information

### 2. Description

**Purpose:** Explain what the project does, who benefits, and why it exists -- in one paragraph.

**What to include:**
- What the project does (the capability)
- Who it is for (the audience)
- Why it matters (the value over alternatives or the status quo)
- Keep to 2-4 sentences maximum

**Quality test:** Could someone with no prior context read this paragraph and explain the project to a colleague? Does it avoid jargon, or define any jargon it uses? Does it state outcomes rather than implementation details?

**Common mistakes:**
- Starting with "This is a..." -- describe what it does, not what it is
- Listing technologies instead of capabilities ("Built with React, Node, and PostgreSQL")
- Assuming the reader knows the problem domain
- Writing a mission statement instead of a product description

### 3. Visual Element

**Purpose:** Show what the project looks like in action -- a screenshot, demo GIF, or architecture diagram.

**What to include:**
- A single visual that demonstrates the primary use case
- Alt text describing the visual for accessibility
- Caption text if the visual needs context
- For CLI tools: a terminal recording (asciinema) or formatted output block
- For libraries: skip this section if there is no visual output

**Quality test:** Does the visual show the project doing its primary job? Is it current (matches the latest version)? Does it have descriptive alt text?

**Common mistakes:**
- Outdated screenshots that do not match the current UI
- GIFs that are too long (keep under 15 seconds) or too large (keep under 5MB)
- Missing alt text, making the README inaccessible to screen readers
- Architecture diagrams that are too detailed for a first impression

### 4. Quick Start

**Purpose:** Get the reader from zero to working in under 5 minutes with copy-pasteable commands.

**What to include:**
- Prerequisites stated upfront (runtime version, OS, required tools)
- Numbered steps, each with a single command
- Expected output after each significant command
- A "you should see" confirmation so the reader knows it worked
- Total step count: aim for 3-5 steps maximum

**Quality test:** Could a new developer follow these steps on a fresh machine without asking questions? Do commands include version-pinned dependencies? Is expected output shown?

**Common mistakes:**
- Assuming tools are already installed (Node, Python, Docker)
- Combining multiple commands into one step without explanation
- Omitting expected output, leaving the reader uncertain if it worked
- Linking to another page instead of showing the steps inline
- Using placeholder values without marking them clearly (e.g., `YOUR_API_KEY`)

### 5. Installation

**Purpose:** Document every supported way to install the project, with version-pinned examples.

**What to include:**
- One subsection per package manager or installation method
- Version-pinned commands (e.g., `npm install package@^2.0.0`)
- System requirements and prerequisites
- Verification command to confirm successful installation
- Platform-specific notes where applicable (macOS, Linux, Windows)

**Quality test:** Are all supported package managers covered? Do examples pin versions? Is there a verification step?

**Common mistakes:**
- Only showing one package manager when multiple are supported
- Using `latest` or unpinned versions in install commands
- Missing prerequisites (e.g., requiring Python 3.10+ but not stating it)
- Forgetting Windows users when the project supports Windows

### 6. Usage

**Purpose:** Demonstrate the 3 most common use cases with runnable code examples.

**What to include:**
- The most basic use case first (the "hello world")
- Two additional examples covering the next most common scenarios
- Each example should be complete and runnable -- no missing imports or setup
- Expected output or behavior after each example
- Language-tagged code fences for syntax highlighting

**Quality test:** Can each example be copied into a file and executed without modification (except configuration values)? Do examples progress from simple to complex? Are imports and setup included?

**Common mistakes:**
- Incomplete examples that omit imports, initialization, or cleanup
- Showing advanced use cases before basic ones
- Examples that depend on external services without noting the dependency
- Using variables without defining them
- Documenting every feature instead of the most common three

### 7. Configuration / API

**Purpose:** Document the key options a user needs to customize behavior, without overwhelming detail.

**What to include:**
- A table of the most important configuration options with name, type, default, and description
- Environment variable names if applicable
- Link to full API documentation or configuration reference for comprehensive details
- Example configuration file if the project uses one

**Quality test:** Are the 5-10 most-used options documented here? Does each option have a default value listed? Is there a link to the full reference?

**Common mistakes:**
- Documenting every option in the README instead of linking to full docs
- Missing default values, forcing readers to check the source code
- Not distinguishing required from optional configuration
- Mixing environment variables with code-level configuration without clear separation

### 8. Contributing

**Purpose:** Welcome contributors and direct them to the right resources.

**What to include:**
- A welcoming sentence acknowledging that contributions are valued
- Link to `CONTRIBUTING.md` for detailed guidelines
- Quick list of ways to contribute (bug reports, feature requests, documentation, code)
- Link to issue tracker with "good first issue" label if available
- Development setup instructions or link to them

**Quality test:** Does the tone welcome newcomers? Is `CONTRIBUTING.md` linked? Are there specific, actionable ways to contribute listed?

**Common mistakes:**
- Saying "contributions welcome" without explaining how
- Repeating the full contributing guide in the README
- Omitting the link to `CONTRIBUTING.md`
- Gatekeeping tone ("only submit PRs if you really know what you're doing")

### 9. License

**Purpose:** State the license type clearly so readers know their rights without opening another file.

**What to include:**
- License name (e.g., "MIT", "Apache 2.0", "GPL-3.0")
- Link to the `LICENSE` file in the repository
- One sentence summarizing what the license permits, if the license is uncommon

**Quality test:** Is the license type named explicitly (not just "see LICENSE file")? Does the link resolve?

**Common mistakes:**
- Writing only "See LICENSE" without naming the license type
- Mismatching the stated license and the actual LICENSE file contents
- Omitting the license section entirely, creating legal ambiguity

### 10. Acknowledgments

**Purpose:** Credit people, projects, and resources that made this project possible.

**What to include:**
- Key contributors or maintainers (link to profiles)
- Projects or libraries that inspired or underpin this work
- Funding sources or sponsors if applicable
- Keep the list short -- this is a thank-you, not a bibliography

**Quality test:** Are credits specific and linked? Is the list concise (under 10 entries)?

**Common mistakes:**
- Turning acknowledgments into a verbose history of the project
- Listing every transitive dependency instead of meaningful inspirations
- Omitting this section entirely when others contributed significantly

## Badge Recommendations

Place badges in a single row directly below the title. Order them by decision relevance:

| Position | Badge | Why |
|---|---|---|
| 1st | Build status | Signals whether the project is healthy right now |
| 2nd | Latest version | Tells the reader what version to install |
| 3rd | License | Answers "can I use this?" immediately |
| 4th | Downloads (optional) | Social proof of adoption |

**Rules:**
- Maximum 4-5 badges. More than that creates visual noise.
- Every badge must link to a live, accurate source.
- Remove badges that are purely decorative (code style, "PRs welcome", emoji badges).
- Use shields.io or the package registry's own badge service for consistency.
- Test badge URLs quarterly -- stale badges erode trust.

## Section Ordering

The recommended section order optimizes for the reader's decision journey:

1. **Title + Badges** -- "Is this project alive?"
2. **Description** -- "What does it do?"
3. **Visual Element** -- "What does it look like?"
4. **Quick Start** -- "Can I try it right now?"
5. **Installation** -- "How do I set it up properly?"
6. **Usage** -- "What can I do with it?"
7. **Configuration / API** -- "How do I customize it?"
8. **Contributing** -- "How do I get involved?"
9. **License** -- "Can I use it?"
10. **Acknowledgments** -- "Who made this?"

This order follows the reader's natural progression from curiosity to evaluation to adoption to contribution. Do not front-load organizational information (contributing, license) before the reader has decided the project is relevant to them.
