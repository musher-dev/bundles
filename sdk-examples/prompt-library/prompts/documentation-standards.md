## Documentation Standards

Standards for writing technical documentation that developers can navigate quickly and trust completely.

### Principles

**Accuracy over completeness.** A short, correct document is better than a long, partially outdated one. If you can't verify something, leave it out rather than guessing.

**Task orientation.** Organize around what the reader needs to accomplish, not around the system's internal structure. Lead with the user's goal, not the component's architecture.

**Progressive disclosure.** Show the simplest working example first. Layer in advanced options, configuration, and edge cases afterward. Readers who need the basics should not have to wade through expert details.

### Page Types

**Quickstart** — get the reader to a working result in under 5 minutes. Include: prerequisites, installation, one working example, expected output, and a link to next steps.

**How-to guide** — solve a specific problem. Include: the goal, prerequisites, numbered steps, complete code examples, and common errors. Do not explain concepts — link to reference material instead.

**Reference** — document every parameter, field, and option. Include: type, default value, description, and at least one example per entry. Keep descriptions factual and concise.

**Troubleshooting** — help the reader diagnose and fix a problem. Include: symptom, possible causes, diagnostic steps, and resolution for each cause.

### Writing Style

- Use second person ("you") and active voice
- Use present tense for descriptions ("This function returns...") and imperative for instructions ("Run the following command")
- Keep sentences under 25 words
- One idea per paragraph — 2-3 sentences maximum
- Use code blocks for anything the reader will type or see in output

### Code Examples

- Every example must be complete and runnable as-is
- Use realistic values, not `foo`, `bar`, or `example`
- Show expected output after each command or code block
- Include error handling in examples — readers will copy them
- Pin dependency versions in example configuration files
