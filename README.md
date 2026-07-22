# Game Dev Portfolio — Anantha Prasanth

A single-page game-development portfolio site: black-and-neon "arcade/CRT" theme, a looping YouTube hero video, scroll-driven accent colors, and detail pages for games, professional work, and Unreal Engine plugins.

**Live pages:**
- `Home.dc.html` — Home / Expertise / Games / Professional / Tools / Journey (single scrollable page, this is what `index.html` redirects to)
- `Project.dc.html?slug=<slug>` — game project detail (Cluckmare, AimGL)
- `ToolsProject.dc.html?slug=<slug>` — Unreal plugin detail (WindField)
- `ProfessionalProject.dc.html?slug=<slug>` — professional role detail (PropVR interactive walkthrough)
- `Moodboard.dc.html` — design/mood reference board (not linked from nav)

## Stack

Built on **DC** (`.dc.html`), a lightweight JSX-like templating format: HTML with `{{ }}` interpolation and `sc-for` / `sc-if` directives, paired with a `<script type="text/x-dc" data-dc-script>` block containing a `Component extends DCLogic` class. `support.js` is the generated runtime (parsing, React reconciliation, template compilation) that boots each page — React, ReactDOM, and Babel load from CDN at runtime, so there's no build step and no `package.json`.

`image-slot.js` adds a `<image-slot>` custom element for drag-and-drop image placeholders (used on the moodboard page).

## Project structure

```
Home.dc.html                 Single-page site (all sections + nav)
Project.dc.html              Game detail template (reads ?slug=)
ToolsProject.dc.html         Plugin detail template (reads ?slug=)
ProfessionalProject.dc.html  Professional role detail template (reads ?slug=)
Moodboard.dc.html            Design direction reference board
support.js                   Generated DC runtime (do not hand-edit)
image-slot.js                <image-slot> custom element
assets/
  games/<slug>/              Per-game screenshots, gifs, code shots
  plugins/windfield/         WindField plugin media
  professional/              PropVR walkthrough media
uploads/                     Raw/source media (gitignored — not published)
dist/, site/                 Earlier build/alternate variants
```

## Running locally

No build step — just serve the folder over plain HTTP (not `file://`, since the runtime does `fetch()` calls and reads `?slug=` query params) and open `Home.dc.html`:

```bash
npx serve .
# or any other static file server
```

## License

MIT — see [LICENSE](LICENSE).
