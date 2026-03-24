# Cover Image Upload Guide

Uploading a cover image is a **two-step process**: first get a signed URL, then upload the file.

## Step 1: Get Signed Upload URL

Include `cover` in your PUT request with image metadata:

```bash
curl -X PUT "https://gateway.srgplus.com/api/v1/contents/{contentId}?hubProfileId={hubProfileId}" \
  -H "X-API-Key: srgplus_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "KEEP EXISTING NAME",
    "cover": {
      "image": {
        "width": 1920,
        "height": 1080,
        "size": 500000,
        "extension": "jpg"
      },
      "generateSignedUrl": true
    },
    "context": [... KEEP EXISTING ...],
    "channels": [... KEEP EXISTING ...],
    "categories": [... KEEP EXISTING ...]
  }'
```

**Cover image fields:**
| Field | Type | Description |
|-------|------|-------------|
| `width` | int | Image width in pixels |
| `height` | int | Image height in pixels |
| `size` | long | File size in bytes |
| `extension` | string | File extension: `jpg`, `png`, `webp` |

**Max file size:** 25 MB

The response contains:
```json
{
  "id": "...",
  "coverSignedUrl": {
    "url": "https://...r2.cloudflarestorage.com/...?X-Amz-..."
  },
  "coverExtension": "jpg",
  "metadataHeaders": {
    "x-amz-meta-type": "ContentCover",
    "x-amz-meta-hub-profile-id": "...",
    "x-amz-meta-object-id": "..."
  }
}
```

## Step 2: Upload the File

Upload your image file directly to the signed URL with the required headers:

```bash
curl -X PUT "{coverSignedUrl.url}" \
  -H "Content-Type: image/jpeg" \
  -H "Content-Length: {exact file size in bytes}" \
  -H "x-amz-meta-type: ContentCover" \
  -H "x-amz-meta-hub-profile-id: {from metadataHeaders}" \
  -H "x-amz-meta-object-id: {from metadataHeaders}" \
  --data-binary @/path/to/your/image.jpg
```

**Important:**
- `Content-Type` must match the extension (`image/jpeg` for jpg, `image/png` for png)
- `Content-Length` must match the exact file size in bytes (same as `size` in Step 1)
- All `x-amz-meta-*` headers from `metadataHeaders` are **required**
- The signed URL **expires in 10 minutes** — upload promptly after getting it
- File size in Step 2 must match what you declared in Step 1

## Complete Example (bash)

```bash
API_KEY="srgplus_YOUR_KEY"
CONTENT_ID="your_content_id"
HUB_ID="your_hub_profile_id"
IMAGE_FILE="/path/to/cover.jpg"
BASE="https://gateway.srgplus.com"

# Get image dimensions and size
WIDTH=$(sips -g pixelWidth "$IMAGE_FILE" | tail -1 | awk '{print $2}')
HEIGHT=$(sips -g pixelHeight "$IMAGE_FILE" | tail -1 | awk '{print $2}')
SIZE=$(stat -f%z "$IMAGE_FILE")
EXT="${IMAGE_FILE##*.}"

echo "Image: ${WIDTH}x${HEIGHT}, ${SIZE} bytes, ${EXT}"

# Step 1: GET existing content first (MANDATORY)
EXISTING=$(curl -s "$BASE/api/v1/contents/$CONTENT_ID?hubProfileId=$HUB_ID" \
  -H "X-API-Key: $API_KEY")

CONTENT_NAME=$(echo "$EXISTING" | python3 -c "import sys,json; print(json.load(sys.stdin).get('name',''))")

# Step 2: PUT with cover metadata (keep all existing fields!)
RESPONSE=$(curl -s -X PUT "$BASE/api/v1/contents/$CONTENT_ID?hubProfileId=$HUB_ID" \
  -H "X-API-Key: $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"name\": \"$CONTENT_NAME\",
    \"cover\": {
      \"image\": {
        \"width\": $WIDTH,
        \"height\": $HEIGHT,
        \"size\": $SIZE,
        \"extension\": \"$EXT\"
      },
      \"generateSignedUrl\": true
    }
  }")

# Extract signed URL and metadata headers
SIGNED_URL=$(echo "$RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['coverSignedUrl']['url'])")
META_TYPE=$(echo "$RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['metadataHeaders']['x-amz-meta-type'])")
META_HUB=$(echo "$RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['metadataHeaders']['x-amz-meta-hub-profile-id'])")
META_OBJ=$(echo "$RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['metadataHeaders']['x-amz-meta-object-id'])")

# Step 3: Upload file to signed URL
curl -X PUT "$SIGNED_URL" \
  -H "Content-Type: image/${EXT}" \
  -H "Content-Length: $SIZE" \
  -H "x-amz-meta-type: $META_TYPE" \
  -H "x-amz-meta-hub-profile-id: $META_HUB" \
  -H "x-amz-meta-object-id: $META_OBJ" \
  --data-binary "@$IMAGE_FILE"

echo "Cover uploaded!"
```

## Preserving Cover on Updates

When updating content with PUT, to **keep the existing cover** without changing it:

```json
{
  "name": "...",
  "cover": {
    "image": null,
    "generateSignedUrl": false
  }
}
```

**Both fields are required:**
- `"image": null` — required field (cannot be omitted)
- `"generateSignedUrl": false` — tells the server "don't change the cover"

**Without `"image": null` you get 400 Bad Request!**

**This is the key to safe updates!** Always include this in EVERY PUT when you don't want to change the cover.
