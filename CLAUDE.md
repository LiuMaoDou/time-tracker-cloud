# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project summary

- Single-file web app: `index.html` contains UI, styles, business logic, and cloud sync.
- Stack: Vanilla HTML/CSS/JS, Supabase JS client (CDN ESM), SheetJS (CDN).
- Product language is Chinese.
- Data model is client-side globals on `window` and synced to a single Supabase row.

## Development commands

This repo has no Node toolchain (`package.json` absent), so build/lint/test commands are not configured.

- Run locally (main workflow): open `index.html` in a browser.
  - macOS shortcut: `open index.html`
- Build: N/A
- Lint: N/A
- Tests: N/A (no automated suite; single-test command not applicable)

## Architecture (big picture)

### 1) Runtime split

`index.html` has two script blocks:

- Main app logic (`<script>`): timer, plans, todos, insights, import/export, theming, rendering.
- Cloud sync module (`<script type="module">`): Supabase client init, initial load, save, realtime subscription.

### 2) Core state model

Primary state lives in `window` globals:

- Collections: `records`, `dailyPlans`, `todos`, `questions`
- Timer/session: `currentTask`, `currentTaskDescription`, `startTime`, `pausedTime`, `isPaused`
- Links between modules: `currentPlanId`, `currentTodoId`
- UI-only state: `todoGroupStates`

### 3) Render + persistence flow

Typical mutation flow:

1. Mutate `window` state
2. Call `update*List()` or `refreshAllUI()`
3. Sync to cloud (`triggerSync()` for immediate, `debouncedSave()` for high-frequency input)

Sync guardrails:

- `isDataLoaded` prevents writes before first Supabase load.
- `saveToCloud()` normalizes dates to ISO before upsert.
- Realtime updates call `handleDataLoad()` to refresh local state safely.

### 4) Supabase data model

- Table: `time_tracker_cloud`
- Single-row document pattern: `id = "my_daily_data"`
- Entire app state is stored under row `data` (JSON object)
- Realtime listens to UPDATE on that row only

### 5) Key behavior coupling

- `finishToday()` exports Excel then clears only `records` + `dailyPlans` (keeps `todos` + `questions`).
- Todo completion uses async two-step sync (mark completed, then remove) to avoid race conditions.
- On load/import, completed todos are filtered out to avoid stale restored items.
- Starting a task from todo removes the todo immediately and starts timer.

## Integrations

- SheetJS CDN for Excel export (`XLSX.writeFile` in browser).
- Supabase credentials and Flomo webhook are hardcoded in `index.html`; keep existing behavior unless explicitly asked to change integration config.

## Repository-specific coding conventions

- **Mandatory:** add comments for all new/modified code.
- Comments should be in Chinese (technical keywords can stay in English).
- For multi-step async logic, document step order clearly in comments.
- When sync order matters, use `async/await` and `await triggerSync()`; avoid fire-and-forget writes.
- After mutating arrays (`records/todos/questions/dailyPlans`), always refresh corresponding UI list before sync.
- If adding new theme variables, update both light (`:root`) and dark (`[data-theme="dark"]`) definitions.

## Other instruction sources checked

- `README.md`: not present
- `.cursorrules` / `.cursor/rules/`: not present
- `.github/copilot-instructions.md`: not present
