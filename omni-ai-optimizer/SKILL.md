---
name: omni-ai-optimizer
description: "Optimize your Omni Analytics model for Blobby, Omni's AI assistant — configure ai_context, ai_fields, sample_queries, and create AI-specific topic extensions. Use when: someone wants to improve AI accuracy in Omni, make Blobby smarter, configure AI context, add example questions, tune AI responses, set up sample queries, curate fields for AI, create AI-optimized topics, or troubleshoot why Blobby gives wrong answers. Triggers: make the AI better, Blobby isn't answering correctly, add context for AI, optimize for AI, teach the AI about our data."
---

# Omni AI Optimizer

Tune Omni's AI assistant (Blobby) by configuring `ai_context`, `ai_fields`, `sample_queries`, and topic extensions.

---

## Prerequisites

```bash
export OMNI_BASE_URL="https://yourorg.omniapp.co"
export OMNI_API_KEY="your-api-key"
```

**Required permissions:** Modeler or Connection Admin role.

> Always work on a branch — use `omni-model-builder` to create one before making changes.

---

## How Blobby Uses Your Model

Blobby analyzes the following when answering questions, in order of impact:

| Signal | Impact |
|---|---|
| `ai_context` | Highest — sets terminology, rules, and perspective |
| `ai_fields` | High — controls which fields Blobby can see |
| `sample_queries` | Medium — teaches by example |
| `synonyms` | Medium — maps alternative names to fields |
| `descriptions` | Lower — adds field-level context |

---

## Optimization Workflow

```
1. Inspect current state     →   review topic and view YAML
2. Review AI usage data      →   identify common questions / failures
3. Write ai_context          →   set rules and terminology
4. Curate ai_fields          →   reduce noise in large models
5. Add sample_queries        →   teach by example
6. Add synonyms              →   map business terms to fields
7. Refine descriptions       →   clarify ambiguous fields
8. Consider topic variants   →   create AI-specific topic extensions
9. Test iteratively          →   run questions through Blobby
```

---

## 1. Writing `ai_context`

Add `ai_context` to your topic YAML. Use it to set:
- **Terminology mapping** — what business terms mean in data terms
- **Behavioral rules** — default filters, date logic, scope
- **Data nuances** — edge cases, exclusions, caveats
- **Analytical perspective** — how to frame answers

```yaml
# in your topic YAML
ai_context: |
  This topic covers e-commerce order performance.

  Key rules:
  - Always filter out 'Returned' and 'Cancelled' orders unless the user specifically asks about them.
  - "Revenue" means total_sale_price, not gross_revenue.
  - Use order_items.created_at for all date filtering unless the user specifies otherwise.
  - "Customer" refers to a unique user_id, not a session.

  Data notes:
  - Orders before 2020-01-01 may have incomplete status data.
  - The 'Pending' status means the order has not yet been confirmed.
```

---

## 2. Curating `ai_fields`

Use `ai_fields` to control which fields Blobby sees. This is especially useful for large models with many redundant or technical fields.

```yaml
# in your topic YAML
ai_fields:
  - order_items.*              # all fields from order_items
  - users.country              # specific field
  - users.state
  - products.brand
  - products.category
  - tag:kpi                    # all fields tagged "kpi"
  - -order_items.internal_id   # exclude with - prefix
  - -order_items.raw_json
```

### Field targeting rules

| Syntax | Meaning |
|---|---|
| `view.*` | All fields in a view |
| `all_views.*` | All fields across all views |
| `view.field` | A specific field |
| `tag:<value>` | All fields with this tag |
| `-view.field` | Exclude this field |

---

## 3. Adding `sample_queries`

Sample queries teach Blobby by example. Pull verified queries from existing workbooks and add them to the topic.

```yaml
sample_queries:
  Monthly Revenue Trend:
    query:
      fields:
        - "order_items.created_at[month]"
        - order_items.total_sale_price
      base_view: order_items
      filters:
        order_items.status:
          not: [ Returned, Cancelled ]
      sorts:
        - field: "order_items.created_at[month]"
      limit: 500
    description: Monthly revenue trend excluding returns and cancellations
    prompt: What is our monthly revenue trend?

  Revenue by Country:
    query:
      fields:
        - users.country
        - order_items.total_sale_price
      base_view: order_items
      filters:
        order_items.status:
          not: [ Returned, Cancelled ]
      sorts:
        - field: order_items.total_sale_price
          desc: true
      limit: 50
    description: Total revenue broken down by user country
    prompt: Which countries drive the most revenue?
```

---

## 4. Adding `synonyms`

Add synonyms to individual field definitions in view YAML to map business terms:

```yaml
dimensions:
  user_id:
    sql: '"USER_ID"'
    label: Customer ID
    synonyms: [ customer, buyer, shopper, account ]

measures:
  total_sale_price:
    sql: ${sale_price}
    aggregate_type: sum
    label: Total Revenue
    synonyms: [ revenue, sales, income, receipts, GMV ]
```

---

## 5. Creating AI-Specific Topic Extensions

For complex models, create a dedicated AI-optimized topic that extends the base topic with curated fields and context:

```yaml
# order_items_ai.topic.yaml
label: Order Items (AI)
extends: order_items

ai_fields:
  - order_items.created_at
  - order_items.total_sale_price
  - order_items.order_count
  - order_items.status
  - users.country
  - users.state
  - products.brand
  - products.category

ai_context: |
  Optimized topic for AI analysis of order performance.
  Default to this topic for all revenue and order questions.
  Always exclude Returned and Cancelled orders.
```

---

## Applying Changes via API

Read the current topic YAML:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/yaml?filename=order_items.topic.yaml"
```

Write the updated YAML to a branch:

```bash
curl -s -X PUT \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "order_items.topic.yaml",
    "content": "<updated yaml>"
  }' \
  "$OMNI_BASE_URL/api/v1/models/{branchModelId}/yaml"
```

---

## Testing

After applying changes, test by asking Blobby the same questions that were previously failing and verify:
- Correct fields are selected
- Default filters are applied
- Terminology is interpreted correctly
- Results match expectations

---

## Related Skills

- **omni-model-explorer** — inspect the current topic and field definitions
- **omni-model-builder** — create branches and write YAML changes
- **omni-query** — verify query results after optimization
