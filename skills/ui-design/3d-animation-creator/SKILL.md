---
name: 3d-animation-creator
description: Takes a video file and builds a scroll-driven website where the video plays forward/backward based on scroll position — Apple-style product launch effect. FFmpeg extracts frames; vanilla JS canvas renders them frame-accurately. Includes starscape, loader, annotation snap-stop cards, specs count-up, glass-morphism features, and navbar pill transform.
---

# 3D Animation Creator

Take a video → extract frames → build a scroll-controlled canvas website.

**Rule:** Complete the interview (Step 0) before touching any code or frames. The entire site is built from those answers.

---

## 1. Step 0: Interview (Mandatory)

Ask in one message:

1. Brand name + logo file (SVG or PNG preferred)
2. Accent color (hex, or describe and suggest options)
3. Background color (dark works best for this effect)
4. Overall vibe — premium tech / luxury / minimal / bold
5. Content source — URL to `WebFetch`, or paste content directly

Optional (include only if user opts in): testimonials section · confetti burst on CTA · Three.js card scanner.

**Prerequisites:** FFmpeg installed (`brew install ffmpeg`). Video 3–10s ideal. **First frame must be white background** — if not, ask for a re-export or separate white hero image.

---

## 2. Design System

| Token | Value |
|-------|-------|
| Heading font | Space Grotesk |
| Body font | Archivo |
| Mono font | JetBrains Mono |
| Cards | `backdrop-filter: blur(20px)`, `border-radius: 20px`, semi-transparent bg |
| Buttons | Primary: accent bg + glow; Secondary: transparent + border |
| Background | Floating orbs (accent tones, blurred), grid overlay, animated starscape |

Text: dark bg → white primary + muted secondary. Light bg → dark primary + muted secondary.

---

## 3. Build Process

**Step 1 — Analyze video:**
```bash
ffprobe -v quiet -print_format json -show_streams -show_format "{VIDEO_PATH}"
```
Extract duration, fps, resolution. Target 60–150 total frames.

**Step 2 — Extract frames:**
```bash
ffmpeg -i "{VIDEO_PATH}" -vf "fps={TARGET_FPS},scale=1920:-2" -q:v 2 "{OUTPUT_DIR}/frames/frame_%04d.jpg"
```
Use JPEG (`-q:v 2`), not PNG. Target <100KB per frame.

**Step 3 — Site sections (top to bottom):**

Starscape → Loader → Scroll progress bar → Navbar → Hero → **Scroll animation** → Specs → Features → CTA → Testimonials* → Card scanner* → Footer

For complete CSS/JS implementation of every section, read `references/sections-guide.md`.

**Step 4 — Key implementation patterns:**

| Pattern | Implementation |
|---------|---------------|
| Canvas sizing | `canvas.width = innerWidth * devicePixelRatio`; style via CSS pixels |
| Frame rendering | Desktop: cover-fit. Mobile: zoomed contain-fit, centered |
| Annotation snap-stop | Detect scroll progress entering snap zone → scroll to exact position → lock `overflow` 600ms → release |
| Navbar transform | Full-width → centered pill (`max-width: 820px`) + glass-morphism on scroll |
| Count-up | `IntersectionObserver` trigger, `easeOutExpo`, staggered 200ms, accent glow pulse |
| Starscape | ~180 stars, random drift + twinkle phase, fixed canvas, `opacity: 0.6` |

**Step 5 — Serve:**
```bash
cd "{OUTPUT_DIR}" && python3 -m http.server 8080
```

**Rule:** Never use placeholder text. All copy comes from the interview answers or fetched URL.

---

## 4. Mobile Adaptations

| Element | Mobile |
|---------|--------|
| Annotation cards | Title only (single line), `bottom: 1.5vh`, hide paragraphs and stats |
| Scroll animation height | `350vh` desktop → `300vh` tablet → `250vh` phone |
| Navbar | Logo + pill only, hide nav links |
| Feature cards | Single column |
| Specs | 2×2 grid |
| Testimonials | Touch-scrollable, snap to card edges |

---

## 5. Troubleshooting

| Issue | Fix |
|-------|-----|
| Choppy animation | Reduce frame count, verify JPEG not PNG, check sizes <100KB each |
| Canvas blurry | Apply `devicePixelRatio` to canvas `width`/`height` |
| Scroll too fast / slow | Adjust `.scroll-animation` height (200vh = fast, 800vh = cinematic) |
| Snap-stop feels jarring | Reduce `HOLD_DURATION` to 400ms or increase `SNAP_ZONE` |
| First frame not white | Ask user to re-export video with white opening frame |
| Frames won't load | Ensure local server is running — cannot load via `file://` protocol |
