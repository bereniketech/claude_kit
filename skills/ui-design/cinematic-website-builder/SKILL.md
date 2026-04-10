---
name: cinematic-website-builder
description: "Autonomous website builder with DESIGN.md-powered theming. Matches user description to 58 curated design systems (Stripe, Vercel, Airbnb, Tesla, etc.) for pixel-perfect brand-quality UI. Clarifies requirements, selects DESIGN.md, extracts brand, optionally applies cinematic GSAP modules, generates images via Gemini, assembles a polished site, and deploys to Vercel. Single-prompt end-to-end."
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebFetch, WebSearch
---

# Cinematic Website Builder

Autonomous end-to-end website builder with **DESIGN.md-powered theming**. Takes a business description, selects from 58 curated design systems (extracted from Stripe, Vercel, Airbnb, Tesla, etc.), and delivers a deployed, polished website — optionally with cinematic GSAP scroll/hover/ambient effects from a 30-module library.

**What is DESIGN.md?** A plain-text design system document that defines colors, typography, components, shadows, spacing, and responsive rules. Introduced by Google Stitch. Drop one into a project and the AI agent generates UI that matches the design system exactly.

---

## 1. Clarify (3 Questions Max)

Gather before building:

| Required | Optional |
|----------|----------|
| Business type (SaaS, restaurant, agency, etc.) | Brand URL to scrape |
| Target audience + primary CTA | Screenshot for inspiration |
| Dark or light preference | Specific sections needed |

**Ask:** "Do you want cinematic scroll/hover effects, or a clean modern site? Any brand aesthetic you'd like to match (e.g., 'like Stripe', 'Tesla-style', 'Airbnb vibe')?"
- Cinematic → module selection in Step 4
- Clean modern → standard path (Tailwind + CSS transitions only)
- Brand reference → direct DESIGN.md selection in Step 2

**Rule:** If user says "just build it" — default to dark theme, standard path, SaaS layout, Vercel design system. Proceed immediately.

---

## 2. DESIGN.md Selection

Based on the user's description, select the best matching DESIGN.md from `skills/ui-design/cinematic-website-builder/design-md/{brand}/DESIGN.md`.

### By Business Type

| Business Type | Primary Pick | Alternates |
|---|---|---|
| SaaS / Developer Tools | vercel | linear.app, cursor, supabase, raycast |
| AI / ML Product | claude | cohere, mistral.ai, together.ai, x.ai |
| Fintech / Banking | stripe | revolut, wise, coinbase |
| Crypto / Trading | kraken | coinbase, revolut |
| E-commerce / Marketplace | airbnb | pinterest, stripe |
| Creative Agency / Design | framer | figma, clay, webflow |
| Docs / Knowledge Base | mintlify | notion, sanity |
| Productivity / Workspace | notion | airtable, superhuman, miro, cal |
| DevOps / Infrastructure | hashicorp | sentry, clickhouse, mongodb |
| Automotive / Luxury | tesla | ferrari, lamborghini, bmw, renault |
| Aerospace / Engineering | spacex | nvidia, ibm |
| Music / Entertainment | spotify | elevenlabs, runwayml |
| Messaging / Support | intercom | resend, zapier |
| Automation / Workflows | zapier | composio |
| Analytics / Data | posthog | sentry, clickhouse |
| Mobile App | expo | lovable |
| Video / Media | runwayml | elevenlabs, minimax |
| Terminal / CLI | warp | ollama, opencode.ai |

### By Mood / Aesthetic

| Mood | Pick |
|---|---|
| Dark + techy + minimal | vercel |
| Dark + cinematic + luxury | tesla |
| Dark + neon + futuristic | nvidia |
| Dark + editorial + premium | stripe |
| Dark + emerald + developer | supabase |
| Light + warm + friendly | airbnb |
| Light + clean + professional | cal |
| Light + playful + colorful | figma |
| Light + editorial + content | notion |
| Bold + motion + design-forward | framer |
| Monochrome + futuristic | spacex |
| Vibrant + music + energy | spotify |
| Purple + precise + minimal | linear.app |

### By Explicit Brand Reference

If the user says "make it look like [brand]" — use that brand's DESIGN.md directly.

**Available brands (58):** airbnb, airtable, apple, bmw, cal, claude, clay, clickhouse, cohere, coinbase, composio, cursor, elevenlabs, expo, ferrari, figma, framer, hashicorp, ibm, intercom, kraken, lamborghini, linear.app, lovable, minimax, mintlify, miro, mistral.ai, mongodb, notion, nvidia, ollama, opencode.ai, pinterest, posthog, raycast, renault, replicate, resend, revolut, runwayml, sanity, sentry, spacex, spotify, stripe, supabase, superhuman, tesla, together.ai, uber, vercel, voltagent, warp, webflow, wise, x.ai, zapier

**Rule:** Read the selected DESIGN.md in full. Extract: color palette (Section 2), typography hierarchy (Section 3), component patterns (Section 4), spacing (Section 5), shadow system (Section 6), Do's/Don'ts (Section 7), responsive rules (Section 8).

Present your pick: *"I'll use the **[Brand] design system** — [one-line aesthetic description]. Want to swap?"*

---

## 3. Brand Extraction (from DESIGN.md + User Input)

The selected DESIGN.md provides the complete design foundation. Layer user-specific overrides on top:

**DESIGN.md provides:** Full color palette, typography hierarchy, shadow system, spacing scale, component patterns, responsive breakpoints. Read Sections 2–8 directly.

**User overrides (optional):**
- **URL given:** Fetch with WebFetch. Extract brand-specific colors/fonts and merge with DESIGN.md structure.
- **Screenshot given:** Analyze with Gemini vision. Extract palette and merge.
- **Specific colors given:** Replace DESIGN.md accent/primary, keep rest intact.

Map to the CSS variable contract (extended from DESIGN.md):
```css
:root {
  --bg: ...; --text: ...; --muted: ...; --accent: ...; --border: ...;
  --shadow-sm: ...; --shadow-md: ...; --shadow-lg: ...;
  --radius: ...;
}
```

Also carry forward from DESIGN.md: font-family + weight hierarchy, letter-spacing values, and component-specific tokens.

---

## 4. Module Selection (Cinematic Path Only)

Select exactly 3 modules — one hero, one body, one ambient:

| Business Type | Hero | Body | Ambient |
|---|---|---|---|
| SaaS / Tech | text-mask | sticky-cards | mesh-gradient |
| E-commerce | zoom-parallax | coverflow | kinetic-marquee |
| Creative Agency | cursor-reactive | split-scroll | glitch-effect |
| Restaurant / Hospitality | curtain-reveal | accordion-slider | mesh-gradient |
| Real Estate / Luxury | color-shift | cursor-reveal | gradient-stroke |
| Healthcare / Professional | svg-draw | sticky-cards | typewriter |
| Education | sticky-stack | horizontal-scroll | text-scramble |
| Personal Brand | text-mask | split-scroll | circular-text |

**Mood modifiers** (override table defaults):
- Cinematic → prefer scroll-driven (text-mask, curtain-reveal, color-shift)
- Playful → prefer cursor/hover (cursor-reactive, magnetic-grid, image-trail)
- Corporate → ambient-only with subtle scroll (sticky-cards + typewriter)
- Editorial → text-mask + split-scroll + typewriter
- Luxury → color-shift + gradient-stroke, slow easing

Present picks with one-line rationale each. Let user swap before proceeding.

**Rule:** Module HTMLs live at `skills/ui-design/cinematic-website-builder/cinematic-site-components/{name}.html`. Read each selected file to extract its CSS and JS.

---

## 5. AI Image Generation

Use Gemini API for hero and section images:

```python
# Requires GEMINI_API_KEY env var
from google import genai
client = genai.Client()
response = client.models.generate_content(
    model="gemini-2.0-flash-exp",  # or gemini-2.5-pro for higher quality
    contents="{business_type} {mood} cinematic hero image, "
             "{color_palette_description}, editorial photography, "
             "16:9 aspect ratio, dramatic lighting",
    config=genai.types.GenerateContentConfig(response_modalities=["IMAGE", "TEXT"])
)
# Save first image part to public/hero.png
```

Generate 1-2 images: hero background + optional section divider. Place in the hero module's image slot.

**Fallback:** If Gemini unavailable — use CSS gradient backgrounds or `https://placehold.co/1920x1080/{accent-hex}/{bg-hex}`.

---

## 6. Site Assembly

### Cinematic Path
1. Read each selected module HTML from `skills/ui-design/cinematic-website-builder/cinematic-site-components/`
2. Extract the `<style>` block and `<script>` block from each
3. Merge into single page: shared `<head>` (GSAP CDN, Google Fonts, unified `:root` vars) → `<body>` sections
4. Structure: `nav` → hero section (module 1) → content sections using module 2 pattern (repeat for features/services/testimonials) → ambient module woven as background → CTA section → footer
5. Replace all placeholder copy with brand-specific text using PAS (Problem-Agitate-Solve) framework
6. Apply DESIGN.md component patterns: buttons, cards, inputs match Section 4 specs
7. Apply DESIGN.md Do's and Don'ts (Section 7) as final style validation
8. Inject AI-generated images into appropriate slots

### Standard Path
1. Single `index.html` with Tailwind CDN (`<script src="https://cdn.tailwindcss.com"></script>`)
2. Responsive layout: hero → features grid → testimonials → CTA → footer
3. CSS transitions for hover/scroll effects (no GSAP dependency)
4. Apply brand colors via Tailwind `extend` config in inline `<script>` block
5. Apply DESIGN.md component patterns for buttons, cards, inputs (Section 4)

### Both Paths — DESIGN.md Integration
- Apply typography hierarchy from DESIGN.md Section 3 (font-family, sizes, weights, letter-spacing)
- Apply shadow system from DESIGN.md Section 6 (use exact shadow values, not generic ones)
- Apply responsive rules from DESIGN.md Section 8 (breakpoints, touch targets)
- Follow DESIGN.md Do's and Don'ts (Section 7) as guardrails
- Add SEO meta tags: `<title>`, `<meta name="description">`, Open Graph, favicon
- Ensure mobile-first responsive design (test at 375px)
- Add `<meta name="viewport" content="width=device-width, initial-scale=1">`

**Rule:** Never hardcode colors. Every color must reference `--bg`, `--text`, `--muted`, `--accent`, or `--border`.

---

## 7. Deploy to Vercel

```bash
# Create project structure if single HTML
mkdir -p my-site/public && cp hero.png my-site/public/ && cp index.html my-site/
cd my-site && npx vercel --prod --yes
```

Capture the deploy URL. If Vercel CLI not installed: `npm i -g vercel`.

**Rule:** If deploy fails (auth, config), output files locally and provide manual instructions: "Run `npx vercel` in the project directory and follow the prompts."

---

## 8. Screenshot & Iterate

```bash
npx playwright screenshot index.html --full-page --viewport-size="1440,900" desktop.png
npx playwright screenshot index.html --full-page --viewport-size="375,812" mobile.png
```

Present both screenshots. Check:
- Text readable against background (contrast ratio >= 4.5:1)
- GSAP animations initialize (check CDN loaded, no console errors)
- CTA visible above the fold on desktop
- Mobile layout not broken (no horizontal overflow)
- Images render (no broken `<img>` tags)

Fix issues, re-deploy, re-screenshot. **Rule:** Max 3 iteration cycles, then present result.

---

## 9. Module Quick Reference

**Scroll-Driven (9):** text-mask, sticky-stack, zoom-parallax, horizontal-scroll, sticky-cards, svg-draw, curtain-reveal, split-scroll, color-shift
**Cursor & Hover (8):** cursor-reactive, accordion-slider, cursor-reveal, image-trail, flip-cards, magnetic-grid, spotlight-border, drag-pan
**Click & Tap (6):** view-transitions, particle-button, odometer, coverflow, dynamic-island, dock-nav
**Ambient & Auto (7):** text-scramble, kinetic-marquee, mesh-gradient, circular-text, glitch-effect, typewriter, gradient-stroke

All modules share: `:root` CSS variable contract, GSAP 3.x + ScrollTrigger CDN, Outfit font default.
Files: `skills/ui-design/cinematic-website-builder/cinematic-site-components/{module-name}.html`

---

## 10. DESIGN.md Library Reference

**Path:** `skills/ui-design/cinematic-website-builder/design-md/{brand}/DESIGN.md`

**Each DESIGN.md contains 9 sections:**

| # | Section | What to extract |
|---|---------|-----------------|
| 1 | Visual Theme & Atmosphere | Mood, density, design philosophy |
| 2 | Color Palette & Roles | Hex values with semantic roles → map to CSS vars |
| 3 | Typography Rules | Font families, full hierarchy (sizes, weights, line-heights, letter-spacing) |
| 4 | Component Stylings | Button, card, input specs with hover/active states |
| 5 | Layout Principles | Spacing scale, grid system, whitespace rules |
| 6 | Depth & Elevation | Shadow values (use exact values, not generic) |
| 7 | Do's and Don'ts | Design guardrails — follow strictly |
| 8 | Responsive Behavior | Breakpoints, touch targets, collapse strategy |
| 9 | Agent Prompt Guide | Quick color reference, ready-to-use prompts |

**Available brands (58):** airbnb, airtable, apple, bmw, cal, claude, clay, clickhouse, cohere, coinbase, composio, cursor, elevenlabs, expo, ferrari, figma, framer, hashicorp, ibm, intercom, kraken, lamborghini, linear.app, lovable, minimax, mintlify, miro, mistral.ai, mongodb, notion, nvidia, ollama, opencode.ai, pinterest, posthog, raycast, renault, replicate, resend, revolut, runwayml, sanity, sentry, spacex, spotify, stripe, supabase, superhuman, tesla, together.ai, uber, vercel, voltagent, warp, webflow, wise, x.ai, zapier

Each brand directory also includes `preview.html` and `preview-dark.html` for visual reference.
