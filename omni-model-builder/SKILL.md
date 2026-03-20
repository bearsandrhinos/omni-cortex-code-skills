---
name: omni-model-builder
description: "Create and edit Omni Analytics semantic model definitions — views, topics, dimensions, measures, relationships, and query views — using YAML through the Omni REST API. Use when: someone wants to add a field, create a new dimension or measure, define a topic, set up joins between tables, modify the data model, build a new view, add a calculated field, create a relationship, edit YAML, work on a branch, or promote model changes. Triggers: add this field, create a view for, set up a join between, model this data, add this metric."
---

# Omni Model Builder

Create and modify Omni's semantic model — views, topics, dimensions, measures, and relationships — using YAML via the REST API.

---

## Prerequisites

```bash
export OMNI_BASE_URL="https://yourorg.omniapp.co"
export OMNI_API_KEY="your-api-key"
```

**Required permissions:** Modeler or Connection Admin role.

> **Always work on a branch — never write directly to the shared model in production.**

---

## Safe Workflow

```
1. Explore existing model   →   omni-model-explorer
2. Create a branch          →   POST /api/v1/models
3. Write YAML to branch     →   PUT /api/v1/models/{modelId}/yaml
4. Validate changes         →   GET /api/v1/models/{modelId}/content-validator
5. Confirm with user        →   ✋ STOP before merging
6. Merge to shared model    →   POST /api/v1/models/{modelId}/merge
```

---

## Step 1 — Check the SQL Dialect

Before writing any SQL expressions, confirm the dialect from the connections endpoint. **Do not guess from the connection name.**

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/connections" | grep -i dialect
```

---

## Step 2 — Create a Branch

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "modelKind": "BRANCH",
    "name": "my-feature-branch",
    "parentModelId": "<sharedModelId>"
  }' \
  "$OMNI_BASE_URL/api/v1/models"
```

Note the returned `modelId` — use this for all subsequent calls on this branch.

---

## Step 3 — Write YAML to the Branch

```bash
curl -s -X PUT \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "ecomm__order_items.view.yaml",
    "content": "<yaml content as string>"
  }' \
  "$OMNI_BASE_URL/api/v1/models/{branchModelId}/yaml"
```

---

## YAML Reference

### View file structure

```yaml
label: Order Items
description: One row per line item in an order

dimensions:
  id:
    primary_key: true
    sql: '"ID"'
    label: Order Item ID

  status:
    sql: '"STATUS"'
    label: Status
    description: Current status of the order item
    synonyms: [ order status, fulfillment state ]

  created_at:
    sql: '"CREATED_AT"'
    label: Created At
    type: time

measures:
  total_sale_price:
    sql: ${sale_price}
    label: Total Sale Price
    aggregate_type: sum
    format: CURRENCY_2
    description: Total revenue from orders
    synonyms: [ Total Revenue, Total Receipts ]

  order_count:
    sql: ${id}
    label: Order Count
    aggregate_type: count
```

---

### Dimension types

**Standard dimension** — maps directly to a column:
```yaml
city:
  sql: '"CITY"'
  label: City
  description: Customer's city
```

**Calculated dimension** — custom SQL expression:
```yaml
full_name:
  sql: CONCAT(${first_name}, ' ', ${last_name})
  label: Full Name
```

**Group dimension** — CASE WHEN grouping:
```yaml
device_category:
  sql: ${device_type}
  label: Device Category
  groups:
    - filter:
        is: [ mobile, tablet ]
      name: Handheld
    - filter:
        is: desktop
      name: Desktop
  else: Other
```

**Bin dimension** — numeric range bucketing:
```yaml
age_bin:
  sql: ${age}
  label: Age Group
  bin_boundaries: [ 18, 35, 50, 65 ]
  description: Customer age group
```

**Duration dimension** — time between two dates:
```yaml
fulfillment_days:
  label: Fulfillment Days
  duration:
    sql_start: ${created_at[date]}
    sql_end: ${delivered_at[date]}
    intervals: [ days ]
```

**Boolean dimension** — becomes a named filter:
```yaml
is_returned:
  sql: '"IS_RETURNED"'
  description: Whether the item was returned
```

---

### Measure types

**Standard measure:**
```yaml
total_revenue:
  sql: ${sale_price}
  aggregate_type: sum
  format: CURRENCY_2
  label: Total Revenue
```

**Filtered measure** — aggregate with CASE WHEN:
```yaml
california_revenue:
  sql: ${sale_price}
  aggregate_type: sum
  label: California Revenue
  filters:
    users.state:
      is: California
```

**Derived measure** — combines existing measures (no `aggregate_type`):
```yaml
gross_margin:
  sql: ${total_revenue} - ${total_cost}
  label: Gross Margin
  format: CURRENCY_2
```

**Count distinct:**
```yaml
unique_customers:
  sql: ${user_id}
  aggregate_type: count_distinct
  label: Unique Customers
```

Supported `aggregate_type` values: `sum`, `count`, `count_distinct`, `avg`, `min`, `max`

---

### Topic file structure

```yaml
label: Order Items
base_view: ecomm__order_items

joins:
  ecomm__users: {}
  ecomm__inventory_items:
    ecomm__products: {}

ai_context: |
  Use this topic to analyze order performance, revenue trends, and customer behavior.
  Always filter out Returned and Cancelled orders unless the user asks about them.

sample_queries:
  Monthly Revenue:
    query:
      fields:
        - "ecomm__order_items.created_at[month]"
        - ecomm__order_items.total_sale_price
      base_view: ecomm__order_items
      sorts:
        - field: "ecomm__order_items.created_at[month]"
    prompt: What is our monthly revenue trend?
```

---

### Relationship file

```yaml
- join_from_view: ecomm__order_items
  join_to_view: ecomm__users
  join_type: always_left
  on_sql: ${ecomm__order_items.user_id} = ${ecomm__users.id}
  relationship_type: assumed_many_to_one
```

---

## Step 4 — Validate the Branch

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{branchModelId}/content-validator"
```

Review any broken field references before proceeding.

> ✋ **STOP** — Show the user the changes and confirm before merging.

---

## Step 5 — Merge to Shared Model

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "branchModelId": "<branchModelId>" }' \
  "$OMNI_BASE_URL/api/v1/models/{sharedModelId}/merge"
```

---

## OpenAPI Reference

When uncertain about available endpoints or parameters:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/openapi.json"
```

---

## Related Skills

- **omni-model-explorer** — explore the existing model before making changes
- **omni-ai-optimizer** — add AI context and sample queries after building
- **omni-query** — test new fields by running queries against the branch
