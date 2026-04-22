---
name: video-editing
description: Edit, cut, and augment real footage through a layered AI-assisted pipeline using FFmpeg, Remotion, ElevenLabs, and fal.ai.
---

# Video Editing

AI-assisted editing for real footage — cutting, structuring, and augmenting existing video, not generating from prompts.

**Core thesis:** AI video editing compresses, structures, and augments real footage. The value is not generation — it is compression.

---

## 1. The Pipeline

```
Raw footage → Claude/Codex → FFmpeg → Remotion → ElevenLabs/fal.ai → Descript/CapCut
```

Each layer has a specific job. Do not skip layers. Do not make one tool do everything.

---

## 2. Layer 1 — Capture

Collect source material:
- **Screen Studio**: polished screen recordings for app demos and coding sessions
- **Raw camera footage**: vlogs, interviews, events
- **VideoDB desktop capture**: session recording with real-time context (see `videodb` skill)

---

## 3. Layer 2 — Organization (Claude / Codex)

Use Claude Code to transcribe, label, and plan structure before touching any video file:

```
"Here's the transcript of a 4-hour recording. Identify the 8 strongest segments
for a 24-minute vlog. Give me FFmpeg cut commands for each segment."
```

Outputs: edit decision list with timestamps, FFmpeg commands, Remotion scaffold.

**Rule:** Get the story right in Layer 2 before touching anything visual.

---

## 4. Layer 3 — Deterministic Cuts (FFmpeg)

Extract a segment:
```bash
ffmpeg -i raw.mp4 -ss 00:12:30 -to 00:15:45 -c copy segment_01.mp4
```

Batch cut from edit decision list (CSV: start,end,label):
```bash
while IFS=, read -r start end label; do
  ffmpeg -i raw.mp4 -ss "$start" -to "$end" -c copy "segments/${label}.mp4"
done < cuts.txt
```

Concatenate segments:
```bash
for f in segments/*.mp4; do echo "file '$f'"; done > concat.txt
ffmpeg -f concat -safe 0 -i concat.txt -c copy assembled.mp4
```

Normalize audio levels:
```bash
ffmpeg -i segment.mp4 -af loudnorm=I=-16:TP=-1.5:LRA=11 -c:v copy normalized.mp4
```

Reframe for social platforms:
```bash
# 16:9 to 9:16 (TikTok/Reels)
ffmpeg -i input.mp4 -vf "crop=ih*9/16:ih,scale=1080:1920" vertical.mp4
# 16:9 to 1:1 (Instagram)
ffmpeg -i input.mp4 -vf "crop=ih:ih,scale=1080:1080" square.mp4
```

Scene change and silence detection:
```bash
ffmpeg -i input.mp4 -vf "select='gt(scene,0.3)',showinfo" -vsync vfr -f null - 2>&1 | grep showinfo
ffmpeg -i input.mp4 -af silencedetect=noise=-30dB:d=2 -f null - 2>&1 | grep silence
```

---

## 5. Layer 4 — Programmable Composition (Remotion)

Use Remotion for overlays, motion graphics, data visualizations, reusable templates, and composable scenes.

```tsx
import { AbsoluteFill, Sequence, Video } from "remotion";

export const VlogComposition: React.FC = () => (
  <AbsoluteFill>
    <Sequence from={0} durationInFrames={300}>
      <Video src="/segments/intro.mp4" />
    </Sequence>
    <Sequence from={30} durationInFrames={90}>
      <AbsoluteFill style={{ justifyContent: "center", alignItems: "center" }}>
        <h1 style={{ fontSize: 72, color: "white", textShadow: "2px 2px 8px rgba(0,0,0,0.8)" }}>Title</h1>
      </AbsoluteFill>
    </Sequence>
  </AbsoluteFill>
);
```

```bash
npx remotion render src/index.ts VlogComposition output.mp4
```

**Rule:** If you will do it more than once, make it a Remotion component.

---

## 6. Layer 5 — Generated Assets (ElevenLabs / fal.ai)

Generate only what you need — not the whole video.

Voiceover with ElevenLabs:
```python
import requests, os
resp = requests.post(
    f"https://api.elevenlabs.io/v1/text-to-speech/{voice_id}",
    headers={"xi-api-key": os.environ["ELEVENLABS_API_KEY"], "Content-Type": "application/json"},
    json={"text": "Narration here", "model_id": "eleven_turbo_v2_5", "voice_settings": {"stability": 0.5, "similarity_boost": 0.75}},
)
with open("voiceover.mp3", "wb") as f:
    f.write(resp.content)
```

Generated thumbnail or b-roll via fal.ai:
```
generate(app_id: "fal-ai/nano-banana-pro", input_data: { "prompt": "tech vlog thumbnail, dark background, code on screen", "image_size": "landscape_16_9" })
```

---

## 7. Layer 6 — Final Polish (Descript / CapCut)

Human layer: pacing, captions, color grading, audio mix, platform export. AI clears the repetitive work. You make the final creative calls.

| Platform | Aspect Ratio | Resolution |
|----------|-------------|------------|
| YouTube | 16:9 | 1920×1080 |
| TikTok / Reels | 9:16 | 1080×1920 |
| Instagram Feed | 1:1 | 1080×1080 |
| X / Twitter | 16:9 or 1:1 | 1280×720 or 720×720 |
