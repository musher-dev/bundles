# Data Workflows

Musher bundle with a data analysis skill for CSV exploration, aggregation, and summarization. Designed as the primary bundle for the OpenAI hosted container skill example, where the agent creates and manipulates files in a `container_auto` environment.

## Registry

- **Name:** `musher-examples/data-workflows`
- **Version:** 1.0.0
- **Replaces:** `musher-examples/data-workflows:1.0.0` (SDK placeholder)

## Assets

| Logical Path | Type | Description |
|---|---|---|
| `skills/csv-insights/SKILL.md` | skill | CSV data analysis, aggregation, and summarization |

## Used By SDK Examples

### Python SDK
- `examples/openai/hosted_inline_skill.py` — exports skill as an inline hosted OpenAI shell skill in a `container_auto` environment
