---
name: asset-generation
description: Generates 3 coordinated AI prompts for scroll-stop website assets — clean hero shot, deconstructed/after state, and video transition. Delivers via a tabbed HTML page with one-click copy buttons. Works with any image generator (Midjourney, Flux, DALL-E) and any video model (Runway, Kling, Pika, Higgsfield).
---

# Asset Generation — Scroll-Stop Prompt Creator

Generate a coordinated 3-prompt set that produces scroll-stopping video content, then deliver via a premium tabbed HTML page.

---

## 1. Confirm the Object

Ask what to shoot if not specified. Default to **laptop**.

Determine the approach from the subject type:

| Subject type | Approach |
|---|---|
| Product with internal components | Deconstruction / exploded view |
| Service or transformation | Before / after states |

---

## 2. The 3-Prompt Framework

**Rule:** Pure white background (`#FFFFFF`, no gradient, no vignette) on all 3 prompts — critical for clean video transitions and scroll-stop website integration.

**Prompt A — Hero Shot (Assembled / Before)**

Clean subject centered, pure white background, soft studio lighting, 16:9, photorealistic, Apple-style product photography. Adjust: camera angle (3/4 default), material-specific details, exact product state.

**Prompt B — Deconstructed / After State**

Same angle, same lighting, same camera position as Prompt A — visual continuity is essential.

- Products: all 8–15 internal components floating with spatial relationships maintained, single explosion axis
- Transformations: vivid after state, same camera position, only the subject's condition has changed

Common component lists:

| Object | Components to list |
|--------|-------------------|
| Laptop | Unibody shell, LCD + ribbon cable, keyboard, trackpad, battery, logic board, SSD, fan + heatpipe, speakers (L+R), hinge, WiFi antenna, camera module |
| Phone | Glass back, battery, OLED, logic board, camera array, SIM tray, speaker, Taptic engine, USB-C, antenna bands, charging coil |
| Shoe | Outer sole, midsole, insole, upper panels, tongue, laces, heel counter, toe cap, eyelets |
| Food/beverage | Explosion style — all ingredients, garnish, ice cubes, glass shards, liquid splash in freeze-frame |

For other objects, list 8–15 real internal components or transformation details.

**Prompt C — Video Transition**

Start frame = Prompt A state. End frame = Prompt B state. White background throughout, locked-off camera, no camera movement.

- Products: pieces separate in cascading sequence (not all at once), slow-in/slow-out easing, 2–3s, slight per-piece rotations to reveal 3D form
- Transformations: progressive sweep reveal, smooth and deliberate

Duration: 5–6s. Aspect ratio: 16:9.

Always offer: reverse version (end → start), loop version, slow-mo (8–10s).

---

## 3. Deliver the HTML Page

Read `assets/prompt-page-template.html`. Replace all placeholders:

| Placeholder | Value |
|---|---|
| `{{OBJECT_NAME}}` | Subject name for page title |
| `{{HEADING_LINE1}}` / `{{HEADING_LINE2}}` | Split heading text |
| `{{TAB_A_NAME}}` / `{{TAB_A_SHORT}}` | Tab A label + mobile short label |
| `{{TAB_B_NAME}}` / `{{TAB_B_SHORT}}` | Tab B label + mobile short label |
| `{{PROMPT_A}}` / `{{PROMPT_B}}` / `{{PROMPT_C}}` | Full prompt text (plain text, no HTML) |

Escape `<`, `>`, `&` as HTML entities in all prompt text.

Write to `prompts.html`. Open: `start prompts.html` (Windows) · `open prompts.html` (macOS) · `xdg-open prompts.html` (Linux).

Also present all 3 prompts in chat as a copy-paste fallback.

---

## 4. Troubleshooting

| Issue | Fix |
|-------|-----|
| Inconsistent lighting between A and B | Add "match exact lighting direction from reference image" to Prompt B |
| Deconstruction looks random | Add "maintain spatial relationships, explosion along single axis" |
| White background not pure | Add "pure white #FFFFFF background, no gradient, no vignette" explicitly |
| Before/after angle mismatch | Copy exact camera description from Prompt A into Prompt B verbatim |
| Video transition too fast/jerky | Increase duration to 8–10s, add "smooth eased motion, slow-in slow-out" |
| HTML special chars in prompt | Always escape `<`, `>`, `&` when inserting into template |
