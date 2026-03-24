# Workspace API Key — Reference

## Base URL
```
https://gateway.srgplus.com
```

## Authentication

Pass your API key in one of two headers:
```
X-API-Key: srgplus_YOUR_KEY
```
or
```
Authorization: Bearer srgplus_YOUR_KEY
```

## Error Codes
| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 204 | Success (no content returned) |
| 400 | Bad request — missing required params or invalid body |
| 401 | Unauthorized — invalid/revoked key or wrong base URL |
| 403 | Forbidden — no access to this resource |
| 404 | Not found |
| 500 | Server error |

## What's Available via API Key
- Content (CRUD)
- Channels (CRUD)
- Channel Categories (CRUD)
- Assets (CRUD)
- Context Widgets (Text, Links)
- Content Sections

## What's NOT Available via API Key
- User management
- Progress tracking
- Community features
- API key management (requires JWT from app)

---

## Contents

### Get Content by ID
```
GET /api/v1/contents/{contentId}?hubProfileId={hubProfileId}
```
`hubProfileId` is **required** as query parameter.

### Create Content
```
POST /api/v1/contents
Content-Type: application/json

{
  "name": "Content Title",
  "hubProfileId": "...",
  "privacy": "Preview",
  "details": "Description",
  "url": "https://...",
  "context": [],
  "channels": [],
  "categories": []
}
```
Required: `name`, `hubProfileId`

### Update Content
```
PUT /api/v1/contents/{contentId}?hubProfileId={hubProfileId}
Content-Type: application/json

{
  "name": "Updated Title",
  "privacy": "Preview",
  "details": "...",
  "url": "...",
  "context": [...],
  "channels": [...],
  "categories": [...]
}
```
Required: `name`, `hubProfileId` (query param)

> **CRITICAL: PUT replaces ALL fields! This WILL destroy data if used incorrectly.**
>
> **MANDATORY workflow — no exceptions:**
> 1. **GET** the content first
> 2. **Copy** ALL existing values from the response
> 3. **Modify** only the fields you need to change
> 4. **Always include cover preservation** (see below)
> 5. **Send** the complete object in PUT
>
> **Always include this to preserve the cover:**
> ```json
> "cover": {
>   "image": null,
>   "generateSignedUrl": false
> }
> ```
> Both `image: null` AND `generateSignedUrl: false` are required. Without `image: null` → 400. Without the whole block → cover deleted.
>
> **What happens if you skip this:**
> - `cover` not included → **cover image DELETED** (must re-upload manually)
> - `channels: []` → removed from all channels
> - `categories: []` → removed from all categories
> - `context: []` → all text/links widgets deleted
>
> **Cover images CANNOT be restored via API** — they require manual re-upload in the app.
> This is the most common mistake and the most painful to fix.

### Filter Content
```
POST /api/v1/contents/{hubProfileId}/filter/
Content-Type: application/json

{
  "pageSize": 20,
  "cursor": null,
  "onlyArchived": false,
  "type": []
}
```

### Search Content
```
POST /api/v1/contents/{hubProfileId}/search
Content-Type: application/json

{
  "search": "keyword",
  "onlyArchived": false
}
```

---

## Context Widgets

Context widgets are added via **Create** or **Update Content** in the `context` array.

### Widget Types

| `$type` | Purpose | Required Fields |
|---------|---------|----------------|
| `Text` | Text block | `content` (string) |
| `LinkList` | List of links | `links` (array) |
| `Media` | Media widget | media data |
| `ContentWidget` | Embedded content | content reference |
| `HubProfile` | Profile reference | profile data |

### Link Types (inside LinkList)

| `$type` | Purpose |
|---------|---------|
| `KnownLink` | URL with auto-fetched favicon/icon |
| `CustomLink` | Plain URL without icon |

Use `KnownLink` for links with preview icons (Google Drive, GHL, etc).

### Example: Update with Context
```bash
curl -X PUT "https://gateway.srgplus.com/api/v1/contents/{contentId}?hubProfileId={hubProfileId}" \
  -H "X-API-Key: srgplus_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Content",
    "context": [
      {
        "$type": "LinkList",
        "title": "Resources",
        "links": [
          {"$type": "KnownLink", "title": "Google Drive", "url": "https://drive.google.com/..."},
          {"$type": "KnownLink", "title": "Planner", "url": "https://app.gohighlevel.com/..."}
        ]
      },
      {
        "$type": "Text",
        "title": "Instagram Caption",
        "content": "Your caption text here..."
      }
    ]
  }'
```

---

## Channels

### Get Channels
```
GET /api/v1/channels/{hubProfileId}?includeArchived=false
```

### Create Channel
```
POST /api/v1/channels
Content-Type: application/json

{
  "name": "Channel Name",
  "hubProfileId": "...",
  "privacy": "Private"
}
```

### Update Channel
```
PUT /api/v1/channels
Content-Type: application/json

{
  "channelId": "...",
  "hubProfileId": "...",
  "name": "Updated Name",
  "privacy": "Private"
}
```

### Delete Channel
```
DELETE /api/v1/channels/{channelId}?hubProfileId={hubProfileId}
```

### Search Channel Content
```
GET /api/v1/channels/search?channelId={channelId}&hubProfileId={hubProfileId}&search={query}
```

---

## Channel Categories

### Create Category
```
POST /api/v1/channels/{channelId}/categories
Content-Type: application/json

{
  "name": "Category Name",
  "isPinned": false,
  "notificationsEnabled": true
}
```

### Update Category
```
PUT /api/v1/channels/{channelId}/categories/{categoryId}
Content-Type: application/json

{
  "name": "Updated Name",
  "isPinned": false,
  "notificationsEnabled": true
}
```

---

## Content ↔ Channel Categories

### Add Content to Category
```
PUT /api/v1/contents/channels/add
Content-Type: application/json

{
  "contentId": "...",
  "channelsCategories": [
    {
      "channelId": "...",
      "categoryIds": ["categoryId1", "categoryId2"]
    }
  ]
}
```

### Remove Content from Category
```
PUT /api/v1/contents/channels/categories/delete
Content-Type: application/json

{
  "contentId": "...",
  "channelsCategories": [
    {"channelId": "...", "categoryId": "..."}
  ]
}
```

### Move Content Between Categories
```
PUT /api/v1/contents/{contentId}/channels/categories/move-to
Content-Type: application/json

{
  "channelId": "...",
  "categoryId": "...",
  "sectionId": "..."
}
```

---

## Content Sections

### Create Section
```
POST /api/v1/contents/{contentId}/Content/sections
Content-Type: application/json

{"name": "Section Name"}
```
Returns: `201` with `{"id": "new_section_id"}`

> **Note:** `categoryName` in URL must be `Content` or `Asset` — NOT the display name.

### Update Section
```
PUT /api/v1/contents/{contentId}/Content/sections/{sectionId}
Content-Type: application/json

{"name": "Updated Name"}
```

### Delete Section
```
DELETE /api/v1/contents/{contentId}/Content/sections/{sectionId}
```

---

## Content Section References (get assets/content from sections)

### Get Section References
```
GET /api/v1/contents/{contentId}/{categoryName}/{sectionId}/references?pageSize=20&order=Ascending
```

- `categoryName`: `Asset` for asset sections, `Content` for content sections
- `order`: `Ascending` or `Descending` (case-sensitive, exactly as written)
- `pageSize`: number of items per page
- Returns signed download URLs for files (valid 7 days)

**Example — get PDF from a section:**
```bash
curl -s "https://gateway.srgplus.com/api/v1/contents/{contentId}/Asset/{sectionId}/references?pageSize=20&order=Ascending" \
  -H "X-API-Key: srgplus_YOUR_KEY"
```

### Get All Category References (without section filter)
```
GET /api/v1/contents/{contentId}/{categoryName}/references?pageSize=20&order=Ascending
```

---

## Assets

### Create Asset
```
POST /api/v1/assets
Content-Type: application/json

{
  "hubProfileId": "...",
  "asset": { ... }
}
```

### Get Asset
```
GET /api/v1/assets/{id}
```

### Filter Assets
```
POST /api/v1/assets/{hubProfileId}/filter/
Content-Type: application/json

{
  "pageSize": 20,
  "type": []
}
```
Types: `Media`, `Embed`, `File`, `Audio`, `Document`, `Other`

### Update Asset
```
PUT /api/v1/assets/{id}
Content-Type: application/json

{
  "name": "Updated Name",
  "readOnly": false
}
```

---

## Quick Start

```bash
API_KEY="srgplus_YOUR_KEY"
HUB_ID="your_hub_profile_id"

# 1. List channels (to get channelId and categoryId)
curl -s "https://gateway.srgplus.com/api/v1/channels/$HUB_ID" \
  -H "X-API-Key: $API_KEY"

# 2. Create content
curl -X POST "https://gateway.srgplus.com/api/v1/contents" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"name\": \"My Content\", \"hubProfileId\": \"$HUB_ID\"}"

# 3. Add content to channel category
curl -X PUT "https://gateway.srgplus.com/api/v1/contents/channels/add" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"contentId\": \"CONTENT_ID\", \"channelsCategories\": [{\"channelId\": \"CHANNEL_ID\", \"categoryIds\": [\"CATEGORY_ID\"]}]}"

# 4. Add context (text + links) via PUT
curl -X PUT "https://gateway.srgplus.com/api/v1/contents/CONTENT_ID?hubProfileId=$HUB_ID" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"name\": \"My Content\", \"context\": [{\"\$type\": \"Text\", \"title\": \"Caption\", \"content\": \"Text here\"}]}"
```
