---
name: omni-admin
description: "Administer an Omni Analytics instance — manage connections, users, groups, user attributes, permissions, schedules, and schema refreshes via the REST API. Use when: someone wants to manage users or groups, set up permissions on a dashboard or folder, configure user attributes, create or modify schedules, manage database connections, refresh a schema, set up access controls, or provision users. Triggers: add a user, give access to, set up permissions, who has access, configure connection, refresh the schema, schedule a delivery."
---

# Omni Admin

Manage users, groups, permissions, schedules, connections, and schema refreshes via the Omni REST API.

---

## Prerequisites

```bash
export OMNI_BASE_URL="https://yourorg.omniapp.co"
export OMNI_API_KEY="your-api-key"
```

> ⚠️ **This skill requires an Organization API Key** — not a Personal Access Token. Generate one from Admin → API Keys in the Omni UI.

---

## User Management (SCIM 2.0)

### Find a user by email

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/scim/v2/Users?filter=email+eq+%22user@example.com%22"
```

### Create a user

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "userName": "user@example.com",
    "name": {
      "givenName": "First",
      "familyName": "Last"
    },
    "emails": [{ "value": "user@example.com", "primary": true }],
    "active": true
  }' \
  "$OMNI_BASE_URL/api/scim/v2/Users"
```

### Deactivate a user

```bash
curl -s -X PATCH \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{ "op": "replace", "path": "active", "value": false }]
  }' \
  "$OMNI_BASE_URL/api/scim/v2/Users/{userId}"
```

### Delete a user

```bash
curl -s -X DELETE \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/scim/v2/Users/{userId}"
```

---

## Group Management (SCIM 2.0)

### List groups

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/scim/v2/Groups"
```

### Create a group

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "displayName": "Data Team",
    "members": [
      { "value": "<userId>" }
    ]
  }' \
  "$OMNI_BASE_URL/api/scim/v2/Groups"
```

### Add a member to a group

```bash
curl -s -X PATCH \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
      "op": "add",
      "path": "members",
      "value": [{ "value": "<userId>" }]
    }]
  }' \
  "$OMNI_BASE_URL/api/scim/v2/Groups/{groupId}"
```

---

## User Attributes

User attributes power row-level security. Assign values per user to filter data automatically.

### List available attributes

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/user-attributes"
```

### Assign an attribute to a user

```bash
curl -s -X PUT \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "value": "West" }' \
  "$OMNI_BASE_URL/api/v1/user-attributes/{attributeId}/users/{userId}"
```

---

## Model Roles

### Assign a model role to a user

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "<userId>",
    "role": "viewer"
  }' \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/roles"
```

Supported roles: `viewer`, `querier`, `editor`, `modeler`

### Assign a model role to a group

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "groupId": "<groupId>",
    "role": "viewer"
  }' \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/roles"
```

---

## Document & Folder Permissions

### Get permissions on a document

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/permissions"
```

### Set permissions on a document

```bash
curl -s -X PUT \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "permissions": [
      { "userId": "<userId>", "role": "viewer" },
      { "groupId": "<groupId>", "role": "editor" }
    ]
  }' \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/permissions"
```

> ⚠️ Always use the document `identifier` field, not `id`.

### Folder permissions

```bash
curl -s -X PUT \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "permissions": [
      { "groupId": "<groupId>", "role": "viewer" }
    ]
  }' \
  "$OMNI_BASE_URL/api/v1/folders/{folderId}/permissions"
```

---

## Schedules

### Create a delivery schedule

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "documentIdentifier": "<identifier>",
    "name": "Weekly Sales Report",
    "frequency": "weekly",
    "dayOfWeek": "monday",
    "hour": 8,
    "timezone": "America/New_York",
    "format": "pdf",
    "recipients": [
      { "email": "team@example.com" }
    ]
  }' \
  "$OMNI_BASE_URL/api/v1/schedules"
```

### List schedules for a document

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/documents/{identifier}/schedules"
```

### Delete a schedule

```bash
curl -s -X DELETE \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/schedules/{scheduleId}"
```

---

## Connections & Schema Refreshes

### List connections

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/connections"
```

### Trigger a schema refresh

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/connections/{connectionId}/refresh-schema"
```

---

## Validation Tools

### Content validator — find broken field references

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/content-validator"
```

### Reset cache policy

```bash
curl -s -X POST \
  -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/models/{modelId}/reset-cache"
```

---

## Related Skills

- **omni-model-builder** — manage model branches and YAML
- **omni-content-explorer** — find documents and folders to set permissions on
- **omni-content-builder** — create content before managing access to it
- **omni-embed** — configure embed users and entity workspaces
