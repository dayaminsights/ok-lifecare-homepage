# Architecture

Deeper reference than CLAUDE.md — read that first for editing rules. This doc is the map of what's inside the template.

## File format (recap)

`OK Lifecare Homepage - standalone.html` is a self-extracting "Bundled Page":

```
lines 1-~135   bootstrap: loading spinner + async unpack script
line ~168      <script type="__bundler/manifest">  {uuid: {mime, data(base64), compressed}, ...}
line ~177      <script type="__bundler/template">   JSON string of the real <html> document
line ~172      <script type="__bundler/ext_resources">  []  (unused so far)
```

Unpack script: blob-ifies manifest assets → `template.split(uuid).join(blobUrl)` for each uuid → `document.documentElement.replaceWith(parsed template)`. Runs in the real page, no iframe — relative URLs resolve against the actual file:// path.

## Page sections (inside the template)

In source order, as built so far:

1. **Hero carousel** — 4 full-bleed slides, one per brand, cross-fading background photo. The dark scrim (92deg side + bottom-to-top gradients) was removed 2026-07-03 for all 4 slides (user wants original photo brightness, not darkened) and legibility was moved to `const textTone` (array indexed by `P.active`, defined right before `heroCta` in the same script) → `{ink, tag, body, shadow, stroke}`, consumed by `s.headline`/`s.tagline`/`s.bodyText` getters on the `<h1>`/tagline `<div>`/`<p>`. That worked fine for Herbofoodceutical and BetterU (dark ink + light shadow, no scrim). **Charakayu did not** — three escalating attempts (darker tag color; fully-opaque unified ink + stronger halo; halo + `WebkitTextStroke`) all still left its text unreadable against that specific photo, so its scrim was put back (same `rgba(26,17,7,...)` de-tinted values as before removal) and its `textTone[1]` reverted to the plain light cream/gold scheme (`ink:'#F5EFE2', tag:'#E7CF9A'`, no stroke). So right now Charakayu is the one slide still using a scrim + light text; the other three are scrim-free with per-photo dark ink. **Don't "clean this up" for consistency** — it's load-bearing, see [[feedback-hero-image-treatment]]. Backgrounds are `<image-slot id="hero-bg-2">` for Sweet Bliss (no photo yet), plain relative-path `<img>` for the 3 that have one.
   - When Sweet Bliss gets a real photo: try the no-scrim + dark-`textTone` approach first (like Herbo/BetterU), but if it turns out unreadable the way Charakayu was, the fallback that's proven to work is scrim-back + light text, not more color/shadow iteration.
   - Small dot row (4 buttons, reusing `s.tile0..3`) plus left/right chevron arrow buttons (glass-style, matching nav) drive manual navigation; autoplay still rotates every `ROTATE_MS`. The old boxed brand-name tiles and, later, an "editorial rail" restyle of them, and separately a per-brand logo chip above the headline, were all tried and rejected — see [[feedback-hero-image-treatment]] before touching this area again.
2. **Nav bar** — tints its background by blending the active brand's `accent` via `color-mix()`, crossfading `.6s` as the hero rotates. Base alpha went `0.28 → 0.14` (glassmorphism request) `→ 0.62` (2026-07-03: user reported nav text unreadable against some brand tints/photo colors showing through the blur — a single fixed dark text color can't stay legible against 4 different, unpredictable translucent backgrounds, so the fix was to keep the surface reliably light instead of chasing text color per brand; nav text also got a light `text-shadow` halo as a second safety net). Don't drop the alpha back down for a "more glassy" look without re-checking legibility across all 4 brands.
   - Logo (`Homepage image/oklifecare logo.png`, circular-cropped) is absolutely centered (`left:50%;top:50%;translate(-50%,-50%)`). Left side (in flow) has Home/Categories/Bestsellers/Our Story; right side has Search/Cart/Login as icon+label pills (SVG icon + text, `href="#search"`/`#cart`/`#login"` are placeholders, no real functionality) — nav's `justifyContent` is `'space-between'` so the two flow-groups sit at opposite edges while the logo floats independently centered.
   - A full 40/60 (text-panel/photo-pane) Apple-style hero rebuild was tried 2026-07-03 and **rejected** ("this is bad") — reverted same session. See [[feedback-hero-image-treatment]] before attempting anything similar again.
3. **Category tiles** — 4 tiles (`cat-cosmetics`, `cat-supplements`, `cat-personalcare`, `cat-wellness`), each an `image-slot`.
4. **Bestsellers grid** — product cards, each with an `image-slot` (`best-1`..`best-4`) plus brand tag / name / rating placeholder / price row.
5. **Footer** — brand list, company links, canvas-based falling-leaf animation (`leafPalette()` — deliberately mixes all four brand hues to echo the OK Lifecare mark).

## Intro splash

`<!-- CINEMATIC INTRO OVERLAY -->` (footer of the template, rendered fullscreen on load): ivory radial-gradient canvas + the OK Lifecare logo (`Homepage image/oklifecare logo.png`, 300px, `mix-blend-mode:screen` since it's not transparent), then fades to the hero. `runIntro()` drives it with the existing `tweenOpacity` rAF helper for opacity — the scale/drift entrance (`scale(0.72) translateY(18px) → scale(1) translateY(0)`) runs in parallel via a plain CSS `transition:transform` on the logo wrapper, triggered by one JS line right after the opacity tween starts, rather than extending `tweenOpacity` itself (that helper is shared with the overlay's own fade-out, didn't want to change its signature for a one-off).

## Class-based JS controller

A single class in the template script drives: hero autoplay/crossfade (`goTo`, `cycleStart`), resize handling, product-card hover lift tween (`liftProdCard` — manual rAF tween, not CSS transition, "for reliability across render harnesses" per its own comment), and the leaf canvas. It attaches `onResize`/watchdog/rAF handles it tears down in a cleanup path — if you add new animated elements, wire teardown the same way or they'll leak on re-entry.

## Brand color key

| Brand | Accent | Category tag |
|---|---|---|
| Herbofoodceutical | `#2E8F68` green | Preventive Nutrition |
| Charakayu | `#C98B52` terracotta | Pure Ayurveda |
| Sweet Bliss | `#D29AA5` pink | Vegan Personal Care |
| BetterU | teal (`rgba(121,176,168,*)` in leaf palette) | — |

Base ink/background is a deep forest green (`#123328` / `#06 1A13`-ish), warm cream text on hero overlays.
