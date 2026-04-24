---
name: fal-ai-media
description: Generate images, videos, and audio using fal.ai models via MCP server.
---

# fal.ai Media Generation

Generate images, videos, and audio using fal.ai models through the fal.ai MCP server.

---

## 1. MCP Setup

Add to `~/.claude.json`:
```json
"fal-ai": {
  "command": "npx",
  "args": ["-y", "fal-ai-mcp-server"],
  "env": { "FAL_KEY": "YOUR_FAL_KEY_HERE" }
}
```

Get an API key at [fal.ai](https://fal.ai).

**Available MCP tools:** `search`, `find`, `generate`, `result`, `status`, `cancel`, `estimate_cost`, `models`, `upload`

**Rule:** Always run `estimate_cost` before generating video or high-cost assets.

---

## 2. Image Generation

Fast iteration — Nano Banana 2:
```
generate(
  app_id: "fal-ai/nano-banana-2",
  input_data: { "prompt": "a futuristic cityscape, cyberpunk style", "image_size": "landscape_16_9", "num_images": 1, "seed": 42 }
)
```

Production quality — Nano Banana Pro:
```
generate(
  app_id: "fal-ai/nano-banana-pro",
  input_data: { "prompt": "product photo of headphones, marble surface, studio lighting", "image_size": "square", "guidance_scale": 7.5 }
)
```

Image editing (inpainting / style transfer):
```
upload(file_path: "/path/to/image.png")
generate(app_id: "fal-ai/nano-banana-2", input_data: { "prompt": "same scene in watercolor style", "image_url": "<uploaded_url>", "image_size": "landscape_16_9" })
```

Key params: `image_size` (`square`, `portrait_4_3`, `landscape_16_9`, `portrait_16_9`), `num_images` (1–4), `seed` (integer for reproducibility), `guidance_scale` (1–20).

---

## 3. Video Generation

Text-to-video — Seedance 1.0 Pro:
```
generate(app_id: "fal-ai/seedance-1-0-pro", input_data: { "prompt": "drone flyover of a mountain lake, cinematic", "duration": "5s", "aspect_ratio": "16:9", "seed": 42 })
```

With native audio — Kling v3 Pro:
```
generate(app_id: "fal-ai/kling-video/v3/pro", input_data: { "prompt": "ocean waves on a rocky coast", "duration": "5s", "aspect_ratio": "16:9" })
```

High visual quality with sound — Veo 3:
```
generate(app_id: "fal-ai/veo-3", input_data: { "prompt": "Tokyo street market at night, crowd noise", "aspect_ratio": "16:9" })
```

Image-to-video:
```
generate(app_id: "fal-ai/seedance-1-0-pro", input_data: { "prompt": "camera slowly zooms out, wind moves trees", "image_url": "<uploaded_url>", "duration": "5s" })
```

**Rule:** Start with Nano Banana 2 for prompt iteration. Switch to Pro models for final assets. Image-to-video produces more controlled results than pure text-to-video.

---

## 4. Audio Generation

Conversational speech — CSM-1B:
```
generate(app_id: "fal-ai/csm-1b", input_data: { "text": "Hello, welcome to the demo.", "speaker_id": 0 })
```

Video-to-audio — ThinkSound:
```
generate(app_id: "fal-ai/thinksound", input_data: { "video_url": "<video_url>", "prompt": "ambient forest sounds with birds" })
```

For professional voice synthesis, use ElevenLabs directly:
```python
import requests, os
resp = requests.post(
    f"https://api.elevenlabs.io/v1/text-to-speech/<voice_id>",
    headers={"xi-api-key": os.environ["ELEVENLABS_API_KEY"], "Content-Type": "application/json"},
    json={"text": "Your text", "model_id": "eleven_turbo_v2_5", "voice_settings": {"stability": 0.5, "similarity_boost": 0.75}},
)
with open("output.mp3", "wb") as f:
    f.write(resp.content)
```

---

## 5. Model Discovery and Cost

Find models for a task:
```
search(query: "text to video")
find(endpoint_ids: ["fal-ai/seedance-1-0-pro"])
models()
```

Estimate cost before running:
```
estimate_cost(estimate_type: "unit_price", endpoints: { "fal-ai/nano-banana-pro": { "unit_quantity": 1 } })
```
