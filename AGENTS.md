# AGENTS.md

Technical reference for coding agents working in this repository.

## What this is

Climb-Buddy-Belay ("Sicherungscheck") is a single-page browser tool that
computes a belay-partner safety matrix for a lead-climbing group from
each person's name and body weight. German/English/French/Spanish UI
(German default), no build step, no backend.

Live: https://dbraun1991.github.io/Climb-Buddy-Belay/

## Stack

- Plain HTML5 + CSS + vanilla JavaScript (ES6+), no framework, no
  TypeScript.
- No package manager, no `node_modules`, no bundler, no transpiler.
- No external runtime dependencies (no CDN scripts, no fonts, no icon
  libraries, no third-party JS at all).
- Hosted as a static site on GitHub Pages.

There is nothing to `npm install` and nothing to build. "Running" the app
means opening `index.html` in a browser, or serving the repo root with
any static file server.

## File map

```
index.html          The entire application: markup, CSS, and JS in one file.
docs/Screenshot.png  Documentation image referenced from README.md.
docs/adr/            Architecture Decision Records (see below).
README.md            User-facing description (German), usage, live link.
LICENSE              MIT.
```

There are no other source files. Do not add a `src/`, `dist/`, or
`assets/` directory without first checking `docs/adr/0001-*.md` — the
single-file structure is a deliberate decision, not an oversight.

## Architecture decision records

Before making a structural change (new dependency, build step, backend
call, persistence, splitting the file, localization), read
[`docs/adr/`](docs/adr/README.md). Each ADR documents a decision that is
already load-bearing in the current code and the trade-off it accepted.
Relevant ones:

- `0001` — single-file, zero-dependency, zero-build architecture
- `0002` — no backend, static/client-only
- `0003` — in-memory state only, no persistence across reloads
- `0004` — vanilla JS, one global `persons` array, full re-render on change
- `0005` — the kg-threshold safety model (see below)
- `0006` — GitHub Pages hosting
- `0007` — German-only UI, no i18n layer (superseded by `0009`)
- `0008` — canvas-drawn icons, no image/SVG/icon-font assets
- `0009` — inline vanilla-JS i18n (DE/EN/FR/ES), no CDN library, no locale files
- `0010` — hand-drawn canvas PNG export (not SVG-foreignObject-to-canvas,
  which Chrome permanently taints — see the ADR before touching `renderMatrixPng`)

If a task requires reversing one of these, say so explicitly rather than
quietly working around it — e.g. adding `localStorage` contradicts `0003`
and changes the "nothing is saved" guarantee stated in `README.md`.

## Code structure inside `index.html`

Everything is in one `<script>` block near the end of the file.

- **State**: `const persons = []` — the only piece of application state.
  Each entry is `{ name: string, kg: number }`.
- **Mutators**: `addPerson(event)` (validates + pushes one), `removePerson(i)`
  (splices), `importPersons(event)` (parses the paste-import textarea,
  strict `Name, kg` per line, one pair per line — reuses `addPerson()`'s
  validation per line via `t()`'s error keys, collects per-line errors
  instead of aborting the whole paste, only clears the textarea if at
  least one line succeeded). All three call `render()` at the end; there
  is no other way state changes reach the DOM. `importPersons` is a
  one-time bulk-entry convenience, not storage — nothing is written
  anywhere, the pasted text just seeds `persons` the same as manual entry
  would, so it doesn't conflict with
  [`docs/adr/0003`](docs/adr/0003-in-memory-state-no-persistence.md).
  `personDrop(e, i)` reorders (native HTML5 drag-and-drop —
  `personDragStart`/`personDragOver`/`personDragLeave`/`personDrop`/
  `personDragEnd`, wired via inline `ondragstart`/etc. attributes on each
  `.pitem` like the rest of the app's event handlers, not
  `addEventListener`) — `persons.splice()` out then back in at the drop
  index, then `render()`. Row/column order in the matrix always follows
  `persons` array order, so reordering the sidebar list reorders the
  matrix too, for free. Mouse/trackpad only — HTML5 DnD has no built-in
  touch equivalent, so this doesn't work on mobile.
- **Domain logic**:
  - `classify(secKg, climKg)` — buckets `climKg - secKg` into
    `'ok' | 'warn' | 'crit' | 'danger'` using fixed thresholds
    (≤10 / ≤15 / ≤20 / >20 kg). This is the core safety rule; see
    `docs/adr/0005-fixed-kg-threshold-classification-model.md`.
  - `SYM` — maps each rating to its glyph (`✓ ! ⚠ ✗`).
  - `delta(secKg, climKg)` — formats the signed kg difference for display.
- **Rendering**: `render()` rebuilds the sidebar person list and the
  matrix `<table>` from `persons` via `innerHTML` string templates, then
  calls `renderSummary()` for the two plain-language sentences below the
  table and the conditional gear-hint box. There is no diffing — the
  whole relevant DOM subtree is replaced on every mutation.
- **Escaping**: `esc(s)` HTML-escapes `& < > "` before any user-supplied
  string (currently just `name`) is interpolated into an `innerHTML`
  template. **Every new place that renders user input into the DOM via
  string templates must go through `esc()`** — there is no framework
  auto-escaping here, and this is the only XSS defense in the app.
- **Decorative graphics**: `drawLogo(id)` and `drawEmpty(id)` paint the
  header carabiner icon and empty-state figure-8 device onto `<canvas>`
  elements using the Canvas 2D API. No image assets exist.
- **i18n**: `TRANSLATIONS` (`{ de: {...}, en: {...}, fr: {...}, es: {...} }`,
  languages listed in `SUPPORTED_LANGS`) plus `t(key, vars)` for lookup/
  `{{var}}` interpolation, `detectLang()` (URL `?lang=` then
  `navigator.language`, matched against `SUPPORTED_LANGS`), and
  `applyStaticTranslations()`/`setLang(lang)` for the switcher — a native
  `<select id="lang-select">` (flag emoji + code per `<option>`), not
  buttons; `setLang` just sets `.value`. Static markup opts in via
  `data-i18n="key"` (sets `innerHTML`) or `data-i18n-placeholder="key"`;
  anything rendered by `render()`/`renderSummary()`/`addPerson()` calls
  `t()` directly instead. Values interpolated into a `t()` template that
  end up in `innerHTML`
  must be pre-escaped with `esc()` first (see `docs/adr/0009`).
- **Responsive sidebar**: below 768px (`@media (max-width: 768px)`),
  `.layout` drops to a single column and the entire `<aside class="sidebar">`
  content is wrapped in `<details class="sidebar-collapse" open>` with a
  `<summary class="sidebar-summary" data-i18n="sidebarSummary">` — a real,
  user-operable disclosure widget on mobile, collapsed on demand, `open` by
  default so the add-person form is visible on first load. Above 768px
  `.sidebar-summary` is `display: none` and the `open` attribute is never
  toggled (nothing can click the hidden summary), so desktop stays visually
  and behaviorally identical to before this existed — no JS involved, pure
  CSS + native `<details>`.
- **Share/export**: `computeSummary(list)` — the derived "can belay all" /
  "can lead-climb with all" lists plus the light/heavy verdict — is the
  single source of truth used by both `renderSummary()` (DOM) and
  `renderMatrixPng()` (canvas export), so the two can never disagree.
  `renderMatrixPng()` draws the export by hand with Canvas 2D primitives
  (`drawMatrixContent()`), *not* by rasterizing the DOM — see `docs/adr/0010`
  for why (Chrome taints any canvas drawn from an SVG containing
  `foreignObject`, which is the more obvious approach and was tried first).
  `parseRichHtml()`/`wrapRichText()`/`drawRichLines()` reuse the existing
  `<strong>`/`<em>`/`<span class="rating-*">` markup already in
  `TRANSLATIONS` strings via a small trusted-content tokenizer — extending
  a translation string with a new inline tag needs a matching case added
  there too, or the export will just print the tag literally.
  `shareMatrix()` branches on `navigator.share`+`canShare` (mobile, gated
  by the same `max-width: 768px` breakpoint as the sidebar) vs.
  `navigator.clipboard.write` (desktop) vs. a plain download (fallback).
  `#btn-share` lives in `.matrix-meta`, which sits directly in `.main` —
  outside `#empty`/`#matrix-wrap` — so the button is always present and
  only its `disabled` attribute (toggled in `render()` alongside the
  `#empty`/`#matrix-wrap` visibility swap) reflects whether there's a
  matrix to export yet.
- **Footer**: `<footer class="site-footer">` — project name (untranslated,
  same as the repo name), GitHub link, MIT license link (`href="LICENSE"`,
  the file already at the repo root) — mirrors the sibling apps'
  (Metroviz/OrgVisualizr) footer pattern, adapted to this app's own
  `data-i18n`/`t()` convention instead of Alpine/i18next. `.hide-mobile`
  (project name + both `·` separators) follows the existing
  `max-width: 768px` breakpoint, matching the source pattern.

Matrix semantics: **row = securing person (Sichernde), column = climbing
person (Kletternde)**. `classify(sec.kg, clim.kg)` is called with the row
person as belayer and column person as climber — do not swap the argument
order when touching this code, it's not symmetric (a cell and its mirror
image across the diagonal represent different, unrelated ratings).

This row/column meaning is shown as two dimension-info bands *inside* the
table itself, not as separate prose above it: `axisClimbers` (`.axis-col-label`,
`<th colspan>` spanning the cell columns, sits between the column person
headers and the first cell row) and `axisBelayers` (`.axis-row-label`,
`<td rowspan>` spanning the cell rows, `writing-mode: vertical-rl` rotated
text, rendered once on the first row only — `si === 0` in `render()`'s
table-building loop). `renderMatrixPng()`'s `drawMatrixContent()` draws the
same two bands by hand (dashed `ctx.setLineDash` separators, `ctx.rotate(-Math.PI/2)`
for the vertical one) — keep both in sync if this changes.

## Domain rules (do not change without updating all three places)

The kg-threshold bands are duplicated in four places that must stay
consistent:

1. `classify()` in the `<script>` block (the actual logic)
2. The legend in the sidebar — `legendOk`/`legendWarn`/`legendCrit`/
   `legendDanger` in **all four** `TRANSLATIONS` languages
3. The table in `README.md`

If thresholds or colors change, update all of these. The ≥15 kg gear-hint
(assisted-braking devices, rope-diameter note) is a separate, static
threshold check in `renderSummary()` (`hasHeavyMismatch`) and its own
copy block (`gearHint1`/`2`/`3` in all four languages) — check it too when
touching the model.

## Conventions

- **Language**: the UI supports German, English, French, and Spanish via
  the inline `TRANSLATIONS` table (see `docs/adr/0009`); German remains
  the default (`detectLang()` falls back to `de` unless the browser or
  `?lang=` requests one of the others). Every new user-facing string
  needs an entry in **all four** `TRANSLATIONS` languages — don't
  hardcode a new string into markup or a template literal. Keep new
  German strings German and consistent in tone/formality (informal "du"
  is not used; the copy is neutral/instructional) with existing copy —
  see `0007`; keep French/Spanish similarly neutral/formal (no "tu"/"vous"
  ambiguity — impersonal or infinitive phrasing where natural, matching
  the existing FR/ES copy's register).
- **No comments explaining what code does** — the existing code has none;
  match that style. Only add a comment for a genuinely non-obvious
  constraint (the existing canvas-drawing functions have a couple of
  these for the shape math).
- **CSS**: custom properties on `:root` for the palette (`--ok`, `--warn`,
  `--crit`, `--danger`, etc.) — reuse these tokens rather than hardcoding
  new colors, especially for anything belay-rating-related.
- **No inline event handler frameworks** — the app uses plain
  `onclick="..."` / `onsubmit="..."` attributes referencing global
  functions. Stay consistent with this rather than introducing
  `addEventListener`-based wiring in some places and inline handlers in
  others.

## Planned future work (not yet implemented)

Captured from product notes, not yet designed or built. Treat as
direction, not spec — clarify open questions with the user before
implementing rather than guessing at exact behavior.

- **Two dimensions: "Lead climb" and "Belay buddy".** Introduce a second
  axis/mode alongside the current lead-climbing check. Exact semantics
  are unspecified — clarify with the user before building: is this a
  mode toggle that changes the classification model (e.g. different
  thresholds for top-rope/"buddy" belaying vs. lead climbing), a second
  independent matrix, or a per-pair relationship tag layered on the
  existing matrix? This likely also touches
  [`docs/adr/0005`](docs/adr/0005-fixed-kg-threshold-classification-model.md)
  (currently modeling lead climbing only).

## Testing / verification

There is no test suite and no CI. Verify changes by opening `index.html`
in a browser and exercising the UI directly:

- Add ≥2 people to confirm the matrix and summary render.
- Add a pair with a ≥15 kg difference to confirm the gear-hint appears.
- Try invalid input (empty name, duplicate name case-insensitively,
  weight outside 30–200) to confirm `form-err` messaging.
- Check both light behavior only — there is no dark mode / theme
  switching in this app.

If browser automation tools are available, prefer actually loading the
page over reasoning about the DOM from source alone, especially for any
visual or interaction change.
