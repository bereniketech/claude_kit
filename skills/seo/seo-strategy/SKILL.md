---
name: seo-strategy
description: Two-mode SEO skill. Mode 1 — article/page optimization: research top competitors, rewrite content, deliver a 3-tab HTML report (revised / changelog / original). Mode 2 — full site audit: crawl multiple pages, identify site-wide issues, deliver a prioritized HTML strategy report.
---

# SEO Strategy

Two modes. Detect from input, then follow the matching workflow.

| Signal | Mode |
|--------|------|
| Article text, single URL + "optimize / improve / rewrite / check" | Mode 1 |
| Root domain + "audit / review / strategy", multi-page request | Mode 2 |
| Just article text with no explicit request | Mode 1 |
| Just a domain with no specific request | Mode 2 |

---

## Mode 1: Article / Page Optimization

**Deliverable:** 3-tab HTML report — Revised Article · Changelog · Original.

**Rule:** Run competitor research before rewriting. Every change must be data-informed, not guesswork.

### Step 1 — Intake (mandatory, single message)

Ask:
1. Target keyword(s) — primary + secondary (or "suggest for me")
2. Target audience
3. Search intent — informational / transactional / commercial investigation / navigational / "determine for me"
4. Geographic focus
5. LSI/semantic keywords to include (or "research for me")
6. Competitor URLs to outrank (optional — will find top results automatically either way)
7. Constraints — word count, tone, topics to avoid, mandatory sections or CTAs

If user says "suggest for me": infer from article, confirm before proceeding. **Never override user-provided keywords** — the user knows their business and audience.

### Step 2 — Competitor Research

`WebSearch` for the target keyword. Analyze top 5–10 results. For each capture:

| Signal | What to extract |
|--------|----------------|
| Title tag | Exact wording, keyword position |
| URL slug | Keyword inclusion |
| H1 + H2 list | Content structure and subtopics covered |
| Word count | Content depth benchmark |
| LSI terms | Semantic keywords appearing in 3+ top results |
| Schema type | Structured data used |

Identify: average word count, content gaps in the user's article, required LSI terms.

### Step 3 — Optimization Plan

Map each planned change to impact level:

| Category | What to address |
|----------|----------------|
| Title + H1 | Target keyword in first 60 chars |
| Meta description | 150–160 chars, keyword + CTA |
| Heading structure | One H1, logical H2/H3, keyword-aligned subheadings |
| Content gaps | Subtopics top competitors cover that user's article misses |
| LSI terms | Weave in semantic keywords naturally |
| Word count | Match or exceed top competitor average |
| Internal links | 2–3 anchor text opportunities |
| Schema | Recommend appropriate markup type |

### Step 4 — Rewrite

Produce the fully revised article. Match search intent. Use the competitor word count range. Integrate all identified LSI terms, fill content gaps, use the title formula winning results use.

### Step 5 — HTML Report

Build a 3-tab HTML page with tabbed navigation:
- **Tab 1: Revised Article** — clean, publish-ready
- **Tab 2: Changelog** — every change grouped by category (Title / Structure / Content / LSI / Technical), each with impact badge (High / Medium / Low)
- **Tab 3: Original** — unmodified for comparison

Write to `seo-report.html`. Open in browser after writing.

---

## Mode 2: Full Website Audit

**Deliverable:** Prioritized HTML strategy report.

### Step 1 — Crawl

Use `mcp__firecrawl__map` to discover all URLs (fall back to sequential `WebFetch` if unavailable). Scrape up to 20 pages. Prioritize: homepage, category/pillar pages, high-traffic blog posts.

### Step 2 — Per-Page Analysis

For each page capture: title tag, meta description, H1, heading structure, word count, internal link count, schema type, canonical URL, robots meta, hreflang (if applicable), image count, render-blocking signals.

### Step 3 — Site-Wide Pattern Analysis

Identify across all pages:
- Duplicate or missing title tags / meta descriptions
- Broken internal links, orphan pages (>3 clicks from home)
- Schema coverage gaps
- Core Web Vitals risk signals (image weight, inline scripts, layout shift sources)
- Keyword cannibalization (multiple pages targeting the same query)

### Step 4 — HTML Report

Build a prioritized strategy report:

- **Executive summary** — top 3 site-wide issues
- **Page-by-page table** — title, meta, H1, internal links, schema, score per page
- **Priority fix list** grouped by tier:
  - Critical — ranking blockers (missing canonical, duplicate titles, crawl errors)
  - Important — ranking improvers (content gaps, schema opportunities, word count)
  - Quick wins — low effort, measurable gain (meta description fixes, alt text, internal links)
- **Architecture recommendations** — nav structure, internal linking, content silo improvements
- **Content opportunities** — keyword gaps, missing pages worth creating

Write to `seo-audit.html`. Open in browser after writing.

**Rule:** Sort the priority fix list by impact × effort. Always lead with the 3 highest-ROI fixes.
