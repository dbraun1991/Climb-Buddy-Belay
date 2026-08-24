# 0003. In-memory state only, no persistence

Date: 2026-08-24

## Status
Accepted

## Context
The `persons` array (`index.html:508`) is the tool's only state, held in a
plain JS variable. Nothing writes it to `localStorage`, a cookie, the URL,
or any backend. The README makes this explicit: "Keine Persistenz — die
Gruppe wird nicht gespeichert und geht beim Neuladen verloren."

The tool models a *session*: a specific group of people, on a specific
day, checking who can belay whom before climbing. Climbing groups are
usually ad hoc — different people show up each session — so a saved
roster from last time is more likely to be stale or wrong than useful, and
carries a real safety cost if someone forgets to update a weight or remove
someone who left.

## Decision
Application state is kept in memory only, for the lifetime of the page
load. Reloading, closing the tab, or navigating away discards the group
with no prompt or recovery path.

## Consequences
- Simplest possible state model: no serialization, no schema migration,
  no storage-quota or private-browsing edge cases to handle.
- No stale data: every session starts from a clean slate, which fits the
  ad hoc nature of climbing groups.
- Users must re-enter the whole group every time, which is friction for
  any group that climbs together repeatedly. If that friction becomes a
  real complaint, the fix (e.g. `localStorage` with an explicit "load
  last group" action, or a URL-encoded roster) should be a deliberate
  follow-up decision, not a silent addition — it changes the "nothing is
  saved" guarantee the README currently promises.
- No accidental data retention: nothing about a group (names, weights)
  persists beyond the tab's lifetime, which is a reasonable default for
  data some users might consider personal (body weight).
