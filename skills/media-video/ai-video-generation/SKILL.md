---
name: ai-video-generation
description: Generate AI video using Runway, Kling, Pika, HeyGen, and Sora. Covers text-to-video, image-to-video, avatar creation, lip sync, and AI video prompt engineering.
---

# AI Video Generation

Create AI-generated videos across platforms with optimized prompts and workflows.

---

## 1. Platform Selection

| Tool | Best for | Quality | Cost |
|---|---|---|---|
| **Runway Gen-3** | Cinematic clips, motion quality | ⭐⭐⭐⭐⭐ | $$$ |
| **Kling 1.6** | Long clips (2–10s), realism | ⭐⭐⭐⭐ | $$ |
| **Pika 1.5** | Quick iterations, text-to-video | ⭐⭐⭐ | $ |
| **HeyGen** | Avatar/talking head, lip sync | ⭐⭐⭐⭐ | $$ |
| **Sora** | High realism, complex scenes | ⭐⭐⭐⭐⭐ | $$$ |
| **Stable Video** | Open-source, local, image-to-video | ⭐⭐⭐ | Free |

---

## 2. Prompt Engineering for Video

### Structure: `[SUBJECT] + [ACTION] + [ENVIRONMENT] + [CAMERA] + [STYLE] + [LIGHTING]`

```
Good prompt:
"A senior engineer in a modern tech office, typing on a keyboard while 
reviewing code on multiple monitors, medium shot, slow dolly forward, 
cinematic lighting with blue-tinted screen glow, photorealistic"

Bad prompt:
"person working on computer"
```

### Motion keywords
- **Camera**: dolly in/out, pan left/right, tilt up/down, orbit, static
- **Speed**: slow motion, time-lapse, real-time, fast-forward
- **Transition**: fade in, dissolve, whip pan, match cut

### Style modifiers (append to any prompt)
```
cinematic, 35mm film, photorealistic, 4K, shallow depth of field,
golden hour lighting, neon-lit, volumetric fog, high contrast,
documentary style, news broadcast aesthetic, corporate clean
```

---

## 3. Runway Gen-3 Workflow

```bash
# API access
curl -X POST https://api.runwayml.com/v1/generation \
  -H "Authorization: Bearer $RUNWAY_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Your detailed prompt",
    "model": "gen3a_turbo",
    "duration": 5,
    "ratio": "1280:768",
    "watermark": false
  }'
```

**Settings:**
- Duration: 5s or 10s clips
- Ratio: 1280:768 (landscape), 768:1280 (vertical), 1:1 (square)
- Seed: fix seed to reproduce similar results

---

## 4. HeyGen Avatar (Talking Head / Lip Sync)

```
Workflow:
1. Upload your script (text or audio)
2. Select avatar (stock or custom upload)
3. Select voice (HeyGen voices or clone your own)
4. Generate and download

Custom avatar requirements:
- 5+ minute video of yourself talking
- Good lighting, neutral background
- Clear audio, no background noise
- Multiple camera angles optional but recommended

Lip sync to existing audio:
1. Upload video of speaker
2. Upload new audio track
3. HeyGen syncs lip movements to audio
```

---

## 5. Image-to-Video

Best for bringing static images to life:

```
Kling workflow:
1. Upload still image
2. Write motion prompt: "The person in the image slowly turns their head left, 
   background wind blowing through trees, cinematic"
3. Set duration: 5s standard
4. Generate → download

Tips:
- Use high-res source image (1080p+)
- Describe motion explicitly — don't assume the model knows what to move
- For product shots: "The [product] slowly rotates 360 degrees, studio lighting"
```

---

## 6. AI Video for Content Production

### B-Roll generation
```
Use when: You need generic B-roll to cover a talking head segment
Prompt template: "[Topic] concept visualization, abstract, [color scheme], 
loop-friendly, no text, no faces"
```

### Product demos
```
"[Product name] on a clean white surface, camera slowly orbiting, 
studio lighting, photorealistic product photography aesthetic"
```

### Data visualization
```
Use Remotion (code-based) for data animations — more reliable than AI for charts
Use AI generation for: background environments, abstract transitions, mood pieces
```

---

## 7. Prompt Iteration Workflow

1. Start with minimal prompt → generate test clip
2. Identify what's wrong (motion, lighting, style, subject)
3. Add specific descriptors for the failing element
4. Fix seed to isolate variable changes
5. Once happy with composition, generate multiple variations (different seeds)
6. Combine best clips in editing software

**Rule:** Generate 3–5 variations per scene. AI video is non-deterministic — the first result is rarely the best.
