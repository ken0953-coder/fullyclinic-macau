---
name: agnes-video-gen
description: Generate videos via Agnes Video V2.0 API. Free, cinematic quality.
version: 1.0.0
author: Hermes
platforms: [windows, macos, linux]
metadata:
  hermes:
    tags: [video-generation, agnes, apihub, async, text-to-video, image-to-video]
---

# Agnes Video Generation

Generate videos via Agnes Video V2.0 API at `https://apihub.agnes-ai.com/v1/videos`.

## Trigger Conditions
- User asks to generate videos from text or images
- User asks for video creation, animation, motion effects
- User provides a prompt that needs video output

## API Details

**Base URL:** `https://apihub.agnes-ai.com`

**Auth:** Same API key as chat completions (`sk-j6wDNDZs6PX2PminVF3YO3Sswrttlb4GP2p5kMdGm2EMYGZV`)

**Model:** `agnes-video-v2.0`

**Price:** $0/second (currently free)

### Workflow (Async)
Video generation is asynchronous — two-step process:
1. **Create task** → returns `video_id` and `task_id`
2. **Poll result** → wait for completion, download video

### Step 1: Create Video Task

**Endpoint:** `POST https://apihub.agnes-ai.com/v1/videos`

**Request Headers:**
```
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

**Required Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| model | string | `agnes-video-v2.0` |
| prompt | string | Video generation prompt |

**Optional Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| image | string[] | Input images for img2video (public URLs) |
| size | string | Resolution, e.g. `1024x576`, `576x1024` |
| duration | number | Video length in seconds (default: varies) |
| return_url | boolean | Return video URL when ready |

**Response:**
```json
{
  "video_id": "xxx",
  "task_id": "xxx",
  "status": "pending"
}
```

### Step 2: Poll Result

**Endpoint (Recommended):** `GET https://apihub.agnes-ai.com/agnesapi?video_id=<VIDEO_ID>`

**Endpoint (Legacy):** `GET https://apihub.agnes-ai.com/v1/videos/<TASK_ID>`

**Response:**
```json
{
  "video_id": "xxx",
  "status": "completed",
  "video_url": "https://...",
  "thumbnail_url": "https://..."
}
```

**Status Values:**
- `pending` — Task created, waiting to start
- `processing` — Video generating
- `completed` — Ready to download
- `failed` — Generation failed

## Workflow Examples

### Text-to-Video
```bash
# Step 1: Create task
TASK_RESPONSE=$(curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "A basketball player dribbling on an indoor court, dynamic camera movement, cinematic lighting",
    "size": "1024x576"
  }')

VIDEO_ID=$(echo "$TASK_RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['video_id'])")
echo "Task created: video_id=$VIDEO_ID"

# Step 2: Poll until complete (loop with sleep)
while true; do
  RESULT=$(curl -s "https://apihub.agnes-ai.com/agnesapi?video_id=$VIDEO_ID" \
    -H "Authorization: Bearer $AGNES_API_KEY")
  
  STATUS=$(echo "$RESULT" | python3 -c "import sys,json; print(json.load(sys.stdin).get('status',''))")
  
  if [ "$STATUS" = "completed" ]; then
    echo "Video ready!"
    echo "$RESULT" | python3 -c "import sys,json; print(json.load(sys.stdin)['video_url'])"
    break
  elif [ "$STATUS" = "failed" ]; then
    echo "Generation failed"
    exit 1
  fi
  
  echo "Status: $STATUS, waiting..."
  sleep 5
done
```

### Image-to-Video
```bash
# Step 1: Create task with image reference (MUST be public URL, NOT base64)
TASK_RESPONSE=$(curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "Add dynamic motion, camera pans slowly, basketball player dribbling",
    "image": ["https://example.com/basketball_player.png"],
    "size": "1024x576"
  }')

VIDEO_ID=$(echo "$TASK_RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['video_id'])")
```

**CRITICAL:** The `image` parameter accepts ONLY publicly accessible URLs. Base64 data URIs will return HTTP 400 error. For local images, upload to a public host first or use text-to-video with detailed description instead.

### Keyframe Animation
```bash
TASK_RESPONSE=$(curl -s -X POST "https://apihub.agnes-ai.com/v1/videos" \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-video-v2.0",
    "prompt": "Smooth transition between scenes",
    "image": [
      "https://example.com/frame1.png",
      "https://example.com/frame2.png",
      "https://example.com/frame3.png"
    ]
  }')
```

## Prompt Engineering Tips

### Motion Keywords
- `camera pans left/right/up/down`
- `zoom in/out`
- `slow motion`
- `dynamic movement`
- `smooth transition`
- `tracking shot`

### Scene Descriptions
- Be specific about subject action
- Describe camera movement
- Specify lighting conditions
- Include atmosphere/mood

### Example Prompts
```
"A basketball player dribbling fast down the court, dynamic low-angle tracking shot, dramatic arena lighting, sweat flying, crowd blurred in background, cinematic sports photography style"

"Product rotating slowly on white background, smooth camera orbit, soft studio lighting, commercial advertisement quality"

"Sunset over ocean with waves crashing, slow zoom out, golden hour lighting, peaceful atmosphere"
```

## Size Recommendations
| Use Case | Size | Aspect Ratio |
|----------|------|--------------|
| Landscape / YouTube | `1024x576` | 16:9 |
| Portrait / TikTok | `576x1024` | 9:16 |
| Square / Instagram | `768x768` | 1:1 |

## Pitfalls
- Video generation is ASYNC — must poll for results
- Polling interval: 5-10 seconds (don't spam)
- **Images must be PUBLIC URLs** — base64 data URIs cause HTTP 400 errors. Use text-to-video with detailed description as fallback.
- For img2video, upload image to a public host first (imgbb, etc.)
- Video length affects cost (currently free but may change)
- Same API key works for all Agnes services (chat, image, video)
- Status field name changed from `status` to `internal_status` in some responses — check both

## Face Preservation Limitation (CRITICAL)
AI video generation tools (including Agnes Video) **DO NOT preserve facial features** from reference photos. They regenerate faces based on the prompt text. If facial likeness is required:
1. Use text-to-video with extremely detailed facial description
2. OR use a separate FaceSwap pipeline: generate base image → extract face → swap face → convert to video
3. See `references/face-swap-workflow.md` for the complete FaceSwap + video pipeline

## Hermes Tool Mapping
- Execute API: `terminal` with `curl`
- Parse JSON: `python3 -c "import json"` or `jq`
- Download video: `wget` or `curl -fSL`
- Wait/poll: Shell loop with `sleep`
- Display result: Open video in browser or play locally

## File Naming Convention
Generated videos saved to: `~/.hermes/downloads/video_<timestamp>.mp4`
