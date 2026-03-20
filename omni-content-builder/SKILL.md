---
name: omni-content-builder
description: "Create, update, and manage Omni Analytics documents and dashboards programmatically — document lifecycle, tiles, visualizations, filters, and layouts — using the REST API. Use when: someone wants to build a dashboard, create a workbook, add tiles or charts, configure dashboard filters, update an existing dashboard, set up a KPI view, create visualizations, rename a workbook, move a document to a folder, or duplicate a dashboard. Triggers: build a dashboard for, create a report showing, add a chart to, make a dashboard, update the dashboard layout."
---

# Omni Content Builder

Create and manage Omni dashboards, workbooks, tiles, and filters via the REST API.

---

## Prerequisites

```bash
export OMNI_BASE_URL="https://yourorg.omniapp.co"
export OMNI_API_KEY="your-api-key"
```

> ⚠️ **Critical rules:**
> - Always use `identifier` — **not** `id` — for document API calls
> - Every query tile must include **at least one measure** or the tile will be empty
> - Always set `prefersChart: true` and choose an appropriate `chartType` — **never default to table**
> - Choose chart types based on the data: `line` for trends, `bar` for comparisons, `single_value` for KPIs, `pie` for part-to-whole
> - Only **published** documents can be duplicated — drafts cannot

---

## Recommended Workflow

```
1. Discover fields       →   omni-model-explorer
2. Find reference dashboards  →  omni-content-explorer
3. Create document with all queries and filters in a single API call
4. Refine layout in the UI
```

---

## Creating a Document

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sales Overview",
    "folderId": "<folderId>"
  }' \
  "$OMNI_BASE_URL/api/v1/documents"
```

Note the returned `identifier` for all subsequent calls.

---

## Adding a Query Tile

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "modelId": "<modelId>",
      "table": "order_items",
      "fields": [
        "order_items.created_at[month]",
        "order_items.total_sale_price"
      ],
      "sorts": [{ "field": "order_items.created_at[month]" }],
      "limit": 500
    },
    "visualization": {
      "prefersChart": true,
      "chartType": "line",
      "xAxis": "order_items.created_at[month]",
      "yAxis": "order_items.total_sale_price"
    },
    "title": "Monthly Revenue"
  }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/tiles"
```

### Choosing a chart type

Always set `prefersChart: true` and pick the most appropriate `chartType` for the data. **Do not use `table` unless the user explicitly asks for one.**

| `chartType` | When to use |
|---|---|
| `line` | Trends over time (time series with a date dimension) |
| `bar` | Comparing values across categories |
| `area` | Cumulative or stacked trends over time |
| `single_value` | KPIs and scorecards (single measure, no dimension) |
| `pie` | Part-to-whole breakdown (few categories, ≤ 6) |
| `scatter` | Correlation between two measures |
| `map` | Data with a geographic dimension (country, state, city) |
| `table` | Only when the user explicitly asks for a table |

---

## Adding a Text / Header Tile

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "text",
    "content": "## Sales Performance\nKey metrics for the current period."
  }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/tiles"
```

---

## Adding a Dashboard Filter

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "field": "order_items.status",
    "label": "Order Status",
    "defaultValue": "Complete"
  }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/filters"
```

> ⚠️ Boolean filters may fail silently when the query includes `pivots`. If a filter seems to have no effect, check for pivot fields in the query.

---

## Updating a Document

### Rename

```bash
curl -s -X PATCH \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "name": "New Dashboard Name" }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}"
```

### Move to a folder

```bash
curl -s -X PATCH \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "folderId": "<targetFolderId>" }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}"
```

---

## Duplicating a Document

Only works on published documents.

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "name": "Copy of Sales Overview" }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/duplicate"
```

---

## Deleting a Document

```bash
curl -s -X DELETE \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}"
```

---

## Publishing a Document

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/publish"
```

---

## Workbook-Specific Model Customizations

Workbooks can extend the shared model with their own joins or field definitions:

```bash
curl -s -X PUT \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "yaml": "<workbook-level yaml>",
    "filename": "custom_view.view.yaml"
  }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/model"
```

---

## Direct Links

| Content type | URL |
|---|---|
| Dashboard | `{OMNI_BASE_URL}/dashboards/{identifier}` |
| Workbook | `{OMNI_BASE_URL}/dashboards/{identifier}` |

---

## Known Issues

| Issue | Workaround |
|---|---|
| Boolean filters drop with pivots | Remove pivots or filter in post-processing |
| Complex chart types show "No chart available" | Check that `prefersChart: true` is set and `xAxis`/`yAxis` fields match the query fields |
| Draft documents cannot be duplicated | Publish first, then duplicate |

---

## Related Skills

- **omni-model-explorer** — discover fields and model IDs before building
- **omni-content-explorer** — find existing dashboards to reference or duplicate
- **omni-query** — test queries before embedding them in tiles
- **omni-admin** — set permissions on new dashboards and folders
