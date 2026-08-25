# MILO Studio — Website Project Handoff

Website for **MILO Studio** (Camilo Márquez Villegas, art director / fashion photographer,
Bogotá, Colombia — positioned as a full production studio, not a solo freelancer).

**Main file: `index.html`** + an **`/images` folder** of real image files (no more embedded
base64 — see "Images & the image gallery/slider" below for why and how this changed). No build
step, no dependencies, but deployment now requires a real host connected to this folder (GitHub +
Netlify/Vercel) rather than dragging a single file into Netlify Drop — see "Deployment" below.

## Contact / brand facts
- Studio name: MILO Studio | Lead: Camilo Márquez Villegas
- Phone/WhatsApp: +57 304 544 8900 (wa.me link in Contact section)
- Email: camilomarquezv@outlook.com
- Behance: https://www.behance.net/marquezvillegas
- Language: site is bilingual EN/ES (see i18n section below), all copy in English by default

## Page structure (in `<body>` order)
Reordered twice. First on 2026-08-24, so Editorial and Campaigns show right after the hero
(before that, About and Services came first). Reordered again on 2026-08-25 to the current order
below — Campaigns now leads (before Editorial), followed by Process, About, Services, then Team
Builder and Contact. Nav link order was left as-is through both reorders (see item 1) and no
longer matches page order. The physical `<section>` blocks were reshuffled directly in
`index.html`; none of the `<script>` blocks needed to move since all of them already live after
every section, at the bottom of `<body>` (they look up elements by ID, so section order in the
markup doesn't matter to them).
1. **Header/Nav** — logo, links (Services, Build a Team, Editorial, Studio, Start a Project),
   language toggle button (`#langToggle`). Nav link order was NOT changed in the reorder — it's
   still Services → Build a Team → Editorial → Studio, which no longer matches page order below.
2. **Hero, split into two sections** (redesigned 2026-08-24 — used to be one pinned section with
   a blur/fade transition into a floating glass-panel headline; that's gone):
   - **Logo header** (`#heroPin` → `.hero-stage`, `position:sticky`): scroll-scrubs through
     `images/site/hero-sequence/frame-01.jpg`…`frame-20.jpg` (20 JPEGs extracted from a `milo.mov`
     clip of the MILO letters flying together, all preloaded as `Image()` objects on load so
     swapping `#logoFrameImg`'s `src` is instant) across the pin's scroll range — frame 1 is
     scattered pieces, frame 20 is the assembled wordmark. Once assembled the logo just stays put;
     there's no blur/fade/scale-out. Positioned near the top of the sticky stage (`padding` on
     `.hero-stage`, not vertically centered) so it reads like a second header bar.
     **`.hero-stage` has no fixed height** — it's sized by its own content (`padding-top` +
     `padding-bottom` + the logo's natural rendered height from its `width:min(78vw,980px)`), not
     a flat `100vh`, specifically so there's no dead space below a short logo on tall viewports.
     Because of that, `.hero-pin`'s height is set in JS (`sizeHeroPin()`, in the hero-pin
     `<script>` block) to `.hero-stage`'s measured height + a fixed `HERO_SCROLL_ROOM` (420px) of
     scroll-scrub distance, re-run on resize and on the logo image's `load` event — **don't** put
     a height back on `.hero-pin` or `.hero-stage` in CSS, it'll fight this. `.hero-stage` also has
     `background:var(--paper)` — the **same** flat color as the body's own base tone — specifically
     to override the generic `section{padding:...}` rule (every `<section>` gets padding, including
     this one, which used to leave a gap between the fixed header and the sticky stage where the
     body's dot grid showed through — fixed via `.hero-pin{padding:0}`) and to blend seamlessly
     against the grid pattern drawn on `body`, since the video frames' own rendered background is a
     very close but not pixel-identical tone. The logo `<img>` also has a soft radial `mask-image`
     fading its outer ~30% margin to transparent, so its rectangular edge never reads as a
     hard-edged box regardless of tiny per-frame color variance, and carries explicit
     `width="1080" height="622"` HTML attributes (the frames' native size) so the browser reserves
     the right box size before the image finishes loading, avoiding layout jump. Frames are native
     1080×622 and the logo is capped at `width:min(78vw,980px)` so it's never upscaled/pixelated —
     on very high-DPI (2x+) displays it won't be quite as crisp as a native-2x asset, since 1080px
     is the hard ceiling from the source clip.
     **Auto-plays once on load** (added 2026-08-25): a visitor who lands on the page and doesn't
     scroll right away used to just see the blank/scattered first frame — looked broken. Now
     `playIntro()` animates frames 1→20 over `HERO_INTRO_MS` (1100ms) automatically as soon as the
     script runs, independent of scroll. While it's running, `update()` (the scroll handler) is a
     no-op — gated behind an `introDone` flag — so a scroll event mid-intro can't fight the timer
     and cause flicker. Once the intro finishes, if the user still hasn't scrolled
     (`window.scrollY` still ~0) the logo just stays on the assembled frame 20 rather than being
     force-synced back to frame 1 (an early version of this called `update()` unconditionally right
     after the intro, which immediately undid it at scrollY 0 — don't reintroduce that). From that
     point on, scroll drives it normally, same as before, including scrolling back up to
     re-scatter it. Two edge cases skip the animated intro and jump straight to the correct
     scroll-synced frame instead: `prefers-reduced-motion: reduce`, and `scrollY > 50` at load time
     (a refresh with restored scroll position, or landing via a `#anchor` — not a fresh top-of-page
     visit, so replaying the intro from scratch would be wrong).
   - **Headline** (`#heroContent`, plain non-pinned section right after `#heroPin`): eyebrow, `h1`,
     paragraph, CTA row, and the brand marquee — just a normal `.reveal` fade-in-on-scroll block
     (same IntersectionObserver pattern used everywhere else on the page), not tied to the logo's
     scroll-scrub at all. `#heroContent` overrides just the generic section rule's *top* padding
     (down to `clamp(20px,3vh,36px)`) so it sits close under the logo instead of double-padded.
     Also has `background:var(--paper)` (added 2026-08-25) so the body's dot grid is hidden behind
     the headline *and* the marquee (`.hero-marquee` is a child of `#heroContent`, not `.wrap`) —
     the grid resumes right after, at Campaigns, which has no such override.
     **The brand marquee is back to the CSS-driven scrolling `<span>` ticker** (`.marquee-band` /
     `.marquee-track`, `@keyframes scroll`). It was briefly a single animated GIF the user supplied
     (`images/site/brand-marquee.gif`, added then reverted same day, 2026-08-25) — looked too
     pixelated at real size once live, so it's back to the original text ticker. The GIF file is
     still in `images/site/` (unused) in case a higher-res version shows up later; nothing
     references it now.
3. **Campaigns** (`#campaigns`) — 10 brand campaigns (Nike, Adidas, Zegna, Valentino, Iceberg,
   Payless, Calvin Klein, Desigual, LCI, Ray-Ban — Fila moved out to Services' E-Commerce slide,
   see item 7). **LCI and Ray-Ban added 2026-08-25:** LCI is a set of 6 finished ad posters for
   an education program (`images/campaigns/lci/`, cover = the brand-wide "Your Creativity, Your
   Rules." shot rather than any single program poster, since it's the least tied to one specific
   course); Ray-Ban is a 3-photo Ray-Ban × Meta smart-glasses shoot (`images/campaigns/rayban/`)
   plus a hover video (see below). Both sets came in already close to the site's usual image size
   convention (~600–900px wide) but noticeably heavier (LCI ~700KB–1.4MB, Ray-Ban ~200KB) than the
   30–70KB `01.jpg` covers elsewhere — resized to 780px wide (matching the existing portrait
   covers) and re-compressed to JPEG q82, landing at 44–110KB each, in line with the rest of the
   gallery images site-wide. Went through two redesigns the same day (2026-08-25):
   first a static two-column grid of tall cards, then — per user feedback wanting "each project as
   its own scroll, showing on its own" like the Services slider — **rebuilt as a scroll-scrubbed
   single-project slider**, the same pin pattern as the hero logo / coverflow / services
   (`#campaignsPin` → `.campaigns-sticky`, height computed in JS via `sizeCampaignsPin()` as the
   sticky content's height + a fixed 1200px `CAMPAIGNS_SCROLL_ROOM`). Scrolling advances through
   all 9 in order, one at a time (`updateCampaignsScroll()`, campaigns `<script>` block near the
   bottom, right after the services slider block). Layout is a 3-column
   `.campaigns-slide{grid-template-columns:140px auto minmax(200px,300px); justify-content:center}`
   — big number + vertical dot nav (same
   `.service-bignum`/`.services-dots` pattern, renamed `.campaign-bignum`/`.campaigns-dots`), a
   `.campaign-stage` photo (`aspect-ratio:3/4`, `border-radius:6px` — carried over from the grid
   version's corner/ratio fix), then title + counter + campaign note (`projectData[key].sub`,
   bilingual). Clicking the stage photo opens the usual project modal. All 9 slides reuse the
   *same* DOM elements (one `<img id="campaignImg">`, not 9 separate tiles) — `renderCampaignSlide()`
   swaps `src`/text on every index change, same technique as the services slider's single
   text-swap and the coverflow's caption swap. **Prev/next arrows** (added 2026-08-25, same day
   — user wanted a manual way to catch a campaign missed by scrolling too fast): reuse the
   existing `.arrow-btn`/`.arrow-prev`/`.arrow-next` styling, plus an `.edge-arrow` modifier
   (34px, smaller than the default 44px) since the user asked for them smaller. They're direct
   children of `.campaigns-sticky` — not of `.campaign-stage-wrap` — specifically so
   `arrow-prev{left:0}`/`arrow-next{right:0}` resolve against the *sticky* element's box (full
   viewport width, and already a positioned element via `position:sticky`) rather than against
   the ~400px-wide photo. First version had them flanking the photo directly; user asked to move
   them "all the way to the page's own edge/corner" instead, hence the reparent. The coverflow's
   arrows were moved the same way right after, for consistency between the two — see item 4's
   note below. `goToCampaign()` wraps around at both ends (index 8 → next → 0). Same "override
   until the next scroll event" behavior as the coverflow's own arrows/dots — clicking doesn't
   move the scroll position, so continued scrolling picks up from wherever raw scroll progress
   maps to. **Composition centered (2026-08-25):** the text column used to be a plain `1fr`, which
   on wide viewports stretched it edge-to-edge — a short note like "Dafiti LATAM Campaign" left a
   lot of *invisible* space to its right, so the actually-visible content (number, photo, text)
   read as pushed toward the left even though the grid technically filled the row. Capped it to
   `minmax(200px,300px)` and added `justify-content:center` on `.campaigns-slide` so the now
   fixed-width three-column cluster centers as a block within `.wrap` instead of stretching.
   **Non-obvious fix needed for this layout specifically:** `.campaigns-sticky` is `display:flex`
   (so it can vertically center the slide, same as `.services-sticky`/`.coverflow-sticky`) — but
   its `.wrap` child has no explicit width, so as a flex item it sizes to shrink-to-fit instead of
   filling the sticky stage. For `.services-slide` (`140px 1fr`, description text capped at
   `max-width:52ch`) that shrink-to-fit result happens to look fine; for `.campaigns-slide`'s 3
   columns, the extra `auto` track (the aspect-ratio'd photo) skewed the shrink-to-fit math hard
   enough that the `1fr` text column collapsed to ~167px — title and note wrapped into a narrow
   ragged column. Fixed with a scoped `.campaigns-sticky > .wrap{width:100%}` (not touched
   site-wide, to avoid disturbing services/coverflow which already look correct as-is). **How to
   apply:** if another pinned/flex-centered slide layout ends up with 3+ grid columns where one
   has an intrinsic (non-1fr) size, check the text column's actual rendered width — don't assume a
   working 2-column sticky pattern generalizes cleanly to 3 columns.
   **Hover video preview on Nike/Adidas/Zegna/Iceberg/Valentino/Payless/Ray-Ban** (Nike/Adidas/
   Zegna added 2026-08-25, carried through both redesigns; the rest added later the same day):
   `#campaignHoverGif`, `position:absolute; inset:0; opacity:0` inside `.campaign-stage`,
   cross-fading to `opacity:1` on `.campaign-stage:hover`. `renderCampaignSlide()` only points its
   `src` at `images/campaigns/<key>/hover.gif` (and un-hides it) for keys in the
   `campaignHoverKeys` Set — `display:none` and `removeAttribute('src')` for the rest, since
   setting `src=""` on an `<img>` makes it re-request the current page URL, a real gotcha. Source
   was BTS `.mov` clips the user supplied one at a time (`nike.mov`, `adidas.mov`, `zegna.mov`,
   `iceberg.mov`, `valentino.mov` — all B&W, portrait originals; Iceberg natively 1440×1920,
   already exactly 3:4, Valentino 1080×1920/9:16, cropped to 3:4 by the stage's own
   `object-fit:cover` same as any other campaign photo); each converted to GIF with Pillow/OpenCV
   since there's no ffmpeg on this machine. First pass (full frame count, 480px wide, dithered)
   came out to 10–14MB *each* — GIF compresses photographic/video content far worse than
   flat-color content. Settled recipe: 260px wide, ~10fps (every 3rd–5th source frame depending on
   original fps), a single 64-color palette sampled from a mid-clip frame, and **no dithering**
   (`dither=Image.NONE` — dithering adds per-pixel noise that LZW/GIF compresses badly; a flat-ish
   quantized image compresses much better and these are B&W BTS clips so banding isn't very
   visible at this size anyway). Nike/Adidas/Zegna capped to a straight 3.5s from the start of the
   clip (0.6–1.5MB each). **Iceberg and Valentino re-cut 2026-08-25** per feedback that both felt
   short and Valentino's opening was mostly a laptop, not product: Iceberg extended to 5.5s
   (close to its full ~5.9s length — was cut short before showing the model against the branded
   wall, now shows it) → 1.8MB/55 frames. Valentino rebuilt from **two separate time ranges of the
   same clip concatenated into one GIF**, not a single contiguous span: ~1.8–2.5s (heels lined up
   on the table, after the opening laptop shot pans away) + ~4.5–6.15s (makeup being applied to
   the model, a completely different moment later in the same 7.8s source) → 1.1MB/24 frames. The
   extraction script builds each range's frame list separately (same per-frame resize/palette
   logic as always) and concatenates the two lists before the shared-palette quantize + GIF save
   step — the palette is still sampled from one mid-sequence frame across the *combined* list, not
   per-segment, so both halves share one 64-color palette and cut together cleanly.
   **Valentino swapped for a new source clip, same day:** user sent a different, tighter-edited
   `valentino.mov` (6.0s vs. the original 7.8s, same underlying BTS footage — heels, laptop,
   makeup, tripod — just re-cut faster) and asked to use it instead. Same two-range technique,
   re-scanned for this clip's own cut points (contact-sheet montages at 0.4s then 0.1s resolution,
   same method as the original cut): ~0–0.45s (heels, laptop small in the corner but not
   dominant — this edit never has a fully laptop-free moment, faster-paced than the original) +
   ~2.45–3.6s (makeup application, clean before the tripod swings into frame around 3.6s) →
   16 frames/0.7MB, smaller than the original re-cut since this source clip is shorter overall.
   **Payless added 2026-08-25** (a "Back to School" campaign clip — kids running, product
   close-ups, no awkward segment to cut around like Valentino's laptop) at the same 5.5s length as
   Iceberg → 2.6MB/55 frames, the largest of the B&W clips since it's a much busier, more varied
   clip (more scene changes compress worse under GIF/LZW than a mostly-static product shot does —
   tried a 4.5s cut too, only saved ~300KB, so kept the fuller 5.5s). **Ray-Ban added 2026-08-25**
   alongside the LCI/Ray-Ban campaign additions (item 3 above) — first *color* hover clip on the
   site (all the others are B&W BTS footage): a subway-billboard shot of the Ray-Ban × Meta ad
   cycling between its day and night (Transitions lens) versions, trains blurring past behind it.
   Same recipe, but bumped the palette from 64 → 96 colors (`quantize(colors=96, ...)`) since flat
   B&W tolerates a small palette far better than skin tones and sky gradients do — went straight
   to 96 rather than starting at 64 and finding out the hard way. 5.0s, straight cut from the
   start (no bad segment to avoid, unlike Valentino) → 1.2MB/50 frames. The files live at
   `images/campaigns/<key>/hover.gif` alongside each project's `01.jpg`. **To add another:** drop
   the clip anywhere, run the same OpenCV/Pillow recipe (single range, or multiple concatenated
   ranges if only part of the clip is usable), copy the result to
   `images/campaigns/<key>/hover.gif`, and add `<key>` to `campaignHoverKeys` in the campaigns
   `<script>` block — that's the only JS change needed.
4. **Editorial** (`#work`) — 3D "coverflow" carousel (CSS `rotateY`/`translateZ`, no library) of
   7 magazine covers (Penida, Elegant, Imirage, Shuba, Scorpio Vin, MOB, Tag). Click a side cover
   to center it; click the centered one to open the project modal. The section head only has the
   eyebrow+`h2` now (the `.desc` paragraph that used to sit next to it — `editorial.desc` — was
   removed 2026-08-25, user found it redundant next to the `.lead` paragraph in
   `.editorial-intro` right below; the translation key is gone too, not just unused). **Scroll-scrubbed** (added
   2026-08-25, same pattern as the hero logo): the carousel itself sits in `.coverflow-sticky`
   inside a `#coverflowPin` wrapper, and while that's pinned, scrolling advances the centered
   cover from Penida straight through to Tag (`updateCoverflowScroll()`, in the coverflow
   `<script>` block — maps scroll progress within the pin directly to a cover index, 0 to 6).
   Same JS-computed-height approach as the hero (`sizeCoverflowPin()`): the pin's height is the
   sticky content's own height plus a fixed `COVERFLOW_SCROLL_ROOM` (520px), not a flat vh, so
   there's no dead scroll space if the carousel is short relative to the viewport. Clicking a
   cover or a dot still works at any time and simply overrides `active` until the next scroll
   event recomputes it from scroll position. **Arrows moved to the page edges (2026-08-25)** to
   match the campaigns slider, added right after that slider got its own edge arrows — `#prevBtn`/
   `#nextBtn` are now direct children of `.coverflow-sticky` (`.arrow-btn arrow-prev/next
   edge-arrow`) instead of `.coverflow-stage`, same reasoning as the campaigns slider: `left:0`/
   `right:0` needs a full-viewport-width positioned ancestor to land at the actual page edge, not
   the narrower carousel stage. `.coverflow-stage` keeps `position:relative` regardless — its
   `::before` ground-shadow pseudo-element still needs it, only the arrows moved out.
5. **Process** (`#process`) — 4-step process (Briefing → Team Assembly → Production → Retouch &
   Deliver). Each `.process-row` inverts on hover (added 2026-08-25) — dark `var(--ink)`
   background, title goes `var(--paper)`, the number goes `var(--accent)`, description goes
   `var(--mist)`. The row uses a negative-margin/matching-padding trick
   (`margin:0 clamp(-20px,-3vw,-32px)` cancelling `padding:1.7rem clamp(20px,3vw,32px)`) so the
   hover background bleeds full-width to `.process-list`'s own edge instead of stopping at the
   row's normal content width.
6. **About** (`#about`) — dark section, studio bio + 3 stats, B&W behind-the-scenes photo.
   **Stats count up on scroll (added 2026-08-25):** `#statRow`'s three `.stat-num` spans
   (`data-target="10+"`, `"20+"`, `"2"`) start at `"00"` in the HTML and animate up via
   `requestAnimationFrame` (ease-out cubic, 900ms) the first time `#statRow` scrolls into view — a
   dedicated `IntersectionObserver` (threshold 0.12, same as the site's generic `.reveal` one, but
   separate since this one runs a counting animation instead of toggling a class). The three are
   staggered 250ms apart (`setTimeout(() => animateCount(el, 900), i * 250)`) rather than starting
   together, so they land in sequence — 10+ first, then 20+, then 2 — instead of all finishing at
   once. Numbers pad to 2 digits mid-count (`"07"`, not `"7"`) and snap to the exact original text
   (`"10+"`, with the `+`) on the final frame rather than relying on the animated math to land
   precisely on it. Skips straight to the final values under `prefers-reduced-motion: reduce`.
   **Gotcha hit while testing:** `requestAnimationFrame` doesn't advance in a backgrounded/
   unfocused browser tab (Chromium throttles rAF for hidden tabs) — a test that scrolls the stat
   row into view but doesn't front the tab will see the counters stuck at `"00"` even though the
   code is correct; front the tab (or check `document.hidden`) before concluding a counting
   animation isn't firing.
7. **Services** (`#services`) — redesigned 2026-08-25 from a static 4-card grid into a
   scroll-scrubbed slider (same pin pattern as the hero logo and editorial coverflow —
   `#servicesPin` → `.services-sticky`, height computed in JS via `sizeServicesPin()` as the
   sticky content's height + a fixed 700px `SERVICES_SCROLL_ROOM`). Scrolling advances through the
   4 services in order (Retouching → Photography → Creative Direction → E-Commerce), each showing
   a big number, a counter (`01 / 04`), title, description, and dot nav (click a dot to jump).
   Text content comes straight from the existing `service1..4.title/.desc` translation keys via
   `t()`, not `data-i18n` (the slide is JS-rendered, like the project modal). **Mostly text-only,
   one photo (added 2026-08-25):** briefly had a `#serviceVisual` placeholder box for all 4 slides,
   removed the same day since the user decided text-only reads fine — but the E-Commerce & Catalog
   slide (`service4`) got a real photo back when Fila moved out of the Campaigns slider into here
   (per user feedback — Fila's shoot reads as catalog/e-commerce work, not a brand campaign;
   reuses the same `images/campaigns/fila/01.jpg` file, no new image asset needed). `.services-slide`
   is a 3-column grid (`140px auto 1fr`) where the middle `#serviceVisual` column is `display:none`
   by default and collapses to ~0 width on its own — `servicesData[i].img` (only set for
   `service4`) toggles it to `display:block` and points `#serviceImg`'s `src` at it in
   `renderServiceSlide()`. Clicking it opens the project modal via `servicesData[i].projectKey`
   (`'fila'`), same `openProjectModal()` used everywhere else. **Height gotcha:** `.services-slide`
   carries an explicit `min-height:min(46vh,380px)` matching `.service-visual`'s own height, so the
   sticky content is the *same total height* whether or not the current slide has a photo —
   `sizeServicesPin()` only measures once at load/resize, not per slide, so without this a taller
   image-slide appearing later would desync the scroll math (verified: `.services-sticky`'s
   `offsetHeight` is identical — 964px at 1280×800 — across all 4 slides).
   **Before/after drag slider on Retouching (`service1`, added 2026-08-25):** user first sent a
   Knight Lab Juxtapose `<iframe>` embed and wanted it "at a lower resolution so it loads fast" —
   pointed out that's not actually possible with a third-party embed (no control over what
   resolution Knight Lab serves), so built a native equivalent instead:
   `#serviceCompare`/`#compareBefore`/`#compareAfter`/`#compareHandle`, same show/hide pattern as
   `#serviceVisual` (`display:none` by default, `servicesData[0].compareBefore`/`.compareAfter`
   toggle it). Both images are full-bleed and stacked (`position:absolute; inset:0`, both
   `object-fit:cover`); only `#compareBefore`'s `clip-path: inset(0 <100-pct>% 0 0)` changes as
   the handle moves — deliberately *not* the more common technique (a width-`%`-clipped wrapper
   around a *fixed-pixel-width* inner image sized to match the full container), which needs a
   resize listener to keep the inner image's pixel width in sync with the container or the two
   photos drift out of alignment. `pointerdown`/`pointermove`/`pointerup` (unified mouse+touch,
   `setPointerCapture` so drags keep tracking even if the pointer leaves the element) compute the
   pointer's `%` position across `#serviceCompare`'s own `getBoundingClientRect()` and call
   `setComparePosition(pct)`, which is also just called directly with `50` on every
   `renderServiceSlide()` so the handle always starts centered. Landscape box
   (`aspect-ratio:2/1`, `height:min(300px,38vw)`) rather than the portrait 3:4 used elsewhere,
   since the source photos are 2:1 landscape and forcing them into a portrait crop would lose most
   of the frame. Source photos (`DSG_Vikram_260403_Adidas_335_Hos before/after.jpg`, 3000×1500,
   2.6–3.9MB each) resized to 1100px wide + JPEG q82 → `images/services/retouching/before.jpg` /
   `after.jpg`, 100–148KB, in line with the rest of the site's image sizes. **Verification note:**
   the browser preview tool's `computer` click/drag actions timed out repeatedly against this
   element (a `computer`-tool-level issue, not a page freeze — `document.title` and other JS
   still responded fine via `javascript_exec` immediately after each timeout); confirmed the drag
   mechanic instead by dispatching real `PointerEvent`s (`pointerdown` → `pointermove` →
   `pointerup`) directly and checking `#compareHandle`'s resulting position, then a screenshot
   showing the handle visually at the expected spot — see
   [[feedback_browser_preview_heavy_page]] for other cases of this same tool's `computer`-action
   unreliability. The other 2 folders (`images/services/photography|creative_direction/`) are
   still empty, ready the same way if photos show up for those too. The section head's copy was
   also fixed from "Two things we
   do really well" to "Four things" — there have always been 4
   cards. Also: `#heroContent`'s padding is now `clamp(20px,3vh,36px)` on *both* top and bottom
   (was top-only) — the bottom half used to stack with `#work`'s own top padding and leave a large
   empty gap below the brand marquee.
8. **Team Builder** (`#team-builder`) — interactive: click role chips (Photographer, Stylist,
   MUA, Casting, Producer, Retoucher, Motion/Video, Set Design) to build a live "team" string.
   A **"Get a Quote"/"Cotizar" button** (`#quoteBtn`, added 2026-08-25) appears in the output box
   once at least one role is selected — clicking it fills the contact form's message textarea
   (`#cfMessage`) with the selected roles (translated, via `team.quoteMessage`), smooth-scrolls to
   `#contact`, and focuses the name field. Hidden again (`.builder-cta` without `.show`) when
   nothing's selected, so it never shows an empty quote request.
9. **Project Modal** (`#projectModal`) — shared lightbox for both editorial covers and campaign
   tiles; pulls from the `projectData` JS object; includes a Behance deep-link when we have the
   real project URL, otherwise links to the profile
10. **Contact** (`#contact`) — dark panel, email + WhatsApp, no form (previously had a form but
    it wasn't wired to anything, so it was replaced with direct contact info)
11. **Footer** — nav links + Behance + tagline. `background:var(--paper-dim)` (added 2026-08-25,
    user wanted it "a bit darker, no grid") — an explicit opaque background paints over the
    body's dot-grid pattern that otherwise shows through every section without its own
    background-color. Reuses the same token `#team-builder` already uses, rather than a new color.

## Design system
- Palette (CSS vars near top of `<style>`): `--ink:#161B22 --slate:#3C4A57 --steel:#6E7F8D
  --mist:#C7D0D6 --paper:#F2F4F5 --paper-dim:#E4E8EA --accent:#2E5C73`
- Font: Archivo (Google Fonts), weights 400–900
- Subtle dot/line grid background on `body`
- No CSS framework — hand-written, mobile breakpoints at 900px / 620px / 700px depending on
  section
- `button, input, select, textarea{font:inherit; color:inherit;}` (added 2026-08-25, right after
  the universal `*{}` reset) — form controls don't inherit page font/color by default in every
  browser; iOS Safari in particular renders an unstyled `<button>`'s text in its own default blue
  instead of the surrounding text color. Found via a real phone screenshot (`.role-chip` text was
  blue on iOS, correctly dark in every desktop browser tested). If a future button looks
  differently-colored than intended on a real device even though it looks right in desktop
  Chrome, check this reset is still in place before assuming the component's own CSS is wrong.

## Bilingual (EN/ES) system — how it works
- Every translatable static text element has `data-i18n="key"` (or `data-i18n-html="hero.h1"`
  for the one element with embedded `<em>` markup).
- A single `translations` object (in the first `<script>` block, right before the hero-pin
  script) maps `key → {en, es}`.
- `currentLang` (`'en'` | `'es'`) is a module-level JS variable, default `'en'`.
- `applyLang()` walks all `[data-i18n]` / `[data-i18n-html]` nodes and sets text/HTML, then fires
  a custom `milo:langchange` event.
- Dynamic JS-rendered content (project modal text, coverflow captions, the team-builder role
  chips/output) listens for `milo:langchange` (or reads `t()` directly) to re-render in the
  current language — see `projectData` (each entry has `sub: {en, es}` and `note: {en, es}`) and
  `roleKey()` in the team-builder script.
- To add a new translatable string: give the element `data-i18n="some.key"`, add
  `'some.key': {en:'...', es:'...'}` to `translations`.

## Images & the project gallery/slider
Images are real files under `/images`, not embedded base64 (this changed on 2026-08-24 — the
site used to be a single ~1.3MB HTML file with everything inlined; extracting the images dropped
`index.html` to ~60KB and made per-image updates trivial). Layout:

```
images/
  site/            — logo-mark.png (header AND footer, 2026-08-25 — footer-logo.jpg is unused
                      now, it had a baked-in background the user didn't want), about-bts.jpg
  editorial/<key>/ — 01.jpg, 02.jpg, ... per magazine cover (penida, elegant, imirage, shuba,
                      scorpiovin, mob, tag)
  campaigns/<key>/ — 01.jpg, 02.jpg, ... per brand (nike, adidas, zegna, valentino, iceberg,
                      payless, calvin_k, desigual, lci, rayban, fila) — fila's 01.jpg is now used
                      by the Services E-Commerce slide instead of the campaigns slider, see below
```

Clicking an editorial cover or a campaign tile opens `#projectModal`, which now renders a
**slider** (prev/next arrows, dot indicators, swipe on touch, arrow-key navigation) instead of a
single static image. Each entry in the `projectData` object (in the second `<script>` block, right
after `translations`) has an `images: [...]` array — e.g.
`nike: { images: ['images/campaigns/nike/01.jpg'], folder: 'images/campaigns/nike', ... }`.
**The slider only shows arrows/dots when a project has more than one image** — a project with a
single `01.jpg` renders as a plain photo, exactly like before.

### Adding or updating project photos
1. Drop new numbered files into the matching folder, e.g. `images/campaigns/nike/02.jpg`,
   `03.jpg`, etc. (Any web-friendly format works; JPG for photos, PNG only if you need
   transparency.)
2. Add the matching path(s) to that project's `images` array in `projectData` inside
   `index.html`, e.g. `images: ['images/campaigns/nike/01.jpg', 'images/campaigns/nike/02.jpg']`.
   The array order is the slide order. **If you delete a photo from a folder, also remove its path
   from that project's `images` array** — otherwise the slider tries to load a file that's gone
   and shows a broken image.
3. Commit and push (or re-sync, if using the Google Drive workflow below) — no other code changes
   needed.

### Google Drive as the photo source
If you keep a Google Drive folder per project (mirroring the `images/<section>/<key>/` structure)
as your working library, **don't hotlink Drive URLs directly in the live site** — Google's sharing
links aren't meant for production hotlinking (view-count limits, no real CDN caching, links can
break). Instead, treat Drive as the staging area: export/download the folder, drop the files into
the matching `images/...` subfolder in this repo, add the paths to `projectData`, and push. This
is a manual sync step for now (either you or Claude Code does it each time photos are refreshed);
automating the Drive → repo sync is possible later if it becomes a recurring chore.

## Known gaps / things not yet done
- Ferragamo was removed from the campaigns grid (no replacement photo was ever provided) — add
  back if a photo shows up.
- No campaign/editorial images have real Behance links except: Penida→Venon Bloom,
  Imirage→Red Muse, Tag→Tag FILM Editorial, Adidas→Adidas Retouch HH Global. Everything else
  falls back to the general Behance profile link. More can be added to `projectData[key].behance`
  as they're confirmed.
- **Contact form is live via Web3Forms** (added 2026-08-24, activated same day). It's the
  `<form id="contactForm">` at the bottom of `#contact`, POSTing to
  `https://api.web3forms.com/submit` via `fetch` (the last `<script>` block, right before
  `</body>`) so it shows an inline success/error message instead of redirecting. The hidden
  `access_key` input holds the real key tied to camilomarquezv@outlook.com — Web3Forms' own docs
  call this a public key safe for client-side code (it only lets someone submit *to* that inbox,
  not read from it), so it's fine committed in the HTML as-is. Verified end-to-end with a live
  test submission (`success:true` from Web3Forms). Has a honeypot field (`botcheck`) for basic
  spam filtering, per Web3Forms' own convention. If the key ever needs to change (e.g. spam
  abuse), regenerate one at [web3forms.com](https://web3forms.com) using the same email and swap
  the `value="..."`.
- Team-builder role labels are keyed off English strings in `data-role` (used internally as a
  stable identifier) — translated labels are a separate lookup (`roleKey()` + `role.*` keys in
  `translations`). Don't rename `data-role` values without updating `roleKey()`'s map.

## Cursor trail
Added 2026-08-25, inspired by a reference site the user liked (a "lasso" line following the
cursor), then refined the same day to actually look like Photoshop's lasso/marching-ants selection
outline rather than a smooth solid line. It's `#cursorTrail`, a full-viewport `<canvas>` (last
script block before `</body>`): tracks `mousemove` in a rolling ~220ms buffer and strokes ONE
dashed path (`ctx.setLineDash([5,4])`) through those points every `requestAnimationFrame`, with
`lineDashOffset` incrementing each frame so the dashes visibly crawl along the line ("marching
ants"). **The speed-responsiveness is free** — no velocity math needed — because the trail is a
fixed *time* window: a fast mouse covers more screen distance in that window, so the line is
naturally longer; a slow or stationary mouse draws a short or empty one. The canvas has
`mix-blend-mode:difference` with a white stroke, so it inverts against whatever's underneath
instead of needing per-section color logic — reads dark on light sections, light on dark ones
(About, Contact) automatically; verified this actually renders correctly on both (a screenshot
over the dark Contact section clearly showed the dashed line). Skips entirely
(`prefers-reduced-motion: reduce` or `hover: none` — touch devices) via an early `return` in the
IIFE, so it never attaches listeners or runs the draw loop there.

## Deployment
No build tooling — but since images now live as real files in `/images`, deployment needs a host
that serves the whole folder, not just Netlify Drop's single-file upload:

1. Push this folder to a GitHub repo.
2. Connect that repo to Netlify or Vercel (either works — "Import an existing project" / "Add new
   site from Git"). No build command needed; publish directory is the repo root.
3. Every push to the connected branch auto-deploys. Connect a custom domain from the host's
   dashboard once it's live.

To preview locally: open `index.html` directly, or run a tiny local server from this folder (e.g.
`python3 -m http.server 8934`) so relative `images/...` paths resolve — opening the raw file with
`file://` also works for basic checks since the image paths are relative.
