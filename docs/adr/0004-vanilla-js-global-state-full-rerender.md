# 0004. Vanilla JS with global mutable state and full re-render

Date: 2026-08-24

## Status
Accepted

## Context
Given the single-file, zero-dependency constraint ([0001](0001-single-file-zero-dependency-architecture.md)),
a framework like React or Vue is off the table without a build step and a
CDN dependency. The UI has three interdependent regions (person list,
matrix table, summary/hints) that all derive from one array of people, and
that array changes rarely (one add/remove at a time) and never holds more
than a handful of entries in realistic use (a climbing group).

## Decision
State is a single global array (`persons`). Every mutation (`addPerson`,
`removePerson`) calls one `render()` function that rebuilds the person
list, matrix table, and summary from scratch via `innerHTML` string
templates — there is no component tree, virtual DOM, or fine-grained
reactivity.

## Consequences
- Trivial to reason about: one function (`render`) is always the source of
  truth for what's on screen; there's no risk of view state drifting from
  `persons`.
- No framework runtime to load, no CDN dependency, consistent with
  [0001](0001-single-file-zero-dependency-architecture.md).
- Full-rebuild-on-every-change is fine at the expected scale (a handful of
  people → an N×N matrix with N in the single digits to low tens); it would
  not scale to hundreds of entries, but that is not a realistic use case
  for a belay-group tool.
- User-supplied text (names) is interpolated into `innerHTML`, so the
  `esc()` helper (`index.html:711`) escaping `&<>"` is load-bearing for
  XSS safety and must be applied to every place user input is rendered —
  there is no framework auto-escaping to fall back on.
- Global mutable state and no module system means the codebase does not
  scale gracefully past its current size; growth beyond "a few UI regions
  driven by one array" would be a signal to revisit this decision (and
  likely [0001](0001-single-file-zero-dependency-architecture.md) with it).
