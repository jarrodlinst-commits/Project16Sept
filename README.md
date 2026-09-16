# UOB IT PMO Kanban Board

A single-file demo/training Kanban board for UOB's internal IT PMO, built as a self-contained `index.html` — no backend, no build step, no dependencies.

**Live demo:** https://jarrodlinst-commits.github.io/Project16Sept/

## Running it

Just open `index.html` in a browser (double-click it, or run `open index.html` on macOS). There's no build/lint/test command — none exist in this project, and none are needed.

## What it does

- Four-column Kanban board (drag-and-drop, with a keyboard-accessible "Move ▸" fallback on each card)
- Add, move, and delete tasks, with inline delete confirmation
- Client-side filtering by project, assignee, and priority
- A summary header with live task counts
- Optional email notification on new-task creation via [FormSubmit](https://formsubmit.co) — fire-and-forget only, never a source of truth, and never blocks or breaks the board if it fails

## Key constraints

This project is intentionally minimal and self-contained:

- **Vanilla HTML/CSS/JS only** — no frameworks, no build tooling, no npm/bundler. Everything lives in `index.html` and runs directly via `file://`.
- **No external resources** — no CDN scripts/styles, no web fonts, no images. Icons are Unicode glyphs; fonts are the system font stack.
- **No persistence** — board state lives only in an in-memory JS object. There's no `localStorage`, `sessionStorage`, `IndexedDB`, or cookies, so **refreshing the page resets the board to its seed data** by design. The header includes a note explaining this to users.
- **FormSubmit is the only network call**, used purely for notification and never as a source of truth.

See [`CLAUDE.md`](./CLAUDE.md) for full architecture notes and the exact hard constraints this project preserves.

## Deployment

This repo deploys automatically to GitHub Pages via GitHub Actions on every push to `main` (see [`.github/workflows/pages.yml`](./.github/workflows/pages.yml)).
