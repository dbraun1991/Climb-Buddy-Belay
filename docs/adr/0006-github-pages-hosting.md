# 0006. GitHub Pages for hosting and deployment

Date: 2026-08-24

## Status
Accepted

## Context
The tool is a fully static site ([0002](0002-no-backend-static-client-only-app.md))
already living in a public GitHub repository. It needs a URL people can
open on a phone at the gym without cloning the repo.

## Decision
The project is deployed via GitHub Pages, linked from the README as
`https://dbraun1991.github.io/Climb-Buddy-Belay/`. No CI/CD pipeline,
custom domain, or alternative host (Netlify, Vercel, etc.) is used.

## Consequences
- Free, zero-maintenance hosting with deploys tied directly to the repo —
  fits a project with no build step ([0001](0001-single-file-zero-dependency-architecture.md)).
- No custom domain, no server-side redirects/headers, no environment
  separation (staging vs. prod) — every push to the configured branch is
  effectively production.
- Coupled to GitHub as a platform and to the repo's current name/owner for
  the URL; renaming the repo or moving it off GitHub breaks the published
  link and requires updating the README.
