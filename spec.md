# Drew & Monique — Tee Off Wedding Intro

**Build specification and asset-generation prompt pack**
Status: Ready to build
Date: May 22, 2027 · Estes Park, Colorado

---

## 0. Purpose of this document

This file is the single source of truth for the wedding intro page. It defines the theme,
copy, layout, motion, asset list, **copy-paste generation prompts**, redirect behavior,
accessibility, performance budget, and acceptance criteria.

The page is a cinematic front door: guests click **Tee Off**, watch a golf ball fly out of
the mountains and shatter the screen, land on the invitation, then **Enter Our Wedding**,
which sends them to the couple's existing Minted wedding site.

Nothing in this spec depends on a framework, build step, or video file. The deliverable is a
static `index.html` plus the image assets listed in §9.

---

## 1. Project overview

| Item | Value |
| --- | --- |
| Deliverable | Static single-page site (`index.html` + `assets/`) |
| Hosting | Any static host (GitHub Pages, Netlify, Cloudflare Pages, S3) |
| Dependencies | None. Vanilla HTML/CSS/JS |
| Core interaction | Tee Off → ball flight → impact → glass shatter → invitation → external redirect |
| Runtime of sequence | ≈ 2.9 s (skippable from 300 ms) |
| Primary destination | Minted wedding site via `MINTED_URL` constant |

**Goals**

1. Feel like a film title card, not a template.
2. Earn the click: the Tee Off button should promise something and deliver it.
3. Retina-crisp at every viewport. No upscaled storyboard crops.
4. Work on a mid-range phone on hotel Wi-Fi.

**Non-goals**

- No fake homepage. The intro ends at Minted.
- No autoplay audio.
- No pre-rendered MP4 required. (A video slot is documented in §16 as an optional upgrade.)

---

## 2. Theme tokens

```css
:root {
  /* Core palette */
  --forest-950: #07140B;
  --forest-900: #0B1A13;
  --forest-800: #10271D;
  --forest-700: #173529;
  --ink:        #14271F;

  --ivory:      #F5F0E4;
  --ivory-warm: #F7F0E2;
  --ivory-dim:  #E6DFD0;

  --gold:       #C7A86B;
  --gold-light: #E2CCA0;
  --gold-deep:  #A8863F;

  /* Scrims */
  --scrim-hero:
    linear-gradient(180deg,
      rgba(7, 20, 11, 0.28) 0%,
      rgba(7, 20, 11, 0.50) 52%,
      rgba(7, 20, 11, 0.90) 100%);
  --scrim-side:
    linear-gradient(90deg,
      rgba(7, 20, 11, 0.72) 0%,
      rgba(7, 20, 11, 0.20) 46%,
      rgba(7, 20, 11, 0) 70%);
  --scrim-invite:
    linear-gradient(180deg,
      rgba(10, 26, 18, 0.42) 0%,
      rgba(10, 26, 18, 0.80) 100%);

  /* Motion */
  --ease-out-expo: cubic-bezier(0.16, 1, 0.30, 1);
  --ease-shatter:  cubic-bezier(0.20, 0.80, 0.20, 1);
  --ease-soft:     cubic-bezier(0.40, 0, 0.20, 1);

  /* Layout */
  --content-max: 760px;
  --gutter: clamp(1.25rem, 5vw, 4rem);
  --radius: 0; /* editorial: sharp corners everywhere */
}
```

Rules:

- Gold is an accent only: hairlines, eyebrows, button borders, one word in a heading.
- Body text is never pure white. Minimum `--ivory` on forest for contrast.
- No drop shadows stronger than `0 10px 40px rgba(0,0,0,.35)`.
- No gradients other than the scrims above, plus the shard specular highlight.

---

## 3. Typography

| Role | Family | Weight | Size | Tracking | Case |
| --- | --- | --- | --- | --- | --- |
| Display names | Cormorant Garamond | 300 | `clamp(3rem, 9vw, 6.8rem)` | `-0.045em` | Title |
| Section heading | Cormorant Garamond | 300 | `clamp(2.8rem, 8vw, 6rem)` | `-0.03em` | Title |
| Tagline | Cormorant Garamond | 400 italic | `clamp(1rem, 2vw, 1.35rem)` | `0.02em` | Sentence |
| Eyebrow | Jost | 500 | `0.7rem` | `0.34em` | UPPER |
| Meta / date | Jost | 400 | `clamp(0.72rem, 1.4vw, 0.82rem)` | `0.20em` | UPPER |
| Button | Jost | 500 | `0.75rem` | `0.20em` | UPPER |
| Small note | Jost | 400 | `0.68rem` | `0.12em` | Sentence |

Fallback stack:

```css
--font-display: "Cormorant Garamond", Georgia, "Times New Roman", serif;
--font-ui: "Jost", "Helvetica Neue", Arial, sans-serif;
```

**Font delivery:** self-host WOFF2 (`/fonts/cormorant-garamond-300.woff2`,
`/fonts/jost-400.woff2`, `/fonts/jost-500.woff2`) with `font-display: swap` and
`<link rel="preload" as="font" type="font/woff2" crossorigin>`. Fall back to Georgia if a
host is not configured. Georgia must be an intentional, shippable fallback, not a broken state.

Line heights: display `0.96–1.0`, body `1.7`, meta `1.9`.

---

## 4. Layout and responsive behavior

### 4.1 Hero

- Full-viewport (`100svh`, with `100vh` fallback), `overflow: hidden`.
- Background art is `<picture>` with a media switch at 768 px:
  - `(max-width: 767px)` → `hero-mobile-*` (9:16, composed independently).
  - otherwise → `hero-desktop-*` (16:9).
- Two stacked scrims: `--scrim-hero` plus `--scrim-side`, both above the image, below the text.
- Content block: `max-width: var(--content-max)`, centered with `text-align: center`.
  - `≥ 1024px`: shift the content block to `58%` horizontal so the golfer stays visible on
    the left third. Nudge, do not hard-position.
  - `< 1024px`: horizontally centered.
- Vertical placement: content center is `46%` of viewport height on desktop, `38%` on mobile,
  so the CTA never collides with the subject or the crop edge.
- Bottom safe area: `padding-bottom: max(2rem, env(safe-area-inset-bottom))`.

### 4.2 Invitation

- Same `<picture>` element as the hero? **No.** Use `invite-backdrop-*` so the reveal feels
  like a new place with softer, type-friendly negative space in the center.
- `position: absolute; inset: 0`, grid-centered, `padding: clamp(1.5rem, 6vw, 4rem)`.
- Gold hairline inset frame: `inset: clamp(10px, 1.6vw, 18px)`, `1px solid var(--gold)`,
  `pointer-events: none`.
- Inner block `max-width: 700px`, `text-shadow: 0 3px 25px rgba(0,0,0,.5)`.

### 4.3 Breakpoints

| Breakpoint | Behavior |
| --- | --- |
| `≤ 380px` | Display `2.9rem`, button full width, meta tracking `0.12em` |
| `381–767px` | Portrait art, centered stack, button `min-width: 260px` |
| `768–1023px` | Landscape art, centered stack |
| `1024–1439px` | Landscape art, content shifted to 58% |
| `≥ 1440px` | Max content width 760 px, same shift, art `object-position: center 42%` |

No horizontal scroll at any width from 320 px to 3840 px.

### 4.4 Safe-area and zoom

- Never lock zoom or disable user scaling.
- All tap targets ≥ 44 × 44 px.
- Test at 200% browser zoom at 1280 px.

---

## 5. Copy deck

All strings are final unless the couple changes them. Do not invent additional copy on the page.

### Hero

| Slot | Copy |
| --- | --- |
| `<title>` | `Drew & Monique — Tee Off` |
| Meta description | `Tee off into our greatest adventure yet. May 22, 2027 · Estes Park, Colorado.` |
| Eyebrow | `Our greatest drive yet` |
| Display | `Drew & Monique` (gold `&`) |
| Meta | `May 22, 2027 · Estes Park, Colorado` |
| Tagline | `A love worth celebrating.` |
| Primary CTA | `Tee Off ↗` |
| Small note | `A cinematic wedding experience` |
| Skip | `Skip intro` |

### Invitation

| Slot | Copy |
| --- | --- |
| Eyebrow | `Together is a beautiful place to be` |
| Display | `Drew & Monique` (two lines on mobile) |
| Meta | `May 22, 2027 · Estes Park, Colorado` |
| Tagline | `A love worth celebrating.` |
| Primary CTA | `Enter Our Wedding ↗` |
| Secondary | `Replay intro` |

### System / accessibility

| Slot | Copy |
| --- | --- |
| Live status | `Playing the intro` |
| Missing URL guard | `Our wedding site isn’t linked yet. Add the Minted URL to MINTED_URL.` |
| Hero art alt | Decorative (`aria-hidden="true"`); meaning is carried by the text |
| Skip aria-label | `Skip the intro animation` |

Typography notes: use a real arrow glyph in a `<span aria-hidden="true">`, not an image,
so it stays sharp at any DPR.

---

## 6. Page states and flow

```
┌─────────┐   click Tee Off   ┌───────────┐   ~2.9 s   ┌────────────┐   click Enter   ┌──────────────┐
│  hero   │ ────────────────► │ sequence  │ ─────────► │ invitation │ ──────────────► │ Minted site  │
└─────────┘                   └───────────┘            └────────────┘                 └──────────────┘
     ▲                                                        │
     └────────────────────── Replay intro ────────────────────┘
```

State rules:

- `hero` is the default on first load.
- Clicking Tee Off twice must not restart or stack animations. Guard with an `isPlaying` flag.
- Skip is available from 300 ms after click and jumps to `invitation` with a 400 ms crossfade.
- `Replay intro` returns to `hero` and clears the `sessionStorage` flag.
- If a guest returns from Minted via Back (`pageshow` with `event.persisted`), show
  `invitation` directly. Never replay the shatter.
- Persist `dm_intro_seen = "1"` in `sessionStorage` when `invitation` is reached. On a fresh
  load with that flag present, start at `invitation`. (Guests who refresh should not sit
  through the sequence again.)

---

## 7. Motion timeline

Impact point, where the ball strikes the "glass":

| Viewport | X | Y |
| --- | --- | --- |
| ≥ 768 px | 62% | 38% |
| < 768 px | 52% | 36% |

Ball origin, on the mountain ridge:

| Viewport | X | Y | Start scale |
| --- | --- | --- | --- |
| ≥ 768 px | 32% | 47% | 0.04 |
| < 768 px | 28% | 44% | 0.05 |

| t (ms) | Beat | What happens |
| ---: | --- | --- |
| 0 | Press | CTA ripple; hero copy fades and lifts (`opacity 0`, `translateY(-8px)`, 220 ms, `--ease-soft`) |
| 0–1400 | Camera push | Art layer `scale(1 → 1.06)` and `translateY(0 → -6px)`; vignette deepens 15% |
| 120–380 | Tee | Ball fades in at origin, `scale(.04)`, with a 4 px sun glint and a tiny dust puff |
| 380–1250 | Flight | Ball arcs to impact point, `scale(.04 → 1.0)`, `blur(0 → 0.4px)` at peak speed; 3 speed streaks; horizon heat shimmer |
| 1250–1370 | Impact | White bloom at 0 → 0.85 → 0.25 opacity; 3 px screen shake (2 cycles); chromatic split ±2 px on the hero layer; ball overshoots to `scale(1.15)` then vanishes |
| 1300–1520 | Cracks | 8 primary + 14 secondary SVG fractures draw outward via `stroke-dashoffset`, 120–220 ms per path, staggered 0–80 ms |
| 1450–2350 | Shatter | ~40 shards fly outward; invitation is already mounted underneath and becomes visible through the gaps |
| 2050–2500 | Invitation settle | Backdrop `scale(1.04 → 1.0)`; inset frame fades in |
| 2150–2900 | Type reveal | Eyebrow → names → rule → meta → tagline → CTA, staggered 70 ms, `translateY(10px → 0)` |
| 2900 | Idle | Sequence complete; only the CTA and Replay remain interactive |

**Easing and feel**

- Flight: `cubic-bezier(.15, .7, .2, 1)` with a slight deceleration at impact.
- Shards: `--ease-shatter`, duration 900–1400 ms, delay proportional to distance from impact.
- Nothing eases with `linear` except the crack draw.
- Total sequence must land at **2.8–3.0 s**. Tune, do not exceed 3.2 s.

**Reduced motion**

`@media (prefers-reduced-motion: reduce)`: skip the sequence entirely. Show `invitation`
immediately, with a 200 ms opacity fade. No ball, no shards, no shake, no camera push.
The Skip button is hidden in this mode. Replay still works but resolves instantly.

---

## 8. Shatter mechanics

No video. The shatter is real geometry over the HD art.

### 8.1 Layers, bottom to top

1. `invite-backdrop` (mounted from the start, visible through the gaps)
2. `invite-content`
3. `hero-art` (the photo, with scrims)
4. `hero-copy`
5. `ball`
6. `crack-overlay` (SVG or image, `mix-blend-mode: screen`)
7. `shard-field` (≈40 clipped divs)
8. `impact-flash`
9. `skip`

### 8.2 Shard construction

Each shard is a `<div>` with:

- `position: absolute; inset: 0;`
- `background-image: var(--hero-art);`
- `background-size: cover;` and a `background-position` matching the viewport so the composite
  image aligns perfectly.
- `clip-path: polygon(...)` from a **precomputed, deterministic** polygon list.
- `will-change: transform, opacity;` only while animating.

Generate the polygons once, at build time, not per load:

- Three rings around the impact point at radii ≈ 12%, 26%, and 46% of the viewport diagonal.
- 8 / 14 / 18 shards per ring, with 6–14° angular jitter per vertex.
- Vertices are shared between neighboring shards so seams read as glass.
- Store as an array of normalized polygon points (`0–1`), scaled to the viewport on resize.
- Ship the array inline in the script. No runtime Voronoi library.

Per-shard motion data is derived from its centroid:

```js
{
  poly: [[0.61,0.30],[0.68,0.26],[0.72,0.34],[0.64,0.39]],
  dx: 0.42,      // fraction of viewport width
  dy: -0.18,     // fraction of viewport height
  rot: -34,      // degrees
  scale: 0.86,
  delay: 0,      // ms, 0–180
  dur: 1150,     // ms, 900–1400
  blur: 0        // px, 0–6
}
```

Keyframe shape:

```css
@keyframes shatterShard {
  0%   { transform: translate3d(0,0,0) rotate(0) scale(1);
         opacity: 1; filter: blur(0); }
  18%  { opacity: 1; }
  100% { transform:
           translate3d(calc(var(--dx) * 100vw),
                       calc(var(--dy) * 100vh), 0)
           rotate(var(--rot)) scale(var(--scale));
         opacity: 0; filter: blur(var(--blur)); }
}
```

Additional shard detail:

- A 0.75 px `::after` inset border in `rgba(255,255,255,.28)` on each shard edge.
- A specular gradient sweep across the glass at impact, 120 ms, `screen` blend.
- Sound is **not** shipped. If `impact.mp3` is added later, it may only play after the
  Tee Off click gesture, and it must be ≤ 80 KB.

### 8.3 Performance rules

- Animate only `transform`, `opacity`, and `filter`. Never animate `top/left/width/height`.
- Cap shard count: 40 at ≥ 768 px, 24 below 768 px.
- Remove `will-change` on `animationend`.
- After the sequence, `display: none` the shard field and crack overlay so they cost nothing.
- No `backdrop-filter` anywhere in the sequence. It is the fastest way to drop frames on mobile.

---

## 9. Asset manifest

All masters are 16:9 or 9:16 landscape/portrait as noted. Export ladder below each master.
Filenames are exact.

| ID | File stem | Master px | AR | Used for |
| --- | --- | --- | --- | --- |
| A1 | `hero-desktop` | 3840 × 2160 | 16:9 | Hero art ≥ 768 px |
| A2 | `hero-mobile` | 2160 × 3840 | 9:16 | Hero art < 768 px |
| A3 | `ball` | 1600 × 1600 | 1:1 | Golf ball during flight |
| A4 | `crack-overlay` | 4096 × 2304 | 16:9 | Impact fracture web |
| A5 | `glass-specular` | 4096 × 2304 | 16:9 | Optional glass edge/highlight overlay |
| A6 | `invite-backdrop` | 3840 × 2160 | 16:9 | Invitation reveal backdrop |
| A7 | `og-card` | 1200 × 630 | 1.91:1 | Social share image |
| A8 | `grain` | 2048 × 2048 | 1:1 | Tileable film grain overlay |

Export ladder (progressive enhancement, largest first in `srcset`):

| Asset | Widths | Formats |
| --- | --- | --- |
| `hero-desktop` | 3840, 2560, 1920, 1280 | AVIF → WebP → JPG |
| `hero-mobile` | 2160, 1440, 1080 | AVIF → WebP → JPG |
| `invite-backdrop` | 3840, 2560, 1920 | AVIF → WebP → JPG |
| `ball` | 1600, 800 | WebP + PNG (transparent) |
| `crack-overlay` | 4096, 2048 | WebP + PNG (transparent or black) |
| `glass-specular` | 4096, 2048 | WebP + PNG (transparent) |
| `og-card` | 1200 × 630 | JPG (quality 82) |
| `grain` | 2048 | PNG (8-bit alpha, tiled) |

Naming example: `assets/hero-desktop-2560.avif`.

---

## 10. Generation prompts

### 10.1 Shared style block

Prepend this to **every** prompt so the set feels like one shoot.

```text
Warm golden-hour Colorado palette: deep forest green (#10271D), ivory (#F5F0E4),
muted gold (#C7A86B). Cinematic 35mm film grain, soft halation around highlights,
natural directional light, gentle shadow rolloff, no HDR halos, no crunchy sharpening,
editorial wedding-magazine color grade, photorealistic, ultra-detailed, 8K detail.
```

Master negative prompt:

```text
text, letters, typography, watermark, signature, logo, brand markings, border, frame,
UI, button, caption, visible face of the subject, extra limbs, extra fingers,
distorted golf club, duplicated golfer, floating objects, cartoon, illustration,
painting, 3D render, plastic skin, fisheye distortion, heavy HDR, oversaturation,
color banding, lens dirt, JPEG artifacts, low resolution, cropped subject
```

Per-model parameter line:

- **Midjourney:** `--ar 16:9 --style raw --s 150 --q 2 --no <negative>`
- **Flux / Seedream / Nano Banana:** guidance `3.5–4.5`, 4 MP or larger native output
- **Ideogram:** explicitly state "no text" twice; it tends to add lettering to posters
- **SDXL family:** CFG `5.5–6.5`, 30–35 steps, DPM++ 2M Karras

Consistency lock: generate **A1 first**. Then use A1 as an image prompt / reference for A2,
A6, and A7 so the location, grade, and light match. Seeds: reuse the same seed for A1 and A6
where the model allows it.

---

### A1 — `hero-desktop` (3840 × 2160, 16:9)

```text
Cinematic editorial photograph, 16:9. A man in a tailored black tuxedo, photographed
from behind at the top of his golf backswing, gripping a driver, on a manicured alpine
golf course in Estes Park, Colorado. He faces a dramatic range of snow-dusted Rocky
Mountain peaks. Golden-hour sun low on the horizon, warm rim light along his shoulders
and the club shaft, long soft shadows, layered valley haze, dark evergreen forest
framing left and right, dew on the fairway. Medium format, 85mm, f/2.8, shallow depth
of field on the peaks, deep forest greens, warm ivory highlights, muted gold sky,
natural film grain, low-contrast shadows with rich detail. Subject in the left third;
generous empty sky and mountain space on the right for typography. Photorealistic,
ultra-detailed.
```

- **Negative:** master negative.
- **Params:** `--ar 16:9 --style raw --s 150 --q 2`
- **QA reject if:** club looks bent, shoulders are duplicated, face is visible, sky bands,
  text appears, or the subject drifts into the center (we need the right-side space).

---

### A2 — `hero-mobile` (2160 × 3840, 9:16)

Compose independently. Do not simply crop A1; the subject must move down and left.

```text
Cinematic editorial photograph, 9:16 vertical. A man in a tailored black tuxedo seen
from behind at the top of his golf backswing, holding a driver, standing on a manicured
alpine fairway in Estes Park, Colorado. He occupies the lower-left of the frame, small
in scale, facing a towering wall of snow-dusted Rocky Mountain peaks in the upper third.
Golden-hour sun low behind the ridgeline, rim light on his shoulders and club, long
shadows across the fairway, dark evergreen silhouettes at the edges. Vast empty sky at
the top for a title, clean fairway at the bottom for a button. Medium format, 50mm,
f/4, photorealistic, ultra-detailed, natural film grain.
```

- **Negative:** master negative + `tight crop, cropped head, subject filling the frame`
- **Params:** `--ar 9:16 --style raw --s 150 --q 2`
- **QA reject if:** the head is cut, the club exits the frame awkwardly, or the top 30% is
  busy (that area holds the title).

---

### A3 — `ball` (1600 × 1600, 1:1)

```text
Extreme macro product photograph of a single white golf ball suspended mid-air against
a pure black background. Crisp dimple pattern, warm golden sunset rim light on the upper
right edge, subtle cool fill on the lower left, tiny dust particles catching the light,
faint motion streak trailing behind. Studio sharpness, physically accurate reflections,
centered with clean margin on all sides. No brand markings, no logos.
```

- **Negative:** master negative + `grass, ground, shadow on floor, multiple balls, hand`
- **Params:** `--ar 1:1 --style raw --s 250 --q 2`
- **Delivery:** key out the black to transparent PNG; also ship a black-background version
  for `mix-blend-mode: screen` fallback.
- **QA reject if:** dimples blur, the ball is not circular, or the highlight is blown.

---

### A4 — `crack-overlay` (4096 × 2304, 16:9)

Generate on pure black, composite with `screen` (or key to alpha in post).

```text
Photorealistic shattered glass crack pattern isolated on pure black, full-frame.
A single impact point slightly right of center with a dense concentric fracture web
radiating outward: 8 to 12 primary radial cracks reaching the frame edges, fine secondary
splinters branching between them, tiny glass chips and dust at the impact, subtle
iridescent dispersion along the fracture edges, thin white and pale-gold lines, evenly
lit, high contrast, 4K, macro detail.
```

- **Negative:** master negative + `scene, landscape, mountains, sky, background objects, colored glass`
- **Params:** `--ar 16:9 --style raw --s 100 --q 2`
- **Delivery:** PNG with alpha preferred. If the generator cannot do alpha, ship the black
  version and use `mix-blend-mode: screen`, which matches the design.
- **QA reject if:** cracks are symmetrical/mechanical, lines are thicker than ~4 px at 4K,
  or the impact center is dead-center (it must sit near 62% / 38%).

---

### A5 — `glass-specular` (4096 × 2304, 16:9) — optional

```text
Photorealistic close-up of a shattered pane of clear glass, isolated on black, viewed
straight on. Jagged shard edges catching warm sunset light, subtle green tint in the
glass thickness, thin specular highlights along the break lines, a few loose fragments
floating just off the surface, shallow depth of field, macro detail, high contrast.
```

- **Negative:** master negative + `colored glass, stained glass, background scene`
- **Params:** `--ar 16:9 --style raw --s 200`
- **Delivery:** alpha PNG/WebP.

---

### A6 — `invite-backdrop` (3840 × 2160, 16:9)

```text
Photorealistic fine-art landscape, 16:9. A still alpine lake at twilight in Rocky
Mountain National Park, glassy water mirroring snow-capped peaks, soft lavender and
peach sky, dark pine silhouettes framing the left and right edges, a warm gold glow
lingering at the horizon. No people. Calm, romantic, quiet. Generous open sky and water
in the center of the frame for a centered invitation. Editorial landscape photography,
long exposure, ultra-detailed, natural film grain, gentle contrast.
```

- **Negative:** master negative + `people, boat, cabin, road, birds in foreground`
- **Params:** `--ar 16:9 --style raw --s 200 --q 2`
- **QA reject if:** the center is busy, the horizon is crooked, or the sky bands.

---

### A7 — `og-card` (1200 × 630, 1.91:1)

```text
Cinematic 1.91:1 crop of a tuxedoed golfer from behind at the top of his backswing on an
alpine golf course at golden hour, Rocky Mountain peaks behind him, subject on the left
third, open sky on the right for a title overlay, warm gold and forest green palette,
photorealistic, editorial, high detail, no text.
```

- **Negative:** master negative.
- **Params:** `--ar 191:100` (MJ) or crop from A1.
- **Delivery:** JPG q82, 1200 × 630. Title/date overlay is added by the page's meta tags,
  not baked into the image.

---

### A8 — `grain` (2048 × 2048, 1:1, tileable)

```text
Uniform 35mm film grain texture, monochrome, mid-gray, fine silver halide structure,
seamlessly tileable, no scratches, no dust, no vignette, no color cast, flat even density.
```

- **Negative:** `scratches, dust, hair, vignette, color, pattern, banding`
- **Params:** not a typical MJ use case; a procedural grain node (Photoshop, Affinity,
  or a CSS `feTurbulence` fallback) is acceptable.
- **Delivery:** PNG with alpha, tiled at `opacity: .06`, `mix-blend-mode: overlay`.

---

### 10.2 Generation workflow

1. Generate A1 at the model's maximum native resolution. Review against the A1 QA list.
2. Upscale 2× (Topaz Gigapixel, Magnific, or Real-ESRGAN) only if native output is below
   target. Do not upscale a JPEG.
3. Inspect the upscaled file at 100% for sky banding, plastic skin, and warped club edges.
4. Feed A1 as an image reference for A2, A6, and A7 to lock the location and grade.
5. Generate A3 and A4 independently; they are isolated assets and do not need scene matching.
6. Key out black backgrounds for A3 and A4 before export.
7. Export the ladder in §9. Verify each AVIF/WebP visually — encoders can crush shadows.

---

## 11. Post-processing and export

| Step | Tool | Notes |
| --- | --- | --- |
| Color match | Lightroom / Photoshop | Match A1 → A2 → A6 with the same grade |
| Alpha key | Photoshop / Affinity | A3, A4, A5. Feather 0.5–1 px, no hard matte fringe |
| Grain | A8 or ACR | Uniform, no visible tiling at 4K |
| Compression | `avifenc`, `cwebp`, `squoosh` | AVIF q 55–65, WebP q 78–82 |
| Ladder | Script or Squoosh batch | Exact widths from §9 |
| Verification | Browser at 200% DPR | No softness, no color shift, no banding |

Reference encoding targets:

```bash
# AVIF
avifenc --min 20 --max 32 --speed 4 hero-desktop-3840.png hero-desktop-3840.avif

# WebP
cwebp -q 80 -m 6 hero-desktop-3840.png -o hero-desktop-3840.webp

# JPG fallback
magick hero-desktop-3840.png -quality 84 -strip hero-desktop-3840.jpg
```

Strip EXIF and ICC profiles unless the profile is needed for color accuracy.

---

## 12. Redirect contract

The invitation CTA is an anchor, not a button:

```html
<a
  id="enter"
  class="btn"
  href="https://www.minted.com/wedding-websites/REPLACE-ME"
  rel="noopener"
  aria-label="Enter our wedding site on Minted"
>Enter Our Wedding <span aria-hidden="true">↗</span></a>
```

Single source of truth, at the top of the inline script:

```js
const MINTED_URL = "https://www.minted.com/wedding-websites/REPLACE-ME";
```

Behavior:

- If `MINTED_URL` still contains `REPLACE-ME`, intercept the click, `preventDefault()`, and
  show the guard message inline near the CTA. Do **not** navigate to a broken page.
- On entering `invitation`, inject `<link rel="preconnect" href="https://www.minted.com">`
  and `<link rel="dns-prefetch" href="//minted.com">` once.
- Navigate in the same tab (`target="_self"`). No popups.
- Optional analytics hook: a single `data-cta="enter-minted"` attribute; no third-party
  scripts by default.
- Keep `rel="noopener"` even for same-tab navigation. It is harmless and future-proof.
- Never auto-redirect. The guest must click.

---

## 13. Accessibility

- **Semantics:** `<main>`, one `<h1>`, invitation `<h2>`, real `<a>` for the Minted link.
- **Keyboard:** Tab reaches Tee Off, Skip, Enter, Replay in DOM order. Visible focus ring:
  `outline: 2px solid var(--gold-light); outline-offset: 4px`.
- **Focus management:** on reveal, move focus to the invitation heading
  (`tabindex="-1"`, `focus({ preventScroll: true })`), and announce via
  `aria-live="polite"`: `Invitation revealed`.
- **Live status:** `role="status"` region announces `Playing the intro`.
- **Skip:** visible and focusable from the start; do not hide it behind hover.
- **Contrast:** ivory on forest ≥ 7:1; gold-on-forest for small text only at ≥ 4.5:1
  (use `--gold-light` on small text if needed).
- **Motion:** honor `prefers-reduced-motion: reduce` per §7.
- **Art:** decorative background images are `aria-hidden="true"`; the text carries meaning.
- **Zoom:** no `maximum-scale`, no `user-scalable=no`.
- **Timing:** nothing is time-limited; Skip exists; the invitation never auto-advances.

---

## 14. Performance budget

| Metric | Budget |
| --- | --- |
| Initial HTML + CSS + JS | ≤ 40 KB uncompressed, ≤ 14 KB brotli |
| Hero image at 1920 px (AVIF) | ≤ 450 KB |
| Hero image on mobile (1080 px AVIF) | ≤ 220 KB |
| Total first-view transfer | ≤ 1.2 MB |
| LCP on 4G / mid-range phone | ≤ 2.5 s |
| Animation frame rate | 60 fps on iPhone 12 / M1, ≥ 50 fps on a 2020 mid-range Android |
| CLS | < 0.02 |
| Third-party requests | 0 by default |

Implementation notes:

- `<link rel="preload" as="image" imagesrcset="..." imagesizes="100vw" fetchpriority="high">`
  with a `media` attribute matching the `<picture>` switch, so only one hero is fetched.
- `decoding="async"` on everything except the hero (`sync` or omit for the hero).
- Mount `invite-backdrop` at parse time with `loading="eager"` but `fetchpriority="low"`, or
  decode it during the hero idle time so the reveal is instant.
- No icon fonts. No animation library. No `backdrop-filter`.
- Serve with Brotli and long cache headers on hashed asset names.

---

## 15. Acceptance criteria

A build is done when all of these pass.

**Functional**

- [ ] Tee Off plays the full sequence once, even on double-click.
- [ ] Sequence lands on the invitation in 2.8–3.0 s.
- [ ] Skip works from 300 ms and crossfades in 400 ms.
- [ ] Replay returns to hero and replays cleanly.
- [ ] Enter navigates to Minted in the same tab.
- [ ] The `REPLACE-ME` guard blocks navigation and shows the inline message.
- [ ] Back from Minted shows the invitation, not a replayed shatter.
- [ ] Refresh after reaching the invitation skips the sequence (`sessionStorage`).

**Visual**

- [ ] Hero art is sharp at 1×, 2×, and 3× DPR.
- [ ] No visible seam between shards at rest.
- [ ] Impact point matches §7 on desktop and mobile.
- [ ] Invitation text is legible over the backdrop at 320 px wide.
- [ ] Gold accent appears in no more than four places per screen.

**Responsive**

- [ ] 320, 375, 414, 768, 1024, 1440, 2560, 3840 px all render without horizontal scroll.
- [ ] Portrait art is used below 768 px; landscape above.
- [ ] Safe-area insets respected on notched phones.

**Accessibility**

- [ ] Full keyboard path: Tee Off → Skip → Enter.
- [ ] Focus ring visible on every interactive element.
- [ ] `prefers-reduced-motion` shows the invitation immediately.
- [ ] Screen reader announces the sequence and the revealed invitation.
- [ ] Lighthouse accessibility ≥ 95.

**Performance**

- [ ] Lighthouse performance ≥ 90 on mobile.
- [ ] 60 fps sequence on the reference devices.
- [ ] No layout shift when fonts swap.

---

## 16. Open decisions and optional upgrades

| Item | Current decision | Notes |
| --- | --- | --- |
| Minted URL | Placeholder `REPLACE-ME` | Replace the constant in §12 before launch |
| Font choice | Cormorant Garamond + Jost | Georgia fallback ships if fonts fail; swappable |
| Audio | None | Optional `impact.mp3` ≤ 80 KB, gesture-gated only |
| Pre-rendered video | Not used | If commissioned later, a 6–8 s 1080p H.264 can replace the sequence behind Tee Off, with the invitation still handled by this page |
| RSVP / registry links | Hosted on Minted | Do not duplicate them here |
| Analytics | None | Add only with consent, as a single deferred script |

**Do not ship** the `inspo/` JPGs. They are storyboard references at ~1/5 the required
resolution and will look soft on any modern display.
