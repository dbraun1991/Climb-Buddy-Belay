# 0010. Hand-drawn canvas PNG export for the share button

Date: 2026-08-24

## Status
Accepted

## Context
The share button needs to turn the matrix + research note + summary +
gear-hint (`#matrix-wrap`) into a PNG: shared via `navigator.share()` on
mobile, or written to the system clipboard via `navigator.clipboard.write()`
on desktop.

The obvious dependency-free approach — and the one first implemented — is
the well-known "screenshot a DOM node without a library" trick: serialize
the target node's `outerHTML` (plus a copy of the page's own `<style>`)
into an SVG `<foreignObject>`, load that SVG as an `<img>`, draw the image
onto a `<canvas>`, then read the pixels back out via `toBlob()`. This is
exactly what small screenshot libraries like `dom-to-image` do internally,
so it looked like a safe way to keep [0001](0001-single-file-zero-dependency-architecture.md)'s
zero-dependency stance while getting a pixel-accurate export "for free"
from the existing CSS.

It doesn't work in Chrome. Chrome (and Chromium-based browsers generally)
permanently taints any canvas that a `<foreignObject>`-containing SVG image
is drawn onto — confirmed directly during implementation:
`ctx.getImageData()` threw `SecurityError: The canvas has been tainted by
cross-origin data`, even with a same-origin `blob:` URL and zero external
resources (no fonts, no images, nothing off-origin). This is a deliberate
Chrome anti-fingerprinting policy, not a bug: it treats "rasterize
arbitrary HTML via the browser's own layout engine, then read the pixels
back out" as a fingerprinting vector regardless of origin. `canvas.toBlob()`
on a tainted canvas doesn't even throw in this Chrome version — the
callback simply never fires, which surfaced first as the export silently
hanging forever. Firefox does not taint in this situation, which is why
the technique is often described as working — it was only ever reliably
working in one browser family, and Chrome is most of this app's users.

The remaining options: (a) pull in a library like html2canvas, which
avoids the taint by never using `foreignObject` — it walks the DOM and
redraws each element with Canvas 2D primitives itself — at the cost of an
external CDN dependency; or (b) write that manual redraw ourselves.

## Decision
`renderMatrixPng()` draws the export by hand with plain Canvas 2D calls
(`fillRect`/`fillText`/`roundRect`), extending the existing `drawLogo`/
`drawEmpty` canvas-icon pattern ([0008](0008-canvas-drawn-icons-no-image-assets.md))
to the whole shareable block instead of just the header/empty-state icons.
It reads `persons` and `computeSummary()` directly rather than scraping the
DOM — the derivation logic is shared with the live `renderSummary()` (see
`computeSummary()`/`buildVerdictHtml()` in `index.html`), so the exported
image and the on-screen summary can never disagree, and the export no
longer depends on CSS/layout at all.

Inline `<strong>`/`<em>`/`<span class="rating-*">` markup already present
in the `TRANSLATIONS` strings is reused as-is via a small trusted-content
tokenizer (`parseRichHtml`) — it recognizes exactly those three patterns
this app's own templates ever produce, not general HTML, and only ever
sees translation strings plus person names that already passed through
`esc()`. A small word-wrapping engine (`wrapRichText`/`drawRichLines`)
lays the resulting styled runs out across lines at a given max width. Two
passes: draw everything once on a disposable canvas purely to measure the
total height (canvas draw calls outside its own bounds are harmless, never
error), then draw again on a correctly-sized real canvas.

## Consequences
- Stays zero-dependency and single-file — no CDN library added, no
  external font or asset loading, consistent with `0001` and `0008`.
- Works identically in Chrome, Firefox, and Safari — no reliance on
  browser-specific `foreignObject`-rasterization behavior.
- Real cost: the export's visual structure (title, research note box,
  table, summary, gear-hint) is now maintained by hand in
  `drawMatrixContent()`, separately from the HTML/CSS that renders the
  same information on screen. A structural change to `#matrix-wrap` (new
  section, reordered blocks, a new color) needs a matching update in
  `drawMatrixContent()` — the two no longer share layout code, only the
  underlying data (`persons`, `computeSummary()`) and copy (`TRANSLATIONS`).
  This is the trade `computeSummary()`'s extraction was specifically meant
  to narrow: the *numbers* can't drift, only the *pixels* can.
- `parseRichHtml`'s tokenizer is deliberately narrow (three tag patterns).
  Adding a new kind of inline markup to any translation string that also
  needs to render in the export means extending this tokenizer too, not
  just the CSS.
- No text selection, no accessibility tree, no way to zoom in the browser
  the way an HTML render would offer — it's a flat image, which is exactly
  what sharing to a phone's share sheet or a clipboard paste needs.
