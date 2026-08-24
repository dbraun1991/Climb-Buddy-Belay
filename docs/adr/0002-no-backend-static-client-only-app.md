# 0002. No backend — static, client-only app

Date: 2026-08-24

## Status
Accepted

## Context
The tool's entire job is arithmetic on numbers the user just typed in
(weight differences bucketed into four bands) plus rendering a table.
Nothing about that requires a server: there is no shared state between
users, no data that outlives a single session worth storing centrally, and
no computation too expensive for a browser.

## Decision
Climb-Buddy-Belay has no backend. It is a static site — HTML/CSS/JS served
as-is — with all logic (validation, classification, rendering) executed
client-side in the browser.

## Consequences
- Deployable to any static host (see [0006](0006-github-pages-hosting.md))
  with no server process to run, monitor, or pay for.
- No network calls means the tool works offline once loaded, and there is
  no API surface to secure or version.
- No possibility of server-side data aggregation, multi-device sync, or
  sharing a group's roster via a link — any such feature would require
  revisiting this decision (see [0003](0003-in-memory-state-no-persistence.md)
  for the related persistence decision).
- All validation (e.g. weight range 30–200 kg) is client-side only and
  trivially bypassable; this is acceptable because there is no server-side
  resource or shared data to protect.
