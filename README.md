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
   - **Headline** (`#heroContent`, plain non-pinned section right after `#heroPin`): eyebrow, `h1`,
     paragraph, CTA row, and the brand marquee — just a normal `.reveal` fade-in-on-scroll block
     (same IntersectionObserver pattern used everywhere else on the page), not tied to the logo's
     scroll-scrub at all. `#heroContent` overrides just the generic section rule's *top* padding
     (down to `clamp(20px,3vh,36px)`) so it sits close under the logo instead of double-padded.
3. **Editorial** (`#work`) — 3D "coverflow" carousel (CSS `rotateY`/`translateZ`, no library) of
   7 magazine covers (Penida, Elegant, Imirage, Shuba, Scorpio Vin, MOB, Tag). Click a side cover
   to center it; click the centered one to open the project modal.
4. **Campaigns** (`#campaigns`) — grid of 9 brand campaign tiles (Nike, Adidas, Zegna, Valentino,
   Iceberg, Payless, Calvin Klein, Desigual, Fila), same modal on click
5. **Services** (`#services`) — 4 service cards (Retouching, Photography, Creative Direction,
   E-Commerce)
6. **About** (`#about`) — dark section, studio bio + 3 stats, B&W behind-the-scenes photo
7. **Team Builder** (`#team-builder`) — interactive: click role chips (Photographer, Stylist,
   MUA, Casting, Producer, Retoucher, Motion/Video, Set Design) to build a live "team" string
8. **Process** (`#process`) — 4-step process (Briefing → Team Assembly → Production → Retouch &
   Deliver)
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
