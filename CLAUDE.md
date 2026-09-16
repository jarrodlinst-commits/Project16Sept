# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file demo/training web app: a Kanban board for UOB's internal IT PMO. Everything — markup, styles, and logic — lives in `index.html`. There is no other source, no README, no tests, no package.json, and no git repo initialized yet.

## Running it

No build step, no server, no dependencies. Open `index.html` directly in a browser (double-click, or `open index.html` on macOS). There is no lint/test/build command — none exist in this project.

## Hard constraints (do not violate when editing)

These were explicit requirements for this project and should be preserved in any future changes:

- **Vanilla HTML/CSS/JS only.** No frameworks, no build tooling, no npm/bundler. Keep it a single file that runs via `file://`.
- **No external resources.** No CDN scripts/styles, no Google Fonts, no image files. Icons are Unicode glyphs; fonts are the system stack (`--font-stack` in the `<style>` block).
- **No persistence.** Board state is an in-memory JS object only — no `localStorage`, `sessionStorage`, `IndexedDB`, or cookies. A page refresh is *intended* to reset the board to seed data; the header's "memory note" documents this to the user.
- **FormSubmit is the only network call**, and it's fire-and-forget notification only — never a source of truth. It must never block or break the board on failure.

## Architecture

Single `state = { tasks: [], filters: {...} }` object is the source of truth (see the script's STATE section). All rendering is a pure function of `state`: `renderBoard()` rebuilds the four Kanban columns from `state.tasks` filtered by `state.filters`/status, and `renderSummary()` recomputes the header counts. There is no direct DOM mutation of card contents outside `renderBoard()`/`renderCard()` — mutate `state`, then call `renderBoard()`.

Key functions (all in the one `<script>` block, in a single IIFE):
- `renderBoard()` / `renderCard(task)` — full re-render from state; cards are built as HTML strings via `escapeHtml()` (all user-supplied strings must go through this before insertion — no raw `innerHTML` of unsanitized input).
- `moveTask(taskId, newStatus)` — the single mutation path for changing a task's column, used by both the native HTML5 Drag-and-Drop handlers (`attachDnDHandlers()`) and the keyboard-accessible "Move ▸" `<select>` fallback on each card (event-delegated in `attachBoardDelegation()`).
- `deleteTask(taskId)` — gated by an inline per-card "Delete? Yes/No" toggle (`deleteConfirmId` in state), not `confirm()`.
- `addTask(values)` — optimistic UI: pushes to `state.tasks` and re-renders immediately, then calls `notifyNewTask()` in parallel; a FormSubmit failure only shows a warning toast, it never removes the card.
- `notifyNewTask(task)` — the sole network call, POSTs to `FORMSUBMIT_ENDPOINT` (config constant at the top of the script, clearly marked). Wrapped in try/catch by its caller.
- `applyFilters()` — pure client-side filter over `state.tasks` (project exact-match, assignee substring, priority exact-match); re-renders via `renderBoard()`.
- `showToast(message, type)` — transient toast into the `aria-live="polite"` region.

## FormSubmit notes

`FORMSUBMIT_ENDPOINT` currently holds a placeholder email (`YOUR_EMAIL@example.com`) — swap it there when wiring this up for real. FormSubmit requires a one-time activation per address: the first AJAX submission sends a confirmation email, and no notification actually delivers until that link is clicked. This is expected and non-blocking — the board must keep working regardless of whether FormSubmit is activated.

## Task ID scheme

IDs are generated as `UOB-ITPM-####` via `nextTaskId()`, which increments the in-memory `idCounter`. Seed data consumes the first 8 IDs on load, so newly added tasks continue from `UOB-ITPM-0009`.
