# Content Management Guide

Step-by-step guide for common content operations via the SRG+ API.

## Workflow: Create Content with Captions and Links

### Step 1: Get your Hub Profile channels
```bash
curl -s "https://gateway.srgplus.com/api/v1/channels/{hubProfileId}" \
  -H "X-API-Key: srgplus_YOUR_KEY"
```
Save the `channelId` and `categoryId` from the response.

### Step 2: Create content
```bash
curl -X POST "https://gateway.srgplus.com/api/v1/contents" \
  -H "X-API-Key: srgplus_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Brand Kit",
    "hubProfileId": "YOUR_HUB_ID",
    "context": [
      {
        "$type": "LinkList",
        "title": "Resources",
        "links": [
          {"$type": "KnownLink", "title": "Final Files", "url": "https://drive.google.com/..."}
        ]
      },
      {
        "$type": "Text",
        "title": "Instagram",
        "content": "Your Instagram caption here...\n\n#hashtags"
      },
      {
        "$type": "Text",
        "title": "Threads",
        "content": "Your Threads caption here..."
      }
    ]
  }'
```
Save the `id` from the response.

### Step 3: Add to channel category
```bash
curl -X PUT "https://gateway.srgplus.com/api/v1/contents/channels/add" \
  -H "X-API-Key: srgplus_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contentId": "CONTENT_ID_FROM_STEP_2",
    "channelsCategories": [
      {
        "channelId": "CHANNEL_ID_FROM_STEP_1",
        "categoryIds": ["CATEGORY_ID_FROM_STEP_1"]
      }
    ]
  }'
```

---

## Workflow: Update Context (safely)

### Step 1: GET current content
```bash
curl -s "https://gateway.srgplus.com/api/v1/contents/{contentId}?hubProfileId={hubProfileId}" \
  -H "X-API-Key: srgplus_YOUR_KEY"
```

### Step 2: Note existing values
Save: `name`, `privacy`, `details`, `url`, `channels`, `categories` from response.

### Step 3: PUT with ALL fields + modified context
```bash
curl -X PUT "https://gateway.srgplus.com/api/v1/contents/{contentId}?hubProfileId={hubProfileId}" \
  -H "X-API-Key: srgplus_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "KEEP ORIGINAL NAME",
    "privacy": "KEEP ORIGINAL",
    "details": "KEEP ORIGINAL",
    "url": "KEEP ORIGINAL",
    "channels": [... KEEP ORIGINAL ...],
    "categories": [... KEEP ORIGINAL ...],
    "context": [
      ... YOUR NEW CONTEXT WIDGETS ...
    ]
  }'
```

---

## Workflow: Manage Channel Categories

### Add content to a category
```bash
curl -X PUT "https://gateway.srgplus.com/api/v1/contents/channels/add" \
  -H "X-API-Key: srgplus_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contentId": "...",
    "channelsCategories": [
      {"channelId": "...", "categoryIds": ["..."]}
    ]
  }'
```

### Remove content from a category
```bash
curl -X PUT "https://gateway.srgplus.com/api/v1/contents/channels/categories/delete" \
  -H "X-API-Key: srgplus_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contentId": "...",
    "channelsCategories": [
      {"channelId": "...", "categoryId": "..."}
    ]
  }'
```

---

## Workflow: Manage Sections

### Create a section inside content
```bash
curl -X POST "https://gateway.srgplus.com/api/v1/contents/{contentId}/Content/sections" \
  -H "X-API-Key: srgplus_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Section Name"}'
```
`categoryName` must be `Content` or `Asset`.

### Delete a section
```bash
curl -X DELETE "https://gateway.srgplus.com/api/v1/contents/{contentId}/Content/sections/{sectionId}" \
  -H "X-API-Key: srgplus_YOUR_KEY"
```
