---
name: videodb
description: Ingest, search, edit, and stream video server-side using the VideoDB SDK. Covers desktop capture, timeline editing, transcription, scene indexing, reframing, and generative media.
---

# VideoDB

Perception, memory, and actions for video — ingest files or live streams, index and search moments with timestamps, perform timeline edits, and generate media assets server-side.

---

## 1. Setup

```bash
pip install "videodb[capture]" python-dotenv
# On Linux if capture fails: pip install videodb python-dotenv
```

Set `VIDEO_DB_API_KEY` via export or `.env` file. Get a free key at [console.videodb.io](https://console.videodb.io).

Always load environment before connecting:
```python
from dotenv import load_dotenv
load_dotenv(".env")
import videodb
conn = videodb.connect()
coll = conn.get_collection()
```

**Rule:** Do not use ffmpeg, moviepy, or local encoding tools when VideoDB supports the operation. VideoDB handles server-side: trimming, combining clips, audio overlays, subtitles, text/image overlays, transcoding, resolution changes, aspect-ratio conversion, transcription, and media generation.

---

## 2. Ingest and Stream

```python
# URL or YouTube
video = coll.upload(url="https://www.youtube.com/watch?v=VIDEO_ID")

# Local file
video = coll.upload(file_path="/path/to/video.mp4")
```

---

## 3. Transcript and Spoken Word Search

```python
video.index_spoken_words(force=True)   # force=True skips error if already indexed
text = video.get_transcript_text()

from videodb.exceptions import InvalidRequestError
try:
    results = video.search("product demo")
    shots = results.get_shots()
    stream_url = results.compile()
except InvalidRequestError as e:
    if "No results found" in str(e):
        shots = []
    else:
        raise
```

---

## 4. Scene Search

```python
import re
from videodb import SearchType, IndexType, SceneExtractionType
from videodb.exceptions import InvalidRequestError

# index_scenes() has no force param — extract existing ID from error if already indexed
try:
    scene_index_id = video.index_scenes(
        extraction_type=SceneExtractionType.shot_based,
        prompt="Describe the visual content in this scene.",
    )
except Exception as e:
    match = re.search(r"id\s+([a-f0-9]+)", str(e))
    if match:
        scene_index_id = match.group(1)
    else:
        raise

try:
    results = video.search(
        query="person writing on a whiteboard",
        search_type=SearchType.semantic,
        index_type=IndexType.scene,
        scene_index_id=scene_index_id,
        score_threshold=0.3,
    )
    shots = results.get_shots()
    stream_url = results.compile()
except InvalidRequestError as e:
    shots = [] if "No results found" in str(e) else (_ for _ in ()).throw(e)
```

---

## 5. Timeline Editing

**Rule:** Always validate timestamps before building a timeline: `start >= 0`, `start < end`, `end <= video.length`.

```python
from videodb.timeline import Timeline
from videodb.asset import VideoAsset, TextAsset, TextStyle

timeline = Timeline(conn)
timeline.add_inline(VideoAsset(asset_id=video.id, start=10, end=30))
timeline.add_overlay(0, TextAsset(text="The End", duration=3, style=TextStyle(fontsize=36)))
stream_url = timeline.generate_stream()
```

---

## 6. Transcode and Reframe

```python
from videodb import TranscodeMode, VideoConfig, AudioConfig

job_id = conn.transcode(
    source="https://example.com/video.mp4",
    callback_url="https://example.com/webhook",
    mode=TranscodeMode.economy,
    video_config=VideoConfig(resolution=720, quality=23, aspect_ratio="16:9"),
    audio_config=AudioConfig(mute=False),
)

# Reframe — always use start/end to limit segment; use callback_url for full-length videos
reframed = video.reframe(start=0, end=60, target="vertical", mode=ReframeMode.smart)
# Presets: "vertical" (9:16), "square" (1:1), "landscape" (16:9)
```

**Rule:** `reframe()` is slow on long videos. Always limit with `start`/`end` or use `callback_url` for async processing.

---

## 7. Generative Media

```python
# Image
image = coll.generate_image(prompt="a sunset over mountains", aspect_ratio="16:9")

# Voice, music, SFX
voiceover = coll.generate_voice(text="Narration here", voice="alloy")
music = coll.generate_music(prompt="lo-fi background for coding vlog", duration=120)
sfx = coll.generate_sound_effect(prompt="subtle whoosh transition")
```

---

## 8. Desktop Capture (macOS only)

```bash
STATE_DIR="${VIDEODB_EVENTS_DIR:-$HOME/.local/state/videodb}"
VIDEODB_EVENTS_DIR="$STATE_DIR" python scripts/ws_listener.py --clear "$STATE_DIR" &
cat "$STATE_DIR/videodb_ws_id"
```

Events are written to `$STATE_DIR/videodb_events.jsonl`. Use `--clear` on each fresh capture run to avoid stale events leaking into the new session.

---

## 9. Common Pitfalls

| Scenario | Solution |
|----------|----------|
| Spoken word index already exists | Use `video.index_spoken_words(force=True)` |
| Scene index already exists | Extract existing `scene_index_id` from error with `re.search(r"id\s+([a-f0-9]+)", str(e))` |
| Search finds no matches | Catch `InvalidRequestError` and treat as empty results |
| Reframe times out | Use `start`/`end` to limit segment, or pass `callback_url` |
| Negative timestamps on Timeline | Validate `start >= 0` before creating `VideoAsset` |
