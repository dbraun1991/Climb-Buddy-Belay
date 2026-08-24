# Architecture Decision Records

This directory records the architecturally significant decisions behind
Climb-Buddy-Belay, in [MADR](https://adr.github.io/madr/)-style format.

These ADRs are **retroactive**: the decisions were made during initial
development (2026-06-26) and are documented here after the fact, based on
the state of the code as of 2026-08-24. Dates on individual records mark
when the record was written, not necessarily when the decision was first
made.

## Index

| # | Title | Status |
|---|-------|--------|
| [0001](0001-single-file-zero-dependency-architecture.md) | Single-file, zero-dependency, zero-build architecture | Accepted |
| [0002](0002-no-backend-static-client-only-app.md) | No backend — static, client-only app | Accepted |
| [0003](0003-in-memory-state-no-persistence.md) | In-memory state only, no persistence | Accepted |
| [0004](0004-vanilla-js-global-state-full-rerender.md) | Vanilla JS with global mutable state and full re-render | Accepted |
| [0005](0005-fixed-kg-threshold-classification-model.md) | Fixed kg-threshold model for belay-safety classification | Accepted |
| [0006](0006-github-pages-hosting.md) | GitHub Pages for hosting and deployment | Accepted |
| [0007](0007-german-only-ui-no-i18n.md) | German-only UI, no i18n abstraction | Accepted |
| [0008](0008-canvas-drawn-icons-no-image-assets.md) | Canvas-drawn icons instead of image/icon-font assets | Accepted |

## Template

New records should follow this shape:

```markdown
# NNNN. Title

Date: YYYY-MM-DD

## Status
Proposed | Accepted | Deprecated | Superseded by [NNNN](NNNN-....md)

## Context
What forces (technical, product, constraints) led to this decision?

## Decision
What was decided, stated as a clear sentence.

## Consequences
What becomes easier or harder as a result? Include trade-offs, not just upside.
```
