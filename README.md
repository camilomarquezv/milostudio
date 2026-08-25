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
2. **Hero** (`#heroPin`) — scroll-pinned section: giant MILO wordmark blurs/frosts on scroll,
   headline fades in over a glass panel (see hero-pin JS block)
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
- No contact form (intentionally removed — wasn't functional). If a real form backend appears
  (e.g. Formspree, a serverless function), it could replace/augment the WhatsApp block in
  `#contact`.
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
