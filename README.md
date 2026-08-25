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
Reordered 2026-08-24 so Editorial and Campaigns show right after the hero — before that, About
and Services came first.
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
3. **Editorial** (`#work`) — 3D "coverflow" carousel (CSS `rotateY`/`translateZ`, no library) of
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
   event recomputes it from scroll position.
4. **Campaigns** (`#campaigns`) — grid of 9 brand campaign tiles (Nike, Adidas, Zegna, Valentino,
   Iceberg, Payless, Calvin Klein, Desigual, Fila), same modal on click. **Redesigned 2026-08-25**
   as a two-column grid of tall cards (`.campaigns-grid{display:grid; grid-template-columns:repeat(2,1fr)}`),
   referencing orionix.framer.website's work grid — photo on top in a `.campaign-media` box
   (`aspect-ratio:3/4`, `border-radius:6px` — user first got 18px radius + a tighter/vh-based
   height and asked for a straighter 3:4 portrait crop with less-rounded corners and more room
   between cards, hence `aspect-ratio` over a `clamp()` height and `gap:64px 48px` on the grid),
   brand name + campaign note below in a plain `.campaign-info` row (the note text comes straight
   from each `projectData` entry's `sub` field via `renderCampaignInfo()`, re-run on
   `milo:langchange` same as the other JS-built bilingual bits — not a `data-i18n` attribute,
   since it's per-brand copy, not a site-wide key). This replaced the earlier flex-grow
   hover-expand layout (tiles growing into their row on hover) — doesn't make sense with
   fixed-size grid cards, so the whole tile now just lifts slightly (`translateY(-4px)` + bigger
   shadow) on hover instead. Single column on mobile (`max-width:700px` breakpoint, `gap:44px`).
   **Hover video preview on 3 tiles** (Nike, Adidas, Zegna — added 2026-08-25, kept as-is through
   the redesign above): each of those three has a second `<img class="tile-hover-gif"
   loading="lazy">` inside `.campaign-media`, `position:absolute; inset:0; opacity:0`,
   cross-fading to `opacity:1` on `.campaign-tile:hover` (plain single-level hover, no specificity
   trap). Now shows much bigger than before since the card itself is taller. Source was 3 BTS
   `.mov` clips the user supplied (`nike.mov`, `adidas.mov`, `zegna.mov` — all B&W, portrait,
   30–60fps originals); converted to GIF with Pillow/OpenCV since there's no ffmpeg on this
   machine. First pass (full frame count, 480px wide, dithered) came out to 10–14MB *each* — GIF
   compresses photographic/video content far worse than flat-color content. Re-encoded at 260px
   wide, ~10fps (every 3rd–5th source frame depending on original fps), capped to 3.5s, a single
   64-color palette sampled from a mid-clip frame, and **no dithering** (`dither=Image.NONE` —
   dithering adds per-pixel noise that LZW/GIF compresses badly; a flat-ish quantized image
   compresses much better and these are B&W BTS clips so banding isn't very visible at this size
   anyway). Landed at 0.6–1.5MB each. The files live at `images/campaigns/<key>/hover.gif`
   alongside each tile's existing `01.jpg`.
5. **Services** (`#services`) — redesigned 2026-08-25 from a static 4-card grid into a
   scroll-scrubbed slider (same pin pattern as the hero logo and editorial coverflow —
   `#servicesPin` → `.services-sticky`, height computed in JS via `sizeServicesPin()` as the
   sticky content's height + a fixed 700px `SERVICES_SCROLL_ROOM`). Scrolling advances through the
   4 services in order (Retouching → Photography → Creative Direction → E-Commerce), each showing
   a big number, a counter (`01 / 04`), title, description, and dot nav (click a dot to jump).
   Text content comes straight from the existing `service1..4.title/.desc` translation keys via
   `t()`, not `data-i18n` (the slide is JS-rendered, like the project modal). **No photo column**
   — briefly had a `#serviceVisual` placeholder box, but the user decided text-only reads fine and
   had it removed the same day; `.services-slide` is a 2-column grid now (`140px 1fr` — number
   column + info column). The empty `images/services/<key>/` folders (`retouching`,
   `photography`, `creative_direction`, `ecommerce`) are still there in case photos come back
   later — see the comment at the top of the `services` `<script>` block (right after the
   coverflow one) for how to restore the 3-column layout + `<img>` if so. The section head's copy
   was also fixed from "Two things we do really well" to "Four things" — there have always been 4
   cards. Also: `#heroContent`'s padding is now `clamp(20px,3vh,36px)` on *both* top and bottom
   (was top-only) — the bottom half used to stack with `#work`'s own top padding and leave a large
   empty gap below the brand marquee.
6. **About** (`#about`) — dark section, studio bio + 3 stats, B&W behind-the-scenes photo
7. **Team Builder** (`#team-builder`) — interactive: click role chips (Photographer, Stylist,
   MUA, Casting, Producer, Retoucher, Motion/Video, Set Design) to build a live "team" string.
   A **"Get a Quote"/"Cotizar" button** (`#quoteBtn`, added 2026-08-25) appears in the output box
   once at least one role is selected — clicking it fills the contact form's message textarea
   (`#cfMessage`) with the selected roles (translated, via `team.quoteMessage`), smooth-scrolls to
   `#contact`, and focuses the name field. Hidden again (`.builder-cta` without `.show`) when
   nothing's selected, so it never shows an empty quote request.
8. **Process** (`#process`) — 4-step process (Briefing → Team Assembly → Production → Retouch &
   Deliver). Each `.process-row` inverts on hover (added 2026-08-25) — dark `var(--ink)`
   background, title goes `var(--paper)`, the number goes `var(--accent)`, description goes
   `var(--mist)`. The row uses a negative-margin/matching-padding trick
   (`margin:0 clamp(-20px,-3vw,-32px)` cancelling `padding:1.7rem clamp(20px,3vw,32px)`) so the
   hover background bleeds full-width to `.process-list`'s own edge instead of stopping at the
   row's normal content width.
9. **Project Modal** (`#projectModal`) — shared lightbox for both editorial covers and campaign
   tiles; pulls from the `projectData` JS object; includes a Behance deep-link when we have the
   real project URL, otherwise links to the profile
10. **Contact** (`#contact`) — dark panel, email + WhatsApp, no form (previously had a form but
    it wasn't wired to anything, so it was replaced with direct contact info)
11. **Footer** — nav links + Behance + tagline

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
  site/            — logo-mark.png (header), hero-wordmark.png, about-bts.jpg, footer-logo.jpg
  editorial/<key>/ — 01.jpg, 02.jpg, ... per magazine cover (penida, elegant, imirage, shuba,
                      scorpiovin, mob, tag)
  campaigns/<key>/ — 01.jpg, 02.jpg, ... per brand (nike, adidas, zegna, valentino, iceberg,
                      payless, calvin_k, desigual, fila)
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
