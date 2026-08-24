# 0008. Canvas-drawn icons instead of image/icon-font assets

Date: 2026-08-24

## Status
Accepted

## Context
The UI needs two small decorative graphics: a carabiner logo in the header
and a figure-8 belay-device illustration for the empty state. Options were
a bitmap/SVG asset file, an icon font/library, or drawing the shapes in
code.

## Decision
Both graphics are drawn programmatically at load time using the Canvas 2D
API (`drawLogo`, `drawEmpty` in `index.html:511-550`), targeting inline
`<canvas>` elements. No image files, SVG assets, or icon libraries are
used anywhere in the project (the only binary asset in the repo,
`docs/Screenshot.png`, is documentation, not part of the served app).

## Decision drivers
- Keeps the app at exactly one file with zero external assets, consistent
  with [0001](0001-single-file-zero-dependency-architecture.md) — no
  `assets/` folder, no asset paths that could break on a different host or
  base path.
- Avoids pulling in an icon font/library, consistent with
  [0001](0001-single-file-zero-dependency-architecture.md)'s
  zero-dependency stance.

## Consequences
- Nothing to fetch for these graphics beyond the HTML itself; no broken
  image paths, no separate caching/versioning concerns for icon assets.
- The shapes are defined as raw drawing-command coordinates in JS, which
  is harder to hand-tune visually than editing an SVG in a design tool,
  and there is no vector source file to open in external tooling.
- Any future icon needs the same treatment (hand-coded canvas/SVG
  drawing) or this decision must be revisited — e.g. if the number or
  complexity of graphics grows, static SVG markup inline in the HTML would
  likely be easier to maintain while still requiring no external asset
  files.
