---
name: omni-model-explorer
description: "Discover and inspect Omni Analytics models, topics, views, fields, dimensions, measures, and relationships using the Omni REST API. Use when: you need to understand what data is available in Omni, explore the semantic model, find specific fields or views, check how tables join together, or see what topics exist. Triggers: what can I query, what fields are available, show me the model, what data do we have, how is this data modeled."
---

# Omni Model Explorer

Browse and inspect Omni's semantic layer — models, topics, views, fields, and relationships.

---

## Prerequisites

Ensure these environment variables are set before running any API calls:

```bash
export OMNI_BASE_URL="https://yourorg.omniapp.co"
export OMNI_API_KEY="your-api-key"
```

---

## Model Architecture

Omni has three model layers:

| Layer | Description |
|---|---|
| **Schema Model** | Auto-generated from the database schema |
| **Shared Model** | Curated, organization-wide definitions — always prefer this |
| **Workbook Model** | Dashboard-level customizations; scoped to a single document |

Always target the **Shared Model** unless the user specifies otherwise.

---

## Workflow

### Step 1 — List Available Models

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models"
```

Identify the shared model and note its `modelId`. To include active branches:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models?includeBranches=true"
```

---

### Step 2 — List Topics in the Model

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/topics"
```

Topics join multiple views together with defaults and AI context. Identify the topic most relevant to the user's question.

---

### Step 3 — Inspect a Topic

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/topic/{topicName}"
```

This returns:
- The base view and all joined views
- Available fields, dimensions, and measures
- AI context and sample queries
- Join relationships and types

---

### Step 4 — Retrieve Model YAML

To inspect raw YAML definitions for views, dimensions, or measures:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/yaml"
```

Filter by filename:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/yaml?filename=ecomm__order_items.view.yaml"
```

Filter by branch:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/yaml?branch=my-branch"
```

---

## Key Concepts

### Views
Views map directly to database tables. A view file defines:
- **Dimensions** — attributes (columns) of the table
- **Measures** — aggregations (SUM, COUNT, AVG, etc.)
- **Primary keys** — marked with `primary_key: true`

### Topics
Topics are the entry point for querying. They define:
- A `base_view` (the primary table)
- `joins` — how other views connect to the base
- `fields` — which fields are exposed
- `ai_context` — guidance for Omni's AI assistant (Blobby)
- `sample_queries` — example queries for AI and users

### Field References
Fields are referenced as `view_name.field_name`. Date fields support timeframe grouping:
- `order_items.created_at[month]`
- `order_items.created_at[year]`
- `order_items.created_at[date]`

---

## Advanced

### Field Impact Analysis

To find where a field is used before modifying it, search across the model YAML for the field name. Use the content-validator endpoint to identify broken references:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/content-validator"
```

### OpenAPI Reference

When uncertain about available endpoints or parameters:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/openapi.json"
```

---

## Related Skills

- **omni-query** — run queries once you know the fields
- **omni-model-builder** — add or modify fields and views
- **omni-ai-optimizer** — improve AI context and sample queries
