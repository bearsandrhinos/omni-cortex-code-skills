---
name: omni-embed
description: "Embed Omni Analytics dashboards in external applications — URL signing, custom themes, iframe events, entity workspaces, and permission-aware content — using the @omni-co/embed SDK and REST API. Use when: someone wants to embed a dashboard, sign an embed URL, customize the embedded theme, handle embed events, listen for clicks or drills in the iframe, send filters to an embedded dashboard, set up entity workspaces, look up embed users, build a permission-aware content list, or white-label a dashboard. Triggers: embed this dashboard, customize the iframe theme, handle click events from the embed, filter the embedded dashboard, set up embedding, what dashboards can this user see."
---

# Omni Embed

Embed Omni dashboards in external applications using signed URLs, custom themes, and iframe event handling.

---

## Prerequisites

```bash
export OMNI_BASE_URL="https://yourorg.omniapp.co"
export OMNI_API_KEY="your-api-key"
export OMNI_EMBED_SECRET="your-embed-secret"
```

**Setup checklist:**
- Install the embed SDK: `npm install @omni-co/embed`
- Configure your embed domain in Omni: Admin → Embed
- Note your embed secret from Admin → Embed

> ⚠️ **Domain note:** Embed URLs use `.embed-omniapp.co` — REST API calls use `.omniapp.co`. These are different domains.

---

## URL Signing

Signed embed URLs are generated **server-side** to keep the embed secret secure. Never sign URLs in client-side code.

### Using the SDK

```typescript
import { embedSsoDashboard } from '@omni-co/embed';

const signedUrl = await embedSsoDashboard({
  organizationName: 'yourorg',
  secret: process.env.OMNI_EMBED_SECRET,
  userEmail: 'user@example.com',
  dashboardId: '<identifier>',
  // Optional: row-level security attributes
  userAttributes: {
    region: 'West',
    department: 'Sales'
  },
  // Optional: pre-apply filters
  filters: {
    'order_items.status': 'Complete'
  }
});
```

### Manual signing (without SDK)

Build the payload:

```json
{
  "organizationName": "yourorg",
  "userEmail": "user@example.com",
  "dashboardId": "<identifier>",
  "expiresAt": "<unix timestamp>",
  "userAttributes": {},
  "filters": {}
}
```

Sign with HMAC-SHA256 using `OMNI_EMBED_SECRET`, then base64url-encode and append as a query parameter.

---

## Rendering the Embed

```html
<iframe
  src="<signedUrl>"
  width="100%"
  height="800px"
  frameborder="0"
  allowfullscreen>
</iframe>
```

Or use the SDK's iframe helper:

```typescript
import { createEmbedIframe } from '@omni-co/embed';

const iframe = createEmbedIframe({
  url: signedUrl,
  container: document.getElementById('dashboard-container'),
  width: '100%',
  height: '800px'
});
```

---

## Theme Customization

Control the visual style of embedded content:

```typescript
const signedUrl = await embedSsoDashboard({
  // ...auth params...
  theme: {
    // Dashboard background
    dashboardBackgroundColor: '#F8F9FA',

    // Tile styling
    tileBackgroundColor: '#FFFFFF',
    tileBorderRadius: '8px',
    tileShadow: '0 1px 4px rgba(0,0,0,0.1)',

    // Typography
    fontFamily: 'Inter, sans-serif',
    fontSize: '14px',
    textColor: '#1A1A2E',

    // Controls
    controlBackgroundColor: '#FFFFFF',
    controlBorderColor: '#DEE2E6',

    // Buttons
    buttonBackgroundColor: '#4F46E5',
    buttonTextColor: '#FFFFFF',

    // Header
    showHeader: false,        // hide Omni branding
    showFilterBar: true
  }
});
```

---

## Iframe Events (postMessage)

Listen for events from the embedded dashboard using `window.addEventListener('message', ...)`.

### Dashboard lifecycle

```javascript
window.addEventListener('message', (event) => {
  if (event.origin !== 'https://yourorg.embed-omniapp.co') return;

  const { type, data } = event.data;

  switch (type) {
    case 'dashboard:loaded':
      console.log('Dashboard ready');
      break;

    case 'dashboard:filters:changed':
      console.log('Filters changed:', data.filters);
      break;

    case 'dashboard:tile-drill':
      // User clicked a data point
      console.log('Drill event:', data);
      break;
  }
});
```

### Sending filters to the embed

```javascript
iframe.contentWindow.postMessage({
  type: 'dashboard:filters:update',
  data: {
    filters: {
      'order_items.status': 'Complete',
      'users.country': 'United States'
    }
  }
}, 'https://yourorg.embed-omniapp.co');
```

### Custom visualization events

```javascript
// Listen for clicks on custom viz elements
window.addEventListener('message', (event) => {
  if (event.data.type === 'custom-viz:click') {
    const { field, value } = event.data;
    // handle the click
  }
});
```

---

## Entity Workspaces

Entity workspaces let embed users create and save their own dashboards within a scoped folder.

```typescript
import { embedSsoDashboard, EmbedSessionMode } from '@omni-co/embed';

const signedUrl = await embedSsoDashboard({
  // ...auth params...
  sessionMode: EmbedSessionMode.Application,
  entityId: 'customer-123',       // unique identifier for this embed user's workspace
  entityFolderName: 'My Reports'  // display name for the workspace folder
});
```

---

## Permission-Aware Content

### List documents accessible to an embed user

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/embed/users/{embedUserEmail}/documents"
```

### Look up an embed user

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/embed/users?email={embedUserEmail}"
```

### Get folder mappings for embed users

```bash
curl -s -H "Authorization: Bearer $OMNI_API_KEY" \
  "$OMNI_BASE_URL/api/v1/embed/folders"
```

Use these to build a content picker that only shows dashboards the current user can access.

---

## White-Labeling Checklist

To fully white-label the embedded experience:

- [ ] Set `showHeader: false` in the theme
- [ ] Set `showFilterBar: false` if using custom filter UI
- [ ] Match `fontFamily` and colors to your application
- [ ] Use your own domain (configure custom domains in Admin → Embed)
- [ ] Handle drill events and open them in your own UI instead of Omni

---

## Security Notes

- Always sign URLs server-side — never expose `OMNI_EMBED_SECRET` to the browser
- Set short expiry times on signed URLs (e.g. 5–15 minutes)
- Validate `event.origin` when handling postMessage events
- Use `userAttributes` for row-level security rather than filtering in the URL

---

## Related Skills

- **omni-content-explorer** — find dashboard identifiers to embed
- **omni-admin** — manage embed users, groups, and permissions
- **omni-content-builder** — create dashboards to embed
