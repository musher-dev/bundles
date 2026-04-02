# Musher Bundle Scaffolding

Scaffold new bundle directories with validated naming and standard structure. This skill enforces the bundle naming grammar (`[technology]-[domain]-[purpose]`), scores proposed names against a 12-point evaluation rubric, and creates the directory with a properly structured README.

## Quick Start

Install via the [Musher CLI](https://github.com/musher-dev/musher-cli):

```sh
musher bundle install musher-dev/musher-bundle-scaffolding
```

Then invoke from any compatible harness (Claude Code, Codex, OpenCode, Copilot, Gemini CLI):

```
/create-bundle
```

## Usage Examples

**Scaffold a new bundle**
```
Create a new bundle for Terraform infrastructure governance
```
Validates the name against the rubric (must score 10+/12), creates the directory, and generates the README with proper metadata.

**Validate a bundle name**
```
Would "terraform-infrastructure-governance" be a good bundle name? Score it against the rubric.
```
Evaluates distinctiveness, breadth signaling, discoverability, and consistency before committing to a name.

<details>
<summary><strong>Components</strong></summary>

### Skills

| Skill | Purpose |
|-------|---------|
| `create-bundle` | Scaffold bundle directory with naming validation and README generation |

### References

| Reference | Purpose |
|-----------|---------|
| `naming-grammar` | Controlled vocabulary for domain and purpose segments |
| `evaluation-rubric` | 4-criteria scoring (distinctiveness, breadth, discoverability, consistency) |
| `bundle-readme-template` | Standard README structure for new bundles |

</details>

---

**Domain:** bundle · **Technologies:** Musher · **Aliases:** bundle-creator, scaffold-bundle, new-bundle
