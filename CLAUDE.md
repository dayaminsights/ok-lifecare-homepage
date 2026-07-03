# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Single-file marketing homepage for OK Lifecare, a multi-brand wellness company. No build system, no package.json, no git history (plain folder). Two things exist:

- `OK Lifecare Homepage - standalone.html` — the entire site, self-contained.
- `Homepage image/` — source image assets referenced by the page.

Open it directly in a browser to view/test: `cmd /c start "" "OK Lifecare Homepage - standalone.html"`. There is no build, lint, or test command — it's a static file.

Four brands run through the page (nav, footer, brand carousel): Herbofoodceutical (green, "Food as the first medicine"), Charakayu (terracotta/amber, "Pure Ayurveda"), Sweet Bliss (pink, vegan personal care), BetterU (teal).

## Critical: this HTML file is not a normal HTML file

It's a "Bundled Page" self-extractor, not hand-editable top to bottom. Structure:

1. A small bootstrap (`<head>`/loading spinner + an async unpacking `<script>`, roughly lines 1–135).
2. `<script type="__bundler/manifest">` — one giant line: JSON object `{ uuid: { mime, data (base64), compressed } }` for every embedded asset (fonts, etc).
3. `<script type="__bundler/template">` — one giant line: a JSON string containing the *actual* page (full `<html>`, all CSS, all JS — hero carousel, product cards, canvas leaf animation, everything). This is the real content.
4. `<script type="__bundler/ext_resources">` — usually `[]`.

At runtime the bootstrap script: decodes each manifest entry to a blob URL, does `template.split(uuid).join(blobUrl)` for every uuid so asset references resolve, then **replaces `document.documentElement`** with the parsed template — in place, no iframe. This matters: relative URLs in the template resolve against the real file:// location of this .html file, not some sandboxed base.

### How to actually edit it

- **Never** use the Read tool on this file, even with offset/limit — single lines run 60k–500k+ chars and it errors or silently misbehaves. Use `Grep` (or bash `grep -ob`) to find byte offsets of target strings, and small `dd`/`awk substr` reads to inspect context.
- Make edits with a Perl slurp script, not the Edit tool: `open $fh, "<:raw", $file` inside its own `{ }` block (so `local $/` doesn't leak scope, which breaks later reads), whole-file substring substitution with `\Q...\E`, write to `.tmp`, then promote. sed/Edit choke on multi-hundred-KB single lines.
- After every edit, validate **both** script blocks still parse: `JSON.parse` the manifest and the template via a quick `node -e` check before trusting the change. It's easy to accidentally duplicate/drop a brace when splicing into the manifest object (happened once already — replacing an anchor that itself started with `{` doubled the opening brace).
- Keep a `.bak` copy before patching since there's no git here yet.

### Images: prefer relative paths over baked-in base64

Sections not yet given a real photo use an inert placeholder tag: `<image-slot id="..." placeholder="description"></image-slot>` — it has no renderer, it's just a marker of what image belongs there.

When filling one in, replace it with a plain `<img>` pointing at a **relative path into `Homepage image/`**, e.g.:
`<img src="Homepage%20image/charakayu%20full%20bleed.png" style="position:absolute;inset:0;width:100%;height:100%;object-fit:cover;opacity:0.9" alt="...">`
(percent-encode spaces — this string lives inside a JSON string inside an HTML attribute).

Never bind a `{{ mustache }}` directly into a `src=`/`href=`-style attribute that triggers a browser fetch, even for something that's legitimately reactive (e.g. "show brand X's logo when brand X is active"). The bootstrap unpack step inserts the raw template HTML as `document.documentElement` via `DOMParser` *before* the component framework's own render pass hydrates it — so the browser's very first paint sees the literal unresolved text (e.g. `src="{{ brand.logo }}"`) and immediately 404s trying to fetch a file literally named that, which the global `error` listener catches and permanently displays in the `#__bundler_err` box (it's append-only, never cleared, so the page looks broken even though the framework fixes the attribute a moment later). This is why every other reactive image in this template uses either an inert `<image-slot>` placeholder or N static per-branch `<img>` elements cross-faded via the same `opacity: P.active===i?1:0` style getter already used for the hero photo slides (`s.slide0..slide3`) — reuse that pattern (a small `position:relative` wrapper with N absolutely-positioned children) for anything that must switch a real image per active index, rather than a single element with a mustache in `src`.

Do **not** embed new images as base64 into the manifest. Because the template swaps in as the live `documentElement` (not an iframe), relative paths resolve fine, and referencing the file means overwriting the PNG on disk updates the live page instantly — no re-patch needed. Baking base64 in freezes a snapshot; editing the source image afterward does nothing until you re-run the patch, which is exactly the mistake to avoid.
