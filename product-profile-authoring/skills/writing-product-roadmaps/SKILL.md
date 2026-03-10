---
name: writing-product-roadmaps
version: 1.0.0
user-invocable: false
description: Author product roadmaps using the Now/Next/Later temporal planning format with three time-horizon buckets, dependencies, assumptions, and open questions. Use when creating product roadmaps, planning feature timelines, organizing product priorities, or updating roadmap documents. Triggered by: product roadmap, now next later, roadmap.md, temporal planning, product priorities, feature timeline, roadmap update.
allowed-tools: Read, Write, Edit, Glob, Grep
---

# Product Roadmap Authoring

## Purpose

Author `roadmap.md` files using the Now/Next/Later temporal planning format. This format communicates product direction without false precision — commitment decreases as the time horizon extends, which is honest and sustainable.

## Instructions

Follow these steps when authoring or updating a product roadmap:

1. **Read existing product context**
   - Read `product.yaml` for canonical metadata (status, use cases, capabilities)
   - Read `positioning.md` for strategic direction
   - Read any existing `roadmap.md` to understand current state

2. **Categorize initiatives into temporal buckets**
   Use the bucket definitions from `./references/now-next-later-format.md`:

   | Bucket | Commitment Level | Granularity |
   |---|---|---|
   | **Now** | Active, resourced, committed | Specific initiatives with owners and targets |
   | **Next** | Prioritized, planned, flexible | Initiatives with expected outcomes and dependencies |
   | **Later** | Aspirational, no commitment | Directional ideas with open questions |

3. **Frame initiatives as outcomes, not features**
   - Write: "Reduce onboarding time from 2 hours to 15 minutes"
   - Not: "Build setup wizard"
   - Each initiative should answer "what changes for the customer?"

4. **Document dependencies**
   - External dependencies (third-party APIs, partner launches, regulatory deadlines)
   - Internal dependencies (other teams, infrastructure, hiring)
   - Technical prerequisites (migrations, refactors)

5. **Document assumptions**
   - Market assumptions (demand, pricing sensitivity, competitive landscape)
   - Technical assumptions (feasibility, performance targets)
   - Resource assumptions (team size, budget)

6. **Document open questions**
   - Unresolved decisions that could change priorities
   - Research needed before committing to Later items
   - Customer feedback gaps

7. **Write the file**
   - Write to `products/<slug>/roadmap.md`
   - Include `Last updated: [date]` at the top
   - Sync the `roadmap_summary` field in `product.yaml` with a one-line summary of each bucket

## Guidelines

- **DO**: State outcomes, not features — "customers can X" not "build Y"
- **DO**: Decrease specificity as commitment decreases (Now = dates, Next = quarters, Later = no dates)
- **DO**: Include open questions — they signal intellectual honesty and invite input
- **DO**: Update `product.yaml` `roadmap_summary` when updating the roadmap
- **DON'T**: Put dates on Later items — that creates false commitment
- **DON'T**: Use the roadmap to list every possible feature — Later should have 3-5 items max
- **DON'T**: Skip dependencies — they're the most common source of roadmap failure

## Edge Cases

- **Early-stage product (all discovery)**: Everything may be in Later. That's fine — the roadmap documents the thinking, not the plan
- **Mature product (mostly maintenance)**: Now items may be stability/performance focused, not feature-focused. Frame maintenance as outcomes ("99.9% uptime" not "fix bugs")
- **Multi-audience product**: Note which audience each initiative serves — different stakeholders care about different items
- **Roadmap review meeting**: Read existing roadmap, ask what has moved between buckets, and update accordingly

## Supporting Files

- [Now/Next/Later Format Reference](./references/now-next-later-format.md)
