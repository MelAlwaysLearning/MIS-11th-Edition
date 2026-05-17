# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the app

No build step or server required. Open `index.html` directly in any modern browser:

```
open index.html          # macOS
start index.html         # Windows
xdg-open index.html      # Linux
```

## Architecture

**Single-file app.** All HTML, CSS, and JavaScript live in `index.html`. There are no external dependencies, no package manager, and no framework — plain vanilla JS with no modules.

### Data layer (JavaScript objects)

Three static data objects defined near the top of the `<script>` block:

- **`GAMES`** — keyed by chapter ID (e.g. `"1"`, `"TG1"`). Each entry has `title`, `emoji`, `tips[]` (study hints rendered before the quiz), and `qs[]` (question objects: `{q, o[], a}` where `a` is the zero-based correct answer index).
- **`NOTES`** — same chapter IDs as keys. Each entry has `overview` (plain string), `concepts[]` (HTML strings), and `terms[]` (`{t, d}` pairs).
- **`CHAPTER_BOOKMARKS`** — seed data array of 17 bookmark objects (Ch 1–14, TG1–TG3) used to initialize `localStorage` on first load.

### Special URL scheme

Bookmarks whose `url` starts with `"game:"` are game cards, not links. The suffix is the chapter ID (e.g. `"game:1"`, `"game:TG2"`). `isGameUrl()` and `gameId()` detect and extract these. Game cards render "Play Learning Game" and "Chapter Notes" buttons instead of a hyperlink.

### State

All runtime state lives in module-level variables in the script:

| Variable | Purpose |
|---|---|
| `bookmarks[]` | Loaded from / persisted to `localStorage` |
| `activeCategory` | Currently selected filter tab |
| `searchQuery` | Live search string |
| `editingId` | `null` when adding, bookmark ID when editing |
| `gameState` | Current quiz progress `{id, qs, cur, score, answered}` |
| `notesCurrentId` | Which chapter is open in the notes modal |
| `sgActiveCategory` | Filter state for the Study Guide panel |

### Persistence

- Bookmarks: `localStorage` key `tjourney_bookmarks_v1` (JSON array)
- Dark mode: `localStorage` key `tjourney_dark` (`"1"` / `"0"`)

On first load, if `tjourney_bookmarks_v1` is absent or empty, `CHAPTER_BOOKMARKS` is written as the seed data.

### UI panels / modals

| Panel | Trigger | Key IDs |
|---|---|---|
| Add/Edit modal | "Add Bookmark" button or card edit icon | `modal-overlay`, `bookmark-form` |
| Game modal | "Play Learning Game" on a game card | `game-overlay` (phases: `game-intro` → `game-quiz` → `game-result`) |
| Chapter Notes modal | "Chapter Notes" on a game card | `notes-overlay` |
| Study Guide panel | "Study Guide" header button | `sg-overlay` (accordion per chapter) |

All modals close on Escape key and on backdrop click.

### Rendering flow

`init()` (called on `DOMContentLoaded`) → `loadFromStorage()` → `renderFilterButtons()` + `renderGrid()`. Any CRUD operation ends with `saveToStorage()`, `populateCategoryDatalist()`, `renderFilterButtons()`, `renderGrid()`. The grid is always re-rendered from scratch; there is no virtual DOM or diffing.
