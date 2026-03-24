# Troubleshooting

## 401 Unauthorized on all endpoints

**Cause:** Wrong gateway URL.

**Fix:** Use `https://gateway.srgplus.com` — NOT the old Cloud Run URL.

```
WRONG: https://srg-api-gateway-tqqbslb7yq-uc.a.run.app
RIGHT: https://gateway.srgplus.com
```

## 400 Bad Request on GET /contents/{id}

**Cause:** Missing `hubProfileId` query parameter.

**Fix:** Add `?hubProfileId=YOUR_HUB_ID`:
```
GET /api/v1/contents/{contentId}?hubProfileId={hubProfileId}
```

## 400 Bad Request on PUT /contents/{id}

**Cause:** Missing `hubProfileId` query parameter.

**Fix:**
```
PUT /api/v1/contents/{contentId}?hubProfileId={hubProfileId}
```

## 500 on POST /contents/{id}/{categoryName}/sections

**Cause:** Invalid `categoryName`. Must be `Content` or `Asset` — not the display name.

**Fix:**
```
POST /api/v1/contents/{contentId}/Content/sections
```

## PUT /contents wiped my data (channels, categories, cover gone)

**Cause:** PUT replaces ALL fields. Empty arrays delete existing data.

**Fix:** Always GET first, then include all existing fields:
```bash
# 1. GET current state
curl -s "https://gateway.srgplus.com/api/v1/contents/{id}?hubProfileId={hubId}" \
  -H "X-API-Key: srgplus_..."

# 2. Copy existing channels, categories, context
# 3. Modify only what you need
# 4. Send full object in PUT
```

## Link previews (icons) missing

**Cause:** Used `CustomLink` instead of `KnownLink`.

**Fix:** Use `$type: "KnownLink"` — server auto-fetches favicon:
```json
{"$type": "KnownLink", "title": "Google Drive", "url": "https://drive.google.com/..."}
```

## How to find hubProfileId

Get it from the workspace settings in the SRG+ app, or ask the SRG+ team.

## How to find channelId and categoryId

```bash
GET /api/v1/channels/{hubProfileId}?includeArchived=false
```
Returns all channels with their IDs and category IDs.
