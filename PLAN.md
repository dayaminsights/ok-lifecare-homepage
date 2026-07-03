# Plan / open items

Working roadmap for the OK Lifecare homepage. Update this as slots get filled or scope changes — it's the fast way to resume without re-scanning the whole template.

## Image slots — status

Hero (full-bleed, 4 brand slides — the 2026-07-03 40/60 redesign attempt was rejected and reverted, see ARCHITECTURE.md):
- [x] `hero-bg-1` Charakayu — `Homepage image/charakayu full bleed.png`
- [x] `hero-bg-0` Herbofoodceutical — `Homepage image/herbofoodceutical full bleed.png`
- [ ] `hero-bg-2` Sweet Bliss — full-bleed personal care photo
- [x] `hero-bg-3` BetterU — `Homepage image/betterU full bleed.png`

Category tiles — done 2026-07-03, rebuilt as a 6-tile (3-col x 2-row) full-bleed grid, real Unsplash stock photos (free-license `images.unsplash.com`, not Unsplash+/premium) in `Homepage image/category-*.jpg`:
- [x] Ayurvedic Cosmetics, Health Supplements, Personal Care Essentials, Herbal Wellness Drinks (original 4)
- [x] Hair & Scalp Care, Immunity Boosters (2 new, added to round out to 6)

Bestsellers — rebuilt 2026-07-03 as an auto-scrolling marquee (`.ok-prod-track`, CSS `@keyframes okProdScroll`, pauses on hover), 15 products across Herbofoodceutical/Charakayu/BetterU only (Sweet Bliss excluded from this section per user's own scope). Product **names** are real where sourced from the brand's own hero photo labels (Curcumin95/Brain Fuel/Vitamin Fuel/Ginseng Fuel/Omega Fuel from the Herbofoodceutical bottle shot; Muktavati/Guggul/Calculi Free/Miko66/Arjun+Cordyceps from Charakayu's; Purifying Face Wash/Radiant Glow Night Cream/Nano-Emulsion Body Lotion from BetterU's, plus 2 invented on-brand extensions). Prices/ratings are still placeholder (`★★★★★ (placeholder)`, made-up ₹ values) — real product data could not be scraped from shop.oklifecare.com, see note below.
- [ ] `best-1` through `best-15` (some `-b` suffixed for the marquee's duplicate loop) — all still `<image-slot>` photo placeholders.

**shop.oklifecare.com is JS-rendered** — its product catalog loads via client-side AJAX after page load, so `WebFetch` only ever sees the empty pre-render shell (confirmed on the homepage, `/product-list`, and `?cid=N` category query params for Charakayu/BetterU/Herbofoodceutical). No product names/prices/photos are recoverable that way. If asked again to pull real product data from that site, either ask for direct individual product-page URLs (might be server-rendered differently) or ask the user to paste the data — don't re-attempt the same fetch approach expecting a different result.

To fill a slot: drop the image in `Homepage image/`, tell Claude the filename, it swaps the matching `<image-slot>` for an `<img>` with a relative `Homepage%20image/...` src (see CLAUDE.md — never bake base64).

When the Sweet Bliss photo arrives, also flip its `textTone[2]` entry (in the template script, right before `const heroCta`) from the current light cream/gold `{ink:'#F5EFE2', tag:'#E7CF9A', ...}` to a dark-ink variant like the other three brands — the hero's dark scrim was removed 2026-07-03, so headline/tagline/body legibility now depends entirely on `textTone` matching how bright each photo actually is, and Sweet Bliss's light-text values only make sense against its current dark placeholder gradient.

`textTone` also carries a `stroke` field (`WebkitTextStroke`, threaded into all 3 style getters) — empty string for Herbofoodceutical/BetterU/Sweet Bliss, but Charakayu needed a real `0.5px rgba(255,255,255,.95)` stroke plus a much stronger multi-layer white halo shadow to actually read against its photo, even after the same dark-ink treatment made the other two legible fine. If Sweet Bliss's real photo also turns out to have a busy/bright zone right behind the text, don't assume the plain halo that worked for Herbo/BetterU will be enough — check it against the actual photo and be ready to add a stroke like Charakayu's.

## Known placeholder content (not yet real)

- Nav search/cart/login icon buttons (top right) are visual only — `href="#search"`/`#cart`/`#login"` don't go anywhere yet, no real search/cart/auth exists.
- New trust-badges section (below Bestsellers: Free Shipping / Quality / Secure / Customer Support) — copy is generic, not sourced from any real policy page.

- Bestseller cards show `★★★★★ (placeholder)` ratings and generic "Pure Herbal Extract" naming — needs real product names/ratings/prices per SKU.
- `Brands` footer links all point to `#top` — need real per-brand anchor/section or page.

## Housekeeping

- No git repo yet. Every hand-patch to the standalone HTML currently gets a sibling `.bak` — consider `git init` once the churn settles, so history replaces the `.bak`-per-edit convention.
- `OK Lifecare Homepage - standalone.html.bak` in the repo root right now is the pre-Charakayu-hero backup.
