---
name: website-intelligence
description: Research-driven competitive intelligence + website builder. Scrapes client site, scores top 5 competitors, produces a PDF-ready HTML report, then builds a GSAP scroll-animated website from an approved brief. Uses Firecrawl MCP (WebFetch fallback).
---

# Website Intelligence

Research a niche, analyze competitors, produce a client-ready report, then build a premium scroll-animated website grounded in real market data — not guesswork.

**Rule:** Complete every phase before starting the next. Each phase produces a file deliverable.

---

## 1. Setup: Firecrawl

Check for `mcp__firecrawl__scrape`, `mcp__firecrawl__map`, `mcp__firecrawl__search` in available tools. If missing, ask for a Firecrawl API key and guide the user to add it as an MCP server (`command: npx -y firecrawl-mcp`, `env: FIRECRAWL_API_KEY`). Fall back to `WebFetch` if MCP is unavailable — fetch individual URLs manually instead of using map/search.

---

## 2. Phase 1 — Client Brand Extraction

Scrape the client's existing site. Extract:

| Asset | Where to find it |
|-------|-----------------|
| Logo | `<img>` in header/nav, `og:image` meta tag |
| Brand colors | CSS custom properties, inline styles, stylesheets |
| Fonts | `font-family` declarations, Google Fonts / Adobe Fonts links |
| Tone | Homepage copy analysis — formal / casual / playful / authoritative |
| Messaging | Headline, tagline, value proposition |
| Site structure | `firecrawl__map` to discover all URLs |

Save → `research/01-client-brand.md`. See `examples/sample-brand-snapshot.md` for expected format.

**Rule:** Always scrape the client's existing site first — never start from scratch when brand assets exist online.

---

## 3. Phase 2 — Competitive Niche Analysis

Find top 10 competitors via Firecrawl search. Score each 1–10 across: search visibility, review quality, visual design, mobile quality, content depth, social proof, CTA clarity, page speed.

Deep-scrape the top 5. Extract per site: hex color palette, typography, headline formula, CTA copy, nav structure, conversion strategy.

Identify 3–5 patterns that separate top-scoring from bottom-scoring sites.

Save → `research/02-competitor-analysis.md` (include comparison table + "Patterns of the Top 10%" section).

---

## 4. Phase 3 — Competitive Analysis Report (HTML)

Build a PDF-ready HTML report saved as `competitive-analysis.html`. Use `references/process-overview.html` as the exact design reference (warm paper tones, Instrument Serif + DM Sans, terracotta accent).

Must include: cover, executive summary, competitor profiles (color swatches + scores), side-by-side comparison table, SEO landscape, top-10% patterns, recommended design direction.

Design constraints: `@media print` rules, A4 format, no JavaScript, responsive at 640px breakpoint.

---

## 5. Phase 4 — Build Brief + Hard Stop

Combine brand + competitor data into `research/03-build-brief.md`:

- **Design direction** — refined palette, typography pairing, animation recommendations, what to avoid
- **Site architecture** — page list, nav structure, CTA strategy per page
- **Content framework** — 3 headline options, section-by-section copy direction, target SEO keywords
- **Conversion playbook** — lead capture method, social proof placement, trust signal checklist

**Rule:** Do not start the build until the user explicitly approves the brief. Present it and ask "Ready to build?" This checkpoint exists because major direction changes are expensive once the build starts.

---

## 6. Phase 5 — Website Build + Quality Audit

HTML + CSS + JS (no frameworks). GSAP + ScrollTrigger for all scroll animations. Mobile-first responsive.

Every page: `<title>`, unique meta description, one H1, logical H2/H3, alt text on all images, schema markup for the business type.

Include a clearly marked placeholder in the hero: `<!-- 3D SCROLL ASSET HERE: [width]×[height]px -->` — the user will drop in scroll-stop video content separately.

Performance targets: Lighthouse 90+, lazy-load images, `prefers-reduced-motion` support, no render-blocking resources.

Run the full checklist before handoff. Save → `research/04-quality-audit.md`. Fix all failures before declaring complete.

**Output structure:**
```
project/
├── research/
│   ├── 01-client-brand.md
│   ├── 02-competitor-analysis.md
│   ├── 03-build-brief.md
│   └── 04-quality-audit.md
├── competitive-analysis.html
├── site/index.html · css/ · js/ · assets/
└── README.md
```
