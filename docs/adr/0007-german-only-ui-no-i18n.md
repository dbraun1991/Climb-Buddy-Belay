# 0007. German-only UI, no i18n abstraction

Date: 2026-08-24

## Status
Superseded by [0009](0009-inline-vanilla-i18n-en-de.md)

## Context
All user-facing text — labels, error messages, legend, summary sentences,
gear hints — is written directly in German and hardcoded into
`index.html` (e.g. `lang="de"` at `index.html:2`, strings like `"Bitte
einen Namen eingeben."` at `index.html:566`). There is no translation
layer, locale file, or language switcher.

## Decision
The UI targets German-speaking users only. Text lives inline in the
markup and JS template strings, with no i18n framework or externalized
string table.

## Consequences
- No i18n infrastructure to build or maintain for a tool with one intended
  audience, consistent with [0001](0001-single-file-zero-dependency-architecture.md)'s
  zero-dependency stance.
- Adding a second language later means either duplicating the whole file
  per language or retrofitting a string-table/switcher — a non-trivial
  restructuring, not a small patch, since text is currently interleaved
  throughout markup, CSS-adjacent labels, and JS logic rather than
  centralized.
- Limits the tool's reach to German-speaking climbers unless this decision
  is revisited.

## Update (2026-08-24)
Superseded by [0009](0009-inline-vanilla-i18n-en-de.md): an EN/DE switcher
was added using an inline translation table, not the string-table
duplication or restructuring predicted above. This record stays as the
historical rationale for the original German-only period.
