# 0001. Single-file, zero-dependency, zero-build architecture

Date: 2026-08-24

## Status
Accepted

## Context
Climb-Buddy-Belay is a small utility: enter names and body weights, get a
color-coded belay-pairing matrix. The intended use case is opening the tool
quickly — often on a phone at a climbing gym — with no setup step between
"I have the link/file" and "I can use it."

Introducing a bundler, package manager, or framework would add a build
step, a `node_modules` tree, and version-drift risk for a tool whose entire
logic fits comfortably in a few hundred lines of markup, styles, and
script.

## Decision
The entire application — markup, CSS, and JavaScript — lives in one file,
`index.html`, with no external dependencies, no package manager, and no
build/bundle step. "Install" and "run" are both satisfied by opening the
file in a browser.

## Consequences
- Zero setup: cloning the repo and opening `index.html` is sufficient; the
  README's "Nutzung" section is accurate as literally `open index.html`.
- No dependency updates, no `package.json`/lockfile to maintain, no
  supply-chain surface from third-party packages.
- Works from `file://` as well as any static host, including GitHub Pages
  (see [0006](0006-github-pages-hosting.md)).
- The file will grow harder to navigate as features are added — there is
  no module boundary between styling, markup, and logic. This is an
  accepted trade-off while the tool stays small; splitting into multiple
  files would reintroduce the setup friction this decision avoids, and
  would need a build step (or careful manual `<script src>`/`<link>`
  wiring) to reassemble into something still deployable as static files.
- Rules out any feature that inherently needs a build step (e.g. JSX,
  TypeScript, CSS preprocessors, npm-only libraries) without revisiting
  this decision first.
