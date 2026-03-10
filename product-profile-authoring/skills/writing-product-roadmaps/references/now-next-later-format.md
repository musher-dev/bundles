# Now/Next/Later Roadmap Format

A temporal planning format that communicates product direction with honest commitment levels. Popularized by Janna Bastow (ProdPad) as an alternative to timeline-based roadmaps.

## Bucket Definitions

### Now — Active & Committed

**Definition**: Work that is actively in progress with committed resources. The team has made a decision to do this and is executing.

| Attribute | Description |
|---|---|
| **Commitment** | High — resourced and in progress |
| **Granularity** | Specific: initiative name, expected outcome, owner, target date or sprint |
| **Typical horizon** | Current sprint to ~4 weeks |
| **Change policy** | Changing Now items has a cost — it means stopping work in progress |

**Framing guidance**: Now items should be stated as outcomes with measurable completion criteria. "Launch beta to 10 design partners" not "build feature X."

### Next — Prioritized & Flexible

**Definition**: Work that has been prioritized and will start when current work completes. The order is intentional but the timing is flexible.

| Attribute | Description |
|---|---|
| **Commitment** | Medium — prioritized but not yet started |
| **Granularity** | Moderate: initiative name, expected outcome, known dependencies |
| **Typical horizon** | ~1-3 months |
| **Change policy** | Reordering Next items is normal and expected as context changes |

**Framing guidance**: Next items should describe what the customer will be able to do, not implementation details. Include dependencies that could affect sequencing.

### Later — Aspirational & Uncommitted

**Definition**: Directional ideas that the team believes are worth exploring but has not committed to delivering. These may never happen.

| Attribute | Description |
|---|---|
| **Commitment** | Low — no promise of delivery |
| **Granularity** | Low: initiative name, expected outcome, open questions |
| **Typical horizon** | 3+ months or undefined |
| **Change policy** | Later items can be added, removed, or promoted freely |

**Framing guidance**: Later items should articulate the desired outcome and the open questions that must be answered before committing. "Self-hosted deployment option — need to understand demand, support cost, and security implications."

## Outcomes vs. Features

| Feature-Oriented (Avoid) | Outcome-Oriented (Prefer) |
|---|---|
| Build dashboard | Enable teams to monitor key metrics in real-time |
| Add SSO support | Enterprise customers can use existing identity providers |
| Create API v2 | Integrators can build workflows with 50% less code |
| Implement caching | Page loads complete in under 200ms at p99 |
| Build mobile app | Users can triage alerts from anywhere |

**Test**: If an initiative could be implemented multiple ways, you've stated it as an outcome. If it specifies the implementation, you've stated it as a feature.

## Audience Variance

Different audiences read the roadmap differently:

| Audience | What They Look For | How to Serve Them |
|---|---|---|
| **Customers** | When will my pain point be addressed? | Tag initiatives by audience segment |
| **Sales** | What can I promise prospects? | Now = safe to promise, Next = mention as direction, Later = don't promise |
| **Engineering** | What should I prepare for? | Include dependencies and technical prerequisites |
| **Executives** | Is the strategy coherent? | Ensure Now/Next/Later tells a strategic story, not just a task list |
| **Investors** | Is the product evolving? | Show progression from current state toward vision |

## Movement Between Buckets

Items move between buckets based on decisions, not time:

```
Later → Next: Team decides this is worth prioritizing
             Trigger: Customer demand, strategic shift, dependency resolved

Next → Now:  Team decides to start work
             Trigger: Current Now item completed, resources available

Now → Done:  Work completed
             Trigger: Completion criteria met

Any → Removed: Team decides not to pursue
               Trigger: Strategy change, low demand, better alternative found
```

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Dates on Later items | Creates false commitment | Remove dates; state open questions instead |
| 20+ items in Later | Wish list, not a roadmap | Limit to 3-5 directional themes |
| Empty Now section | No execution clarity | If nothing is active, that's a planning problem to address |
| No open questions | False certainty | Every Later item should have at least one open question |
| Feature-only framing | No customer value visible | Rewrite as outcomes |
| No dependencies section | Roadmap failures surprise the team | Document all external and internal dependencies |
