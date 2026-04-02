---
name: csv-insights
description: >
  Analyze CSV data by computing aggregations, filtering rows, and generating
  readable summaries. Use when working with tabular data files in a container
  environment where the agent creates and manipulates files directly.
---

# CSV Insights

Analyze CSV files to extract statistics, filter data, and produce human-readable summaries of findings.

## Approach

### 1. Read and Validate the Data

Before computing anything, understand the dataset:

- Load the CSV and inspect the first few rows to confirm column names and types
- Check for missing values, inconsistent formatting, or mixed types in columns
- Note the row count and column count as baseline context
- If the CSV does not exist yet, create it from the provided data with clean headers

### 2. Compute Column Statistics

For each numeric column, compute:

- **Count** — number of non-null values
- **Sum** — total of all values
- **Average** — arithmetic mean
- **Min / Max** — range boundaries
- **Median** — middle value for skew detection

For categorical columns, compute:

- **Distinct values** — unique entries and their counts
- **Most frequent** — mode and its percentage of total rows
- **Missing rate** — percentage of null or empty entries

### 3. Filter and Group

Apply filters and groupings based on the analysis goal:

- Filter rows by column value (exact match, range, or pattern)
- Group by one or more categorical columns
- Compute aggregations within each group (sum, average, count)
- Sort results by the aggregated metric to surface top/bottom segments
- Limit output to the most relevant rows when datasets are large

### 4. Generate a Summary

Produce a readable narrative from the computed results:

- Lead with the key finding — the single most important insight
- Support with 2-3 secondary observations backed by specific numbers
- Flag anomalies — outliers, unexpected nulls, or skewed distributions
- State the scope — how many rows and columns were analyzed, any filters applied
- Use plain language — avoid jargon, explain what the numbers mean in context

## Output Format

Structure results as:

1. **Dataset overview** — row count, column inventory, data quality notes
2. **Statistics table** — per-column metrics in a readable table
3. **Grouped analysis** — aggregations by category if grouping was applied
4. **Key findings** — narrative summary of the most important insights
5. **Data quality notes** — any issues found (missing values, type mismatches, duplicates)

## Guidelines

- Prefer exact numbers over vague descriptions ("revenue is 42,350" not "revenue is high")
- When comparing groups, include both absolute values and relative differences
- If the dataset is too large to display fully, show representative samples and totals
- Preserve the original data — create new files for transformed outputs rather than modifying the source
