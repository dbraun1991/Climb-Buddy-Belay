# AGENTS.md

Technical reference for coding agents working in this repository.

## What this is

Climb-Buddy-Belay ("Sicherungscheck") is a single-page browser tool that
computes a belay-partner safety matrix for a lead-climbing group from
each person's name and body weight. German/English UI (German default),
no build step, no backend.

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
- `0009` — inline vanilla-JS i18n (EN/DE), no CDN library, no locale files

If a task requires reversing one of these, say so explicitly rather than
quietly working around it — e.g. adding `localStorage` contradicts `0003`
and changes the "nothing is saved" guarantee stated in `README.md`.

## Code structure inside `index.html`

Everything is in one `<script>` block near the end of the file.

- **State**: `const persons = []` — the only piece of application state.
  Each entry is `{ name: string, kg: number }`.
- **Mutators**: `addPerson(event)` (validates + pushes), `removePerson(i)`
  (splices). Both call `render()` at the end; there is no other way state
  changes reach the DOM.
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
- **i18n**: `TRANSLATIONS` (`{ de: {...}, en: {...} }`) plus `t(key, vars)`
  for lookup/`{{var}}` interpolation, `detectLang()` (URL `?lang=` then
  `navigator.language`), and `applyStaticTranslations()`/`setLang(lang)`
  for the switcher. Static markup opts in via `data-i18n="key"` (sets
  `innerHTML`) or `data-i18n-placeholder="key"`; anything rendered by
  `render()`/`renderSummary()`/`addPerson()` calls `t()` directly instead.
  Values interpolated into a `t()` template that end up in `innerHTML`
  must be pre-escaped with `esc()` first (see `docs/adr/0009`).

Matrix semantics: **row = securing person (Sichernde), column = climbing
person (Kletternde)**. `classify(sec.kg, clim.kg)` is called with the row
person as belayer and column person as climber — do not swap the argument
order when touching this code, it's not symmetric (a cell and its mirror
image across the diagonal represent different, unrelated ratings).

## Domain rules (do not change without updating all three places)

The kg-threshold bands are duplicated in four places that must stay
consistent:

1. `classify()` in the `<script>` block (the actual logic)
2. The legend in the sidebar — `legendOk`/`legendWarn`/`legendCrit`/
   `legendDanger` in **both** `TRANSLATIONS.de` and `TRANSLATIONS.en`
3. The table in `README.md`

If thresholds or colors change, update all of these. The ≥15 kg gear-hint
(assisted-braking devices, rope-diameter note) is a separate, static
threshold check in `renderSummary()` (`hasHeavyMismatch`) and its own
copy block (`gearHint1`/`2`/`3` in both languages) — check it too when
touching the model.

## Conventions

- **Language**: the UI supports German and English via the inline
  `TRANSLATIONS` table (see `docs/adr/0009`); German remains the default
  (`detectLang()` falls back to `de` unless the browser or `?lang=`
  requests English). Every new user-facing string needs an entry in
  **both** `TRANSLATIONS.de` and `TRANSLATIONS.en` — don't hardcode a new
  string into markup or a template literal. Keep new German strings
  German and consistent in tone/formality (informal "du" is not used;
  the copy is neutral/instructional) with existing copy — see `0007`.
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

- **Simple text import ("load" a group).** Let users paste/type a plain
  text list (e.g. one `Name, kg` pair per line) to populate `persons` in
  one step, instead of adding people one at a time through the form.
  Explicitly requested as a *lighter-weight* alternative to a full
  JSON export/import round-trip like Metroviz's/OrgVisualizr's
  `file-manager.js` — import only, no export, no file dialogs, no schema
  versioning; just a textarea/paste target parsed line-by-line with the
  same validation `addPerson()` already applies per line (name required,
  30–200 kg, no case-insensitive duplicates), reporting which lines failed
  rather than rejecting the whole paste. This is a deliberate,
  user-initiated exception to
  [`docs/adr/0003`](docs/adr/0003-in-memory-state-no-persistence.md) (no
  persistence) — it's a one-time bulk-entry convenience, not storage:
  nothing is written anywhere, the pasted text just seeds `persons` the
  same as manual entry would. Doesn't need its own ADR unless the scope
  grows into export/storage territory.
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
