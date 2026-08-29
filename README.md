# Photography studio site

Static site. No framework, no build step, no `node_modules`. Plain HTML, CSS
with custom properties, and a little vanilla JS. Deploys by copying the shipped
folders to any static host.

Site language: Simplified Chinese (`lang="zh-CN"`).

---

## Structure

```
.
├─ index.html                 home
├─ galleries.html             gallery index (lists the sets)
├─ about.html                 the photographer
├─ contact.html               enquiries
├─ sets/                      one thin file per gallery set
│    └─ <slug>.html           set data only; layout is shared
├─ assets/
│    ├─ css/
│    │    ├─ tokens.css       design system — colour, type, spacing, boxes
│    │    └─ site.css         reset, base type, shared components
│    └─ js/
│         └─ gallery.js       lightbox, reveal-on-scroll (enhancement only)
│    └─ img/
│         ├─ sets/<slug>/     generated WebP derivatives (shipped)
│         └─ ui/              logo, share images
│
├─ tools/                     committed, not part of the site
│    ├─ sets.map              slug | source file | orientation
│    └─ build-images.sh       generates the WebP derivatives
├─ _source/                   GITIGNORED — full-resolution originals
│    ├─ originals/            untouched camera files
│    └─ sets/<slug>/          originals grouped by set, numbered 01.jpg…
└─ .cache/                    GITIGNORED — encode cache, safe to delete
```

`_source/` is gitignored: 168MB, and git history is permanent. The committed
derivatives in `assets/img/` are all the site needs, so it builds and deploys
without the originals present — but keep them somewhere, or you cannot re-cut
derivatives at new sizes.

---

## Adding a gallery set

1. Drop the full-size JPEGs into `_source/sets/<slug>/`, numbered `01.jpg`,
   `02.jpg`, …
2. Add a line to `tools/sets.map`:
   ```
   <slug>|<any-source-file>|portrait     # or landscape
   ```
3. Run the image script, then copy any existing `sets/<slug>.html`, change the
   title, the metadata line, the description, and the image paths. All layout and
   behaviour comes from the shared CSS and JS, so the file is just content.
4. Add the set to the card grid on `galleries.html`, to the footer list on every
   page, and to the prev/next links on the neighbouring set pages.

Generate the derivatives:

```bash
FFMPEG="/path/to/ffmpeg" bash tools/build-images.sh
```

It is idempotent: existing, up-to-date outputs are skipped, and identical source
files are encoded once and copied. Widths are cut to the sizes the layout
actually requests at 1x and 2x — landscape 352 / 528 / 704 / 1056 / 1760,
portrait 264 / 396 / 528 / 792 / 1320 — plus a 32 px blur-up placeholder.
WebP, quality 82.

---

## GitHub Pages

**All paths are relative, never root-absolute.** A project site is served from
`https://<user>.github.io/<repo>/`, so `/assets/css/site.css` would resolve to
`https://<user>.github.io/assets/css/site.css` and 404. Root pages therefore link
to `assets/…` and set pages to `../assets/…`. Verified by serving the folder both
at a domain root and one level down at `/Photography/`: all nine pages, every
`srcset` candidate, and every lightbox `data-full` target return 200 in both
cases. This also means the site works unchanged on a custom domain or a
`<user>.github.io` repo — do not "simplify" these back to absolute paths.

**`.nojekyll` is deliberately absent.** Jekyll is a no-op here: no served
directory starts with an underscore, there is no Liquid syntax anywhere, and no
page has front matter, so Jekyll copies everything through untouched. Adding
`.nojekyll` would also stop Jekyll excluding `_`-prefixed directories, which is
the one thing keeping `_source/` unpublished if it ever gets committed by
accident.

**Nothing here needs an absolute URL.** `og:image` is relative like everything
else. Some scrapers resolve relative Open Graph images and some ignore them, but
ignoring one degrades to "no preview image", whereas a placeholder origin would
be a broken URL everywhere. If link previews matter later, making `og:image`
absolute is a one-line change per page.

---

## Header, footer, and why they are duplicated

The header and footer are real markup repeated in all nine pages rather than
rendered by JavaScript. Baidu does not reliably execute JS, and this is a
Chinese-language site, so navigation and footer links have to be present in the
HTML to be crawled at all. The cost is that changing the nav means editing nine
files — do it with a find-and-replace across the repo, and keep the blocks
byte-identical.

`assets/js/gallery.js` is enhancement only. Every photograph and every link is
already in the HTML; the script adds the reveal-on-scroll and the lightbox. If it
fails to load, nothing breaks.

---

## Placeholders to replace

All placeholders are bare uppercase Latin so they are impossible to miss and
trivial to grep:

`STUDIO NAME` · `GALLERY 1`–`GALLERY 5` · `PHOTOGRAPHER NAME` · `BASED IN CITY`
· `LOCATION` · `SEASON` · `TIME OF DAY` · `PHOTO CAPTION` · `EMAIL ADDRESS` ·
`WECHAT ID` · `INSTAGRAM HANDLE` · `WECHAT QR CODE` · `BUSINESS REGISTRATION` ·
`META DESCRIPTION` · `HERO HEADLINE` · `SECTION HEADLINE` · `IMAGE ALT`

Set slugs and image folder names are placeholders too (`gallery-1` …), so renaming
a set means renaming its HTML file, its `_source/sets/` folder, its
`assets/img/sets/` folder, and its line in `tools/sets.map`.

All Chinese body copy is generated filler. It is written to be plausible in tone
and length so the layout can be judged, but none of it is the client's.

---

## Design system

Everything visual is a token in `assets/css/tokens.css`. Change it there, not in
component CSS.

**Paper and ink.** The background is a warm off-white (`#F8F5F0`) in 48px gutters inside a 1200px
container, giving 1104px of content tuned to the
milky highlight that runs through the photography, so images dissolve into the
page rather than sitting in boxes. Text is a warm near-black (`#1F1D1A`), never
pure black. Contrast on paper: body 17.2:1, secondary 10.7:1, labels 6.7:1 — all
pass WCAG AA.

**Accent.** One structural accent, indigo `#2B3855` (10.7:1). Cool on purpose, so
it never competes with the warm skin tones present in every photograph. Nothing
is underlined anywhere — links carry state through colour alone, and the current
nav item is marked by a 4px ink bar 10px beneath it.

**Paper grain.** SVG fibre noise generated in-browser, so no image request. It
sits on a fixed layer behind all content, which means photographs are never
textured. Strength is `--grain-opacity`, currently `0.035`.

**Type.** One weight only — Alibaba PuHuiTi Medium (65), loaded from the client's
own repo. This is the site's only external request. Hierarchy comes from size,
letter-spacing and ink depth, never from weight; `font-synthesis: none` stops
the browser faking a bold, which looks bad on Hanzi. Body is 16px with 1.8
leading, because Hanzi need more size and air than Latin at the same optical
weight. `font-display: swap` means text paints immediately in the system face,
so the font download never blocks rendering.

**Standalone photograph sizes.** Outside a grid, every landscape image occupies
the same rectangle and every portrait image its reciprocal — 880×660 and 660×880. Equal optical
area, so neither orientation dominates. Sources already at 4:3 or 3:4 display
untouched; anything off-ratio takes a mild centre crop, steerable per image with
`--focal: <x> <y>`.

**Gallery grids.** 1104px of content keeps every density on a whole-number slot
that stays clean at 4:3:

| class | column gap | slot |
|---|---|---|
| `.gallery-grid--2` | 48px | 528 |
| `.gallery-grid--3` | 24px | 352 |
| `.gallery-grid--4` | 16px | 264 |
| `.gallery-grid--cards` | 72px | 320 |

Columns are explicit rather than `auto-fill`, so slots always match the widths the
derivatives were cut for. Every figure in a grid fills its slot regardless of
orientation, so the grid's outer edges stay flush with the page's content edges.
Grids collapse to two columns at 900px and one at 600px.

`--cards` also carries an 80px row gap, because the caption row sits between
rows and needs the extra air.

**Two figure treatments.** On set pages, `.photo--landscape` and `.photo--portrait`
keep the true source ratio and are never cropped — a portrait set renders 352×469,
a landscape set 352×264, uniform within the set.

On gallery-index cards, `.photo--cover` forces a square box instead. Mixed
orientations in one grid would otherwise give every card a different height,
push the caption rows onto different lines, and leave side gaps on portrait cards
so the grid no longer lined up with the page edges. The square box makes card
heights uniform, puts every caption row on one line, and keeps the grid flush.
The cost is a centre crop on the cover thumbnail, roughly 25% off one axis and
symmetric between the orientations, steerable per card with `--focal: <x> <y>`.
The uncropped photo is always on the set page.

---

## Local preview

Serve the folder over HTTP; opening files directly breaks the root-absolute paths.

```bash
python -m http.server 4173
```
