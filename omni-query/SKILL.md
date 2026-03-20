---
name: omni-query
description: "Run queries against Omni Analytics' semantic layer using the REST API, interpret results, and chain queries for multi-step analysis. Use when: someone wants to query data through Omni, run a report, get metrics, pull numbers, analyze data, or retrieve dashboard query results. Triggers: how many, what's the trend, show me the data, run a query, pull this report."
---

# Omni Query

Execute queries against Omni's semantic layer via REST API and interpret the results.

---

## Prerequisites

```bash
export OMNI_BASE_URL="https://yourorg.omniapp.co"
export OMNI_API_KEY="your-api-key"
```

You will also need:
- A **model ID** — use `omni-model-explorer` to find it
- Knowledge of the **topic** and **field names** — use `omni-model-explorer` to discover them

---

## Running a Query

### Basic Query

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "modelId": "<modelId>",
    "query": {
      "table": "<view_name>",
      "fields": [
        "<view_name>.<dimension>",
        "<view_name>.<measure>"
      ],
      "limit": 100
    },
    "format": "csv"
  }' \
  "$OMNI_BASE_URL/api/v1/query/run"
```

> **Tip:** Set `"format": "csv"` for human-readable results. The default response is base64-encoded Apache Arrow format.

---

## Field References

Fields use the format `view_name.field_name`.

**Date fields** support timeframe grouping using bracket notation:

| Field reference | Meaning |
|---|---|
| `order_items.created_at[month]` | Truncate to month |
| `order_items.created_at[year]` | Truncate to year |
| `order_items.created_at[date]` | Truncate to date |
| `order_items.created_at[week]` | Truncate to week |
| `order_items.created_at[quarter]` | Truncate to quarter |

---

## Filtering

Add a `filters` object to the query body:

```json
"filters": {
  "order_items.status": {
    "is": "Complete"
  }
}
```

### Filter expression examples

| Goal | Filter syntax |
|---|---|
| Exact match | `{ "is": "Complete" }` |
| Multiple values | `{ "is": ["New York", "California"] }` |
| Exclude values | `{ "not": ["Returned", "Cancelled"] }` |
| Greater than | `{ ">": 100 }` |
| Between | `{ "between": [10, 100] }` |
| Contains text | `{ "contains": "promo" }` |
| Date range | `{ "time_for_duration": ["6 months ago", "6 months"] }` |
| This year to today | `{ "between_dates": ["this year", "today"] }` |
| Not null | `{ "is_not_null": true }` |

---

## Sorting and Limiting

```json
"sorts": [
  { "field": "order_items.created_at[month]", "desc": true }
],
"limit": 500
```

---

## Long-Running Queries

For large queries that time out, poll the wait endpoint:

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "queryId": "<queryId>" }' \
  "$OMNI_BASE_URL/api/v1/query/wait"
```

---

## Extracting Queries from a Dashboard

To retrieve and re-run the queries powering an existing dashboard:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/queries"
```

> **Note:** Use the document `identifier` field, not `id`. The `id` field may be null for workbook-type documents.

Take each query definition returned and POST it to `/api/v1/query/run`.

---

## Multi-Step Analysis

Chain queries to answer complex questions:
1. Run a broad query to identify dimensions of interest (e.g. top products by revenue)
2. Use the results to parameterize a follow-up query (e.g. trend for those specific products)
3. Combine results and interpret

---

## Known Issues

| Issue | Workaround |
|---|---|
| `IS_NOT_NULL` filters may invert | Verify results look correct; try `is_null: false` if needed |
| Boolean filters drop when `pivots` is present | Remove pivots or apply filter in post-processing |

---

## Related Skills

- **omni-model-explorer** — discover model IDs, topics, and field names
- **omni-content-explorer** — find dashboard identifiers to extract queries from
- **omni-content-builder** — create dashboards from query results
