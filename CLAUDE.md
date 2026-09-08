# Goal Tracker — project memory

Yearly employee goal tracker. **One file, zero dependencies, no build, no server, no Docker** — everything (HTML + CSS + vanilla JS) lives in `index.html`. Keep it that way: no frameworks, no npm, no bundlers.

## Storage (deliberate decisions — don't regress)

- Data is saved **only** to `goals.json` via the File System Access API (Chrome/Edge). There is **no localStorage/browser-storage fallback** — it existed once and was removed on purpose. Unsupported browsers get a "needs Chrome or Edge" screen.
- The user picks a **folder** (not a file) once — `showDirectoryPicker` → `goals.json` is loaded from it or created in it (`FILE_NAME` const). Intended folder: the one containing `index.html`.
- The folder handle persists in IndexedDB (db `goal-tracker`, store `kv`, key `dir`). On load: permission `granted` → silent reconnect; `prompt` → "Welcome back / Reconnect" screen.
- All writes funnel through `queuePersist()` (serialized promise chain) → `doPersist()`. UI never touches storage directly.

## Data schema (`goals.json`)

```json
{ "version": 1, "goals": [ {
    "id": "g_xxxx", "year": 2026, "title": "", "description": "",
    "impact": "", "successMeasures": "", "status": "active|completed",
    "createdAt": "ISO", "completedAt": null,
    "milestones": [{ "id": "m_xxxx", "title": "", "done": false, "createdAt": "ISO" }],
    "updates":    [{ "id": "u_xxxx", "text": "", "createdAt": "ISO" }] } ] }
```

- Array position in `data.goals` is the canonical display order; ▲▼ arrows swap neighbors **within the same year + status group** (`moveGoal`). New goals are `unshift`ed (appear on top).
- `normalize()` sanitizes anything loaded/imported; keep it in sync with schema changes and bump `version` on breaking changes.

## UI architecture

- Full re-render: `render()` = `renderHeader()` + `renderMain()` (rebuilds `#main` innerHTML from state). Events use **delegation** on `#main` (`data-action` attributes) — never bind per-element listeners in cards.
- Re-renders wipe inputs: half-typed milestone/update drafts survive via `captureDrafts()`/`restoreDrafts()`; `focusAfterRender` restores focus (e.g. rapid milestone entry).
- State: `data`, `dirHandle`, `fileHandle`, `pendingDir`, `selectedYear`, `expanded` (Set), `showCompleted`, `editingMilestone`, `saveState`.
- Always escape user text with `esc()` in template literals.

## Testing / verification

- No test framework. Verify in the Claude browser pane: `.claude/launch.json` defines the `goal-tracker` preview (python http.server on port 8642).
- The native folder picker can't be automated. Use the console hook instead — it runs the real code path with an OPFS handle:
  `await goalTracker.adoptDir(await navigator.storage.getDirectory())`
  Clean up afterwards: remove OPFS `goals.json` and `indexedDB.deleteDatabase('goal-tracker')`.
- Synthetic Enter keypresses from automation don't trigger implicit form submission — click the form's button, or dispatch a bubbling `KeyboardEvent` for the milestone-edit commit path.

## Repo

- Remote: `git@github.com:gevorgalaverdyan/goal-tracker.git`, branch `master`.
- `goals.json` is **gitignored** (personal data) — never commit it or weaken that rule. `.claude/` and `.remember/` are ignored too.
