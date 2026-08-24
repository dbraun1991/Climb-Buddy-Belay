# 0009. Inline vanilla-JS i18n for German + English

Date: 2026-08-24

## Status
Accepted — supersedes [0007](0007-german-only-ui-no-i18n.md)

## Context
Two sibling projects in the same workspace (Metroviz, OrgVisualizr) both
support EN/DE via i18next loaded from a CDN, with strings externalized to
`locales/<lang>/translation.json`. Porting that pattern as-is into
Climb-Buddy-Belay would add an external runtime dependency (violating the
"no CDN scripts, no third-party JS" stance in `AGENTS.md`'s Stack section)
and would require fetching JSON locale files, which fails under `file://`
without a local server — breaking the "just open `index.html`" workflow
that [0001](0001-single-file-zero-dependency-architecture.md) exists to
protect.

The product need itself (an EN/DE switcher) is real and was requested
explicitly, so [0007](0007-german-only-ui-no-i18n.md) needed revisiting —
but the *mechanism* the sibling apps use isn't compatible with this app's
other load-bearing decisions.

## Decision
Localization is implemented as a single inline `TRANSLATIONS` object
(`{ de: {...}, en: {...} }`) inside `index.html`'s existing `<script>`
block, with a small `t(key, vars)` lookup/interpolation helper and a
`data-i18n` / `data-i18n-placeholder` attribute convention for static
markup. No external library, no separate locale files. Language choice is
read from `?lang=` (falling back to `navigator.language`) and written back
to the URL via `history.replaceState` on switch — never to `localStorage`
or any other storage, consistent with [0003](0003-in-memory-state-no-persistence.md).

## Consequences
- `index.html` remains the entire application; `0001` and the
  zero-external-dependency stance both hold.
- Still opens directly via `file://` with no local server required.
- Every user-facing string now lives in one `TRANSLATIONS` table instead
  of being interleaved through markup/JS — new strings must be added to
  *both* `de` and `en` there rather than hardcoded inline.
- Language is per-tab/session state carried only in the URL, matching how
  the sibling apps also treat *view* state (e.g. their own `?lang=`) as
  distinct from persisted *data* — no new persistence surface, no
  `localStorage` key added.
- Adding a third language means adding one more object to `TRANSLATIONS`
  and one more `.lang-btn` in the header — no build step, no new file.
- Unlike i18next, there's no pluralization engine, ICU message format, or
  lazy-loading — acceptable for this app's small, fixed string set (~40
  keys) but wouldn't scale gracefully to many languages or complex plural
  rules. If that need arises, revisit this decision rather than
  hand-rolling more i18n infrastructure inline.
