# Schema.org Property Mapping

Maps `product.yaml` keys to Schema.org [Product](https://schema.org/Product) and [SoftwareApplication](https://schema.org/SoftwareApplication) properties.

## Mapping Table

| YAML Key | Schema.org Property | Type | Notes |
|---|---|---|---|
| `id` | `identifier` | Text | Unique product identifier |
| `name` | `name` | Text | Official product name |
| `tagline` | `slogan` | Text | Short value proposition |
| `status` | `creativeWorkStatus` | Text | Lifecycle stage (custom vocabulary) |
| `owner` | `maintainer` | Organization / Person | Responsible team or individual |
| `audiences` | `audience.audienceType` | Text[] | Target user roles and contexts |
| `problem_statement` | `description` | Text | Extended description of the problem solved |
| `use_cases` | `featureList` | Text[] | Primary use cases as verb-noun pairs |
| `core_capabilities` | `offers.itemOffered` | Text[] | What the product delivers |
| `differentiators` | `additionalProperty` | PropertyValue[] | Unique capabilities with proof |
| `non_goals` | — | Text[] | No direct Schema.org mapping |
| `pricing_packaging` | `offers` | Offer | Pricing model summary |
| `integrations` | `softwareRequirements` | Text[] | Connected systems and APIs |
| `security_compliance` | `additionalProperty` | PropertyValue[] | Compliance certifications |
| `proof_points` | `review` / `aggregateRating` | Review[] | Metrics, testimonials, case studies |
| `docs_url` | `url` | URL | Documentation link |
| `marketing_url` | `sameAs` | URL | Marketing site link |
| `roadmap_summary` | `releaseNotes` | Text | High-level roadmap state |
| `last_reviewed` | `dateModified` | Date | ISO 8601 date of last review |

## Usage Notes

- Fields without a direct Schema.org mapping (`non_goals`) use `additionalProperty` with a custom `propertyID` when generating JSON-LD
- The `differentiators` array maps to `additionalProperty` with `propertyID: "differentiator"` and `value` containing the capability, while `proof` maps to `description`
- `status` uses a custom vocabulary (`discovery | development | beta | ga | sunset | retired`) that maps to `creativeWorkStatus` for semantic web compatibility
- Downstream consumers can transform this YAML into Schema.org JSON-LD using the mapping above
