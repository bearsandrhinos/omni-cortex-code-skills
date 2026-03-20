---
name: omni-content-explorer
description: "Find, browse, and organize content in Omni Analytics — dashboards, workbooks, folders, and labels — using the REST API. Use when: someone wants to find an existing dashboard, search for content, list workbooks, browse folders, download a dashboard, manage labels, or see what reports exist. Triggers: find the dashboard about, what reports do we have, show me our dashboards, where is the sales report, download this dashboard."
---

# Omni Content Explorer

Discover, browse, and organize dashboards, workbooks, and folders in Omni via the REST API.

---

## Prerequisites

```bash
export OMNI_BASE_URL="https://yourorg.omniapp.co"
export OMNI_API_KEY="your-api-key"
```

> ⚠️ **Critical:** Always use the `identifier` field — **not** `id` — for all document API calls. The `id` field may be `null` for workbook-type documents, causing silent failures.

---

## Listing Content

### All content

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents"
```

### Filter by label

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents?label=<label-name>"
```

### Sort options

| Parameter | Result |
|---|---|
| `?sort=popular` | Most viewed first |
| `?sort=recent` | Most recently updated first |
| `?sort=name` | Alphabetical |

---

## Getting a Specific Document

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}"
```

Returns full metadata including title, folder, labels, and content type.

---

## Retrieving Dashboard Queries

Get the query definitions powering every tile in a dashboard:

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/queries"
```

Use these with `omni-query` to re-run or adapt the queries programmatically.

---

## Folders

### List all folders

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/folders"
```

### Create a folder

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "name": "My New Folder" }' \
  "$OMNI_BASE_URL/api/v1/folders"
```

---

## Labels

### List all labels

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/labels"
```

### Add a label to a document

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "label": "<label-name>" }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/labels"
```

### Remove a label from a document

```bash
curl -s -X DELETE \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/labels/<label-name>"
```

---

## Favorites

### Mark as favorite

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/favorite"
```

### Remove from favorites

```bash
curl -s -X DELETE \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/favorite"
```

---

## Downloading a Dashboard

Dashboard exports are asynchronous. First initiate the job, then poll for completion.

### Start the export

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "format": "pdf" }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/export"
```

Supported formats: `pdf`, `png`

### Poll for completion

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/export-jobs/{jobId}"
```

Poll until `status` is `complete`, then download from the returned URL.

---

## Direct Links

| Content type | URL format |
|---|---|
| Dashboard | `{OMNI_BASE_URL}/dashboards/{identifier}` |
| Workbook | `{OMNI_BASE_URL}/w/{identifier}` |

---

## Related Skills

- **omni-query** — run queries from dashboards you've found
- **omni-content-builder** — create or update dashboards
- **omni-admin** — manage permissions on folders and documents
- **omni-embed** — embed dashboards in external applications
