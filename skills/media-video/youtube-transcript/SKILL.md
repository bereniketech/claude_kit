---
name: youtube-transcript
description: Fetch YouTube video transcripts, summarize key points, extract insights, and repurpose into blog posts, social clips, newsletters, or new video scripts.
---

# YouTube Transcript

Extract and repurpose YouTube transcript content into multiple formats.

---

## 1. Fetch Transcript

```python
# Using youtube-transcript-api (pip install youtube-transcript-api)
from youtube_transcript_api import YouTubeTranscriptApi

def get_transcript(video_url: str) -> str:
    video_id = video_url.split("v=")[-1].split("&")[0]
    transcript = YouTubeTranscriptApi.get_transcript(video_id)
    return " ".join([entry['text'] for entry in transcript])

# With timestamps
def get_timestamped(video_id: str) -> list[dict]:
    return YouTubeTranscriptApi.get_transcript(video_id)
    # Returns: [{"text": "...", "start": 0.0, "duration": 2.5}, ...]
```

```bash
# CLI alternative
pip install youtube-transcript-api
python -c "from youtube_transcript_api import YouTubeTranscriptApi; print(' '.join([x['text'] for x in YouTubeTranscriptApi.get_transcript('VIDEO_ID')]))"
```

If transcript unavailable: use `yt-dlp --write-auto-sub --skip-download URL` for auto-generated captions.

---

## 2. Summarize

Structure the summary as:

```
## Video Summary: [Title]
**Source**: [URL]
**Duration**: [X min]

### Key Thesis
[One sentence: the main argument or takeaway]

### Main Points
1. [Point 1 with timestamp 0:00]
2. [Point 2 with timestamp X:XX]
3. [Point 3 with timestamp X:XX]

### Quotable Moments
> "[Direct quote worth sharing]" — [timestamp]

### Key Data / Stats
- [Stat 1]
- [Stat 2]

### Action Items
- [ ] [Actionable takeaway 1]
- [ ] [Actionable takeaway 2]
```

---

## 3. Repurpose Outputs

### → Blog Post
```
Headline: Use the video title formula but optimize for search intent
Structure:
  Intro (150 words): hook + problem + promise
  H2 sections: one per main point from transcript
  Examples: pull directly from transcript, add context
  Conclusion: recap + CTA (watch the video, subscribe, download)
```

### → LinkedIn Post
```
Opening line: strongest quote or stat from transcript
Body: 3 bullet points of insight (short, specific)
CTA: "Full breakdown in comments" or "Watch the full video: [link]"
No images needed: LinkedIn text posts outperform image posts for reach
```

### → Twitter/X Thread
```
Tweet 1: Bold claim (the thesis)
Tweet 2–N: One insight per tweet, numbered (1/ 2/ 3/)
Last tweet: "Full video: [link] — follow for more [topic]"
```

### → Newsletter Section
```
Subject: [Key insight from video] — [your newsletter name]
Body:
  "I watched [creator]'s video on [topic]. Here's what stood out:"
  [3 bullet points]
  [1 quote]
  "Worth watching: [link]"
```

### → New Video Script
Extract the best segment (most quotable, most surprising) as a "reaction" or "commentary" video:
```
Hook: Play the original clip (3–5 seconds)
Analysis: "Here's why this matters..."
Your take: What you agree/disagree with
Expansion: One thing the original video missed
CTA: "Comment below — do you agree?"
```

---

## 4. Quality Rules

- **Quote accurately**: use exact transcript text when quoting, don't paraphrase as a quote
- **Add timestamps**: every claim attributed to transcript should include `[X:XX]`
- **Credit source**: always include original video URL and creator name
- **Check transcript errors**: auto-generated captions have homophones and proper-noun errors — verify key terms
- **Don't over-summarize**: preserve the specific examples and numbers that make content valuable
