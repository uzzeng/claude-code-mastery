# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build step or package manager. Open directly in a browser:

```bash
# Option 1: Python static server
python -m http.server 8000
# Then visit http://localhost:8000

# Option 2: Open index.html directly in browser
open index.html
```

## Architecture

Two-layer separation with no framework:

**`js/storage.js`** — `BucketStorage` singleton object (data layer)
- All reads/writes go through `load()` / `save()` which wrap `localStorage`
- Every mutation method calls `load()` → mutate → `save()` (no in-memory state)
- `getFilteredList(filter)` accepts `'all'` | `'active'` | `'completed'`

**`js/app.js`** — `BucketListApp` class (UI layer)
- Holds only two pieces of state: `currentFilter` and `editingId`
- `render()` is a full re-render on every user action — no partial DOM updates
- Item HTML is built via `createBucketItemHTML()`, which calls `escapeHtml()` before inserting user input into the DOM (XSS guard)
- Event handlers for items are wired via inline `onclick="app.handleToggle(...)"` referencing the global `app` instance

## Data Model

```js
{
  id: Date.now().toString(),   // timestamp-based unique ID
  title: string,
  completed: boolean,
  createdAt: ISO string,
  completedAt: ISO string | null
}
```

New items are prepended (`unshift`) so the list is newest-first.

## Key Constraints

- No tests, no linter, no CI — validate changes by opening the app in a browser
- Tailwind CSS is loaded from CDN (`cdn.tailwindcss.com`); no local Tailwind config exists
- `css/styles.css` only supplements Tailwind — filter button active state, `slideIn`/`fadeIn`/`scaleIn` keyframe animations, mobile breakpoint (`max-width: 640px`), and a basic dark mode via `prefers-color-scheme`
