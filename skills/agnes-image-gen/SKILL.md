---
name: agnes-image-gen
description: Generate images via Agnes Image 2.1 Flash API. Free, high-quality.
version: 1.0.0
author: Hermes
platforms: [windows, macos, linux]
metadata:
  hermes:
    tags: [image-generation, agnes, apihub, flux, text-to-image]
---

# Agnes Image Generation

Generate images via Agnes Image 2.1 Flash API at `https://apihub.agnes-ai.com/v1/images/generations`.

## Trigger Conditions
- User asks to generate images from text
- User asks for image creation, illustration, concept art
- User provides a prompt that needs visual output
- User asks about "生圖", "image generation", "create an image"

## API Details

**Endpoint:** `POST https://apihub.agnes-ai.com/v1/images/generations`

**Auth:** Bearer token (same as chat completions)

**Model:** `agnes-image-2.1-flash`

**Required Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| model | string | `agnes-image-2.1-flash` |
| prompt | string | Image generation prompt (detailed descriptions work best) |
| size | string | Output size, e.g. `1024x768`, `768x1024`, `1024x1024` |

**Optional Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| image | string[] | Input images for img2img (URL or Base64) |
| return_base64 | boolean | Return base64 instead of URL |
| extra_body.response_format | string | `url` or `b64_json` |

**Response Format:**
```json
{
  "data": [{
    "url": "https://platform-outputs.agnes-ai.space/images/t2i/xxxx.png",
    "b64_json": null
  }]
}
```

## Workflow

### Text-to-Image (Basic)
```bash
curl -s -X POST "https://apihub.agnes-ai.com/v1/images/generations" \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "your detailed prompt here",
    "size": "1024x768"
  }'
```

### Download Generated Image
```bash
# Extract URL from response, then download:
wget -O output.png "$IMAGE_URL"
# or
curl -fSL "$IMAGE_URL" -o output.png
```

### Image-to-Image
```bash
curl -s -X POST "https://apihub.agnes-ai.com/v1/images/generations" \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-image-2.1-flash",
    "prompt": "edit instructions",
    "size": "1024x768",
    "image": ["https://example.com/input.jpg"]
  }'
```

## Size Recommendations
| Use Case | Size | Aspect Ratio |
|----------|------|--------------|
| Landscape / banner | `1024x768` | 4:3 |
| Portrait / mobile | `768x1024` | 3:4 |
| Square / social media | `1024x1024` | 1:1 |
| Wide / wallpaper | `1280x720` | 16:9 |
| Storyboard / cinematic | `1920x1080` | 16:9 |

## Prompt Engineering Tips
- **Be specific**: Include subject, action, setting, style, lighting, mood
- **Style keywords**: `anime style`, `photorealistic`, `oil painting`, `watercolor`, `3D render`, `concept art`
- **Quality boosters**: `highly detailed`, `cinematic lighting`, `sharp focus`, `masterpiece`
- **Composition**: `close-up`, `wide angle`, `low angle`, `rule of thirds`
- **Language**: English prompts work better (model trained primarily on English)

## Example Prompts
```
# Anime basketball player
"A dynamic anime-style basketball player dribbling on an indoor court, 
sweat flying, intense expression, cel-shaded, dramatic lighting, 
motion blur background, sports anime aesthetic"

# Product visualization
"Professional product photography of a sleek wireless headphone on 
a minimalist white background, soft studio lighting, commercial style"

# Concept art
"Epic fantasy landscape with floating islands, waterfalls, ancient 
temples, golden hour lighting, highly detailed, digital painting"
```

## Pitfalls
- **CRITICAL: img2img does NOT preserve facial features** — passing a reference photo via `image` parameter only provides composition/style guidance, NOT identity. The model regenerates the face based on the prompt text. If facial likeness is required, use text-to-image with detailed facial description instead.
- API key is same as chat completions — no separate key needed
- Response format defaults to URL; use `return_base64: true` if you need base64
- Images are free ($0/z) but rate limits may apply
- Prompt should be under 1000 characters for optimal results
- For img2img, input images must be publicly accessible URLs or valid Base64
- Model name is case-sensitive: use `agnes-image-2.1-flash` (not `Agnes`)

## Hermes Tool Mapping
- Execute API: `terminal` with `curl`
- Download images: `terminal` with `wget` or `curl -fSL`
- Display result: `vision_analyze` to verify generated image
- Save output: `write_file` for base64 decoding if needed
