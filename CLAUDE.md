# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Journal S·P·É** is a French-language CBT (cognitive-behavioral therapy) journaling app — Situation, Pensée (thought), Émotion. It is a **single self-contained HTML file** (`journal-spe_1.html`) with no build step, no dependencies, and no server. Open the file directly in a browser to run it.

Licensed under GPL v3.

## Architecture

Everything lives in `journal-spe_1.html`: CSS in `<style>`, HTML structure, and all JavaScript inline in `<script>`. There are no external JS dependencies — only two Google Fonts are loaded remotely.

### State model

A single global object `S` holds all runtime state:
- `S.view` — current tab (`'list'` or `'form'`)
- `S.entries` — array of journal entries, persisted to `localStorage` under the key `'spe'`
- `S.form` — the in-progress entry object `{ eventDate, situation, pensee, emotion, intensite, note }`
- `S.expanded` / `S.confirmDel` — UI state for the list view
- `S.pickerMode` — `'vertical'` (accordion) or `'wheel'` (SVG)

### Emotion data

`EMOTIONS` is a hardcoded array of 8 primary emotions (Plutchik's wheel), each with 3 secondary groups, each with 3 tertiary items — 72 leaf emotions total. Colors are defined per primary emotion and lightened algorithmically for sub-levels.

### Rendering

The app uses **innerHTML string-based rendering** — no virtual DOM, no framework. The two main render functions are:
- `renderList()` — builds the entries list and import/export buttons
- `renderForm()` — builds the new-entry form including the emotion picker

`snapForm()` reads current DOM values back into `S.form` before any state change that triggers a re-render (picker interactions, tab switches).

### Emotion picker

Two modes toggled by `S.pickerMode`:
- **Vertical** (`buildPicker()`): accordion-style nested buttons
- **Wheel** (`buildWheel()`): procedurally generated SVG using arc path math. The wheel is rebuilt as a string on every selection change.

`findColor(label)` resolves any emotion label (primary, secondary, or tertiary) to its primary hex color — used throughout for consistent theming.

### Storage & data portability

- `save()` / `load()` — read/write `S.entries` as JSON in `localStorage`
- Export: uses Web Share API (mobile) with fallback to `<a download>` (desktop)
- Import: merges by `id`, deduplicating against existing entries, then sorts by recorded date

## Development

No build tooling. Edit `journal-spe_1.html` directly and open in a browser. Use browser DevTools for debugging.

To validate HTML/JS syntax locally, any standard linter works (e.g. `npx htmlhint journal-spe_1.html` or browser console errors on load).

## Key conventions

- All user-facing strings are in **French**.
- HTML output is sanitized through `ehtml(s)` which escapes `& < > "` — always use it when interpolating user data or emotion names into innerHTML strings.
- Entry IDs are `Date.now().toString()` — millisecond timestamps as strings.
- `nowLocal()` produces a `datetime-local`-compatible string adjusted for the user's timezone offset.
