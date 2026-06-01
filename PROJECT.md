---
name: mage-todo-project
description: Product spec and architecture decisions for MaGe ToDo.
updated: 2026-05-31
---

# MaGe ToDo — Project Spec

## 1§ Goal

A simple, personal to-do app for daily planning and coordination. Single-user, portable desktop tool — no cloud backend, no accounts, data lives next to the exe.

Target users: the developer themselves, for personal task tracking and daily coordination.

## 2§ Core features

- Multi-project board: each project is a set of lists (kanban columns); the last list is always the mandatory `Done` list.
- Tasks carry text, an optional due date (ISO), and a completion date. Completing a task (CheckCircle or drag into `Done`) stamps the completion date; dragging out clears it.
- List grouping inside a column: by prefix (first word, when it occurs ≥ 2×), by due date, or none (alphabetical). Group headers carry a count badge when collapsed; expand/collapse-all toggle.
- Relative due labels (`Today`, `in N days`, `+N days` overdue) within 7 days, absolute date (settings format) otherwise; today = warning colour, overdue = error colour.
- Drag-and-drop: reorder projects (navbar), reorder lists within a project, and move tasks between lists — all with an insertion-line indicator.
- Inline edit mode on the board (no separate edit page): the edit button turns the project title/emoji and every list title/emoji into emoji-pickers + inputs, with per-list delete; add-list and delete-project live in the board header. The header is a wrapping row of intact blocks (between dividers) so it never clips on narrow windows.
- Board header shows the project's priority — its position in the navbar order (top = 1) — plus a health LED (blinks at error level), completion bar, and two-line today / overdue counts.
- Dashboard: overall health semi-gauge and a no-due-task doughnut (per-project segments) on the left, world clocks on the right (the local clock is largest and pinned far right); a flat divider-separated KPI strip (Completion, Total/Overdue/Due-today, Total/Avg delay — each with a day-over-day delta and a 7-day sparkline coloured success/error by trend); one chart panel with a chart selector (Tasks / Progress / Priority) and a shared date range (1M/3M/1Y/MAX); and per-project cards with a per-list open-count badge breakdown.
- Settings: theme/accent (default accent `#00ff00`), animation, language (English/German, fully localized UI), date format, number format, do-not-disturb, per-event notification toggles with per-event send times, a sounds toggle (chime), configurable project-health thresholds, and named alternate time zones (shown as extra dashboard clocks; none by default).
- Notifications engine: completing a task notifies immediately; a scheduler fires due-today / overdue / daily-summary once per day at their configured times; every notification honours the master switch, per-event toggle and do-not-disturb, and plays a chime when sounds are on. Agent toasts share the same chime.
- Opens on the Dashboard. First run seeds a set of example projects from a bundled dataset (`seed-todo.json`) so the app is non-empty out of the box.
- Persistent storage of the full dataset in `data/todo.json` (atomic write); UI preferences (theme, accent, language, date format, world clocks, notification settings) in `localStorage`.
- Native desktop integration: single-instance enforcement, window state restore, custom DWM-tinted titlebar (the build stamp shows only in dev builds).
- MCP agent server for external AI-agent control (`agent.rs`, discovery via `app_data_dir/agent.json`): full read + write surface over the data model (projects, lists, items, completion, stats) plus navigation, settings and notifications. See §4.11.

## 3§ Non-goals

- No multi-user or roaming support
- No cloud sync or remote backend
- No mobile (Windows only)
- No complex project / sub-task hierarchy in the initial version
- No in-app auto-update — distributed as a standalone NSIS installer; a new version is installed by downloading and running the new `setup.exe` (perMachine upgrade in place)

## 4§ Architecture decisions

### 4.1§ Installable build (per-machine, installed-local)

`data_policy: installed-local`. The app ships as an NSIS `setup.exe` (`bundle.targets: ["nsis"]`, `installMode: perMachine`) that installs into Program Files. The data folder lives under the install dir (`<install>\data`); `base_dir()` resolves it from `current_exe().parent()/data`, which equals the install dir at runtime. The NSIS POSTINSTALL hook (`installer-hooks.nsh`) creates `<install>\data` and grants the BUILTIN\Users group (well-known SID `S-1-5-32-545`) modify rights via the built-in `icacls`, so standard users can read/write data without elevation. AccessControl (the NSIS plugin the studio template assumes) is NOT bundled with Tauri's NSIS here, hence icacls. The POSTUNINSTALL hook removes `<install>\data` and the install dir. The installer is personalized: custom header/sidebar BMPs (generated from the logo, in `icons/`), installer icon, an English/German language selector (`languages`, `displayLanguageSelector`), a license page (`bundle.licenseFile` → `Licence.txt`), and a custom NSIS template (`bundle.windows.nsis.template` → `installer.nsi`, derived from Tauri's default) that adds a finish-page thank-you text and a Ko-fi support link (localized `LangString`s). The finish page also keeps Tauri's default optional "create desktop shortcut" and "run app" checkboxes. `compression: lzma`; `webviewInstallMode: embedBootstrapper` embeds the small WebView2 bootstrapper (keeps the setup a few MB; it fetches the runtime at install time only if WebView2 is missing).

### 4.2§ Notifications

The NSIS installer sets the app's AUMID, so native Windows toasts via `tauri-plugin-notification` are attributed correctly — no portable-build AUMID caveat applies.

A frontend notification engine (`fireNotification` in `shell/src/App.tsx`) drives delivery. It reads the Settings (persisted in `localStorage`) and gates every notification on the master switch, the per-event toggle and the Do-Not-Disturb window (`inDnd`, overnight-aware). It shows a SILENT OS toast (`sendNotification({ silent: true })`) and plays the in-app `playChime()` Web-Audio sound as the single, consistent sound source governed by the Sounds toggle. Triggers: completing a task (`completeItem` / drag into Done → `notifDone`), and a 30 s scheduler that fires `notifDueToday` (09:00), `notifOverdue` (10:00) and `notifSummary` (14:00) once per day at/after their configured time (catch-up on launch; due-today/overdue stay quiet when their count is 0). Times default in `NOTIF_EVENTS` and persist under `notifTimes`; per-day de-dup via `notifFired_<key>`. Agent/MCP `send_notification` emits a `notify:chime` event so agent-driven toasts also sound via the same chime (honouring Sounds + DND).

### 4.3§ Self-contained UI assets

`shell/public/` holds the full MaGe_UI kit (theme, components, behaviors, charts, icons, fonts, vendor). No sync script — what is in `public/` is what ships. Pull kit updates from `MaGe_UI/core/` manually when needed.

### 4.4§ No APP_MANIFEST.json

App identity lives in `tauri.conf.json` (`productName`, `version`) and `Cargo.toml` (`[workspace.package] version`). These two must match before every release.

### 4.5§ Single workspace

Single Cargo workspace (`crates/core` + `shell/src-tauri`). Split into engine + shell workspaces only if compute crates dominate build time.

### 4.6§ Tauri-specta type bridge

Every command annotated `#[specta::specta]` and registered via `collect_commands!`. Bindings emitted to `shell/src/bindings.ts` on every debug build. Frontend always calls `commands.<name>(args)`, never raw `invoke()`.

### 4.7§ Plugin baseline

- `tauri-plugin-single-instance` — second launch focuses the first window
- `tauri-plugin-log`, `tauri-plugin-store`, `tauri-plugin-window-state`
- `tauri-plugin-deep-link`, `tauri-plugin-notification`, `tauri-plugin-opener`
- `tauri-plugin-dialog`, `tauri-plugin-process`, `tauri-plugin-os`
- `tauri-plugin-clipboard-manager`, `tauri-plugin-global-shortcut`
- `tauri-plugin-autostart`

Opt out per feature by removing the crate from `Cargo.toml`, the plugin call from `lib.rs`, and the permission from `capabilities/default.json`.

### 4.8§ Data model and persistence

The full dataset (`TodoData`) is defined in `crates/core` with `specta`/`serde`: `{ active, projects[], ui }`, each project `{ name, emoji, lists[] }`, each list `{ name, icon, items[] }`, each item `{ text, due, done }` (both dates optional ISO `YYYY-MM-DD`). `TodoItem` has a backward-compatible manual `Deserialize` (a bare string loads as `{ text, due: null, done: null }`). `normalize()` guarantees every project ends with a `Done` list. Loaded/saved as a whole via the `load_todos` / `save_todos` commands (atomic write to `data/todo.json`); the frontend persists the entire object on every change. On first run (no `data/todo.json`), `load_todos` seeds from the bundled `seed-todo.json` resource (a set of example projects, accent `#00ff00`) and writes it to the data dir; if the seed is absent it falls back to the compiled `TodoData::default()`. UI-only preferences (theme, accent, language, `dateFormat`, `tzClocks`, notification settings) live in `localStorage`, not in `TodoData`. The frontend opens on the Dashboard page by default.

### 4.9§ Dashboard analytics are derived, not stored

There is no history table. All dashboard time-series (open/overdue counts, KPI day-over-day deltas + sparklines, project priority ranking) are reconstructed at render time from the `due` and `done` dates over a fixed window (30 days for KPIs/trends, 1M/3M/1Y for the charts). Helpers live near the dashboard components in `shell/src/App.tsx` (`dailyMetrics`, `completionAt`, `priorityHistory`, `dateSeries`). Charts use `window.MaGePlot.chart(...)` with full ECharts options (not the wrapper builders) so the dashboard controls the option directly; `cssVar()` resolves design tokens for ECharts colours.

The priority chart is a bump chart: `priorityHistory()` returns a `(number|null)[][]` keyed `[projectIndex][dayIndex]`. Each day ranks only the projects that already existed (earliest `done`/`due` per project via `projectStart()`); not-yet-created projects are `null` so no line is drawn. The last day's rank is the live navbar order (`pi + 1`); earlier days are a deterministic, seed-stable pseudo-random history (FNV-1a hash on the project name, segmented) so reordering the navbar only moves today's point. Tasks chart rounds each day's topmost stacked segment via per-data-point `itemStyle`. The no-due ring uses a background-coloured border for inter-segment gaps and `appendToBody` tooltips so they are not clipped by the small chart box.

### 4.9.1§ Project health model

`projStats()` (frontend) computes status from two ratios, ignoring completed tasks: `overdueRatio = overdue / open` (overdue OPEN tasks over all OPEN tasks) and `delayRatio = activeDelaySum / plannedSpan`, where `activeDelaySum` is the summed days-overdue of OPEN tasks and `plannedSpan` is the earliest→latest due over ALL tasks (completed ones included only to locate the project start), clamped to 1. `risk = max(overdueRatio, delayRatio) * 100` is the single driving figure. The `totalDelay`/`avgDelay` KPIs remain completion-delay metrics and are unrelated to health. Two thresholds, `healthT1` (success↔warning, default 5) and `healthT2` (warning↔error, default 20), are module-level signals persisted in `localStorage` and editable in Settings → Projects. `healthLevel`: success when `risk ≤ t1`, error when `risk > t2`, warning otherwise. The 0–100 `health` score maps risk onto the thresholds (100→80 across success, 80→40 across warning, 40→0 beyond t2) so the gauge needle reacts to threshold changes. The dashboard "Overall health" gauge does NOT use a merged-tasks score — it is the AVERAGE of per-project status points (success→0, warning→50, error→100), matching the gauge's green→red zones. `agent.rs` mirrors this model with fixed defaults (`HEALTH_T1 = 5`, `HEALTH_T2 = 20`).

### 4.9.2§ Analog clock sizing

An ECharts gauge draws its dial on only ~64% of its canvas, while the no-due pie ring fills ~85%. To make a clock's visible face equal the ring's, `AnalogClock` uses a larger canvas (200 for a 150 ring) and crops the surplus whitespace with negative margins on all sides (`-(size-150)/2` per side). ECharts also freezes the canvas to the container size at init time and the clock tick loop only calls `setOption` (never `resize`); so `AnalogClock` captures the instance and calls `inst.resize({ width, height })` with explicit pixel dims after init and on every `ResizeObserver` callback. `.dash-clock > .plot { flex-shrink: 0 }` keeps the dial from being squashed by its two-line foot.

### 4.9.3§ Date field

`DateField` is a masked text input plus a hidden native `<input type=date>` driver. Typing digits auto-formats live into the active Settings date format (e.g. `12032026` → `12.03.2026`); the separator and token order are derived from `dateFormat()`. A value is emitted only once the date is complete and valid. The calendar icon (muted at rest, full on hover via `--icon-2-a`) calls `showPicker()` on the hidden input.

### 4.10§ Kit additions (synced to MaGe_UI)

Generic additions made in `MaGe_UI/` and mirrored into `shell/public/`: a `.led` status-light component (concave well + semantic fill, matching the radio LED spec), a `clock(el, offsetH)` time-zone argument on `MaGePlot` (backward compatible — no offset = local time), and two icon-registry entries (`circle`, `square` outline icons). App-specific composition (board, dashboard) stays in `shell/src/app.css`; only genuinely reusable primitives go back to the kit.

### 4.11§ MCP agent control surface

The MCP server (`shell/src-tauri/src/agent.rs`) is the app's complete AI control plane, not a token sample. Tools, grouped:

- Reads: `get_app_info`, `get_today`, `get_state`, `get_ui`, `list_projects`, `get_project`, `get_stats`, `list_items`, `search_items`, `list_overdue`, `list_due_soon`.
- Project mutations: `add_project`, `update_project`, `delete_project`, `set_active_project`, `reorder_project` (priority), `duplicate_project`.
- List mutations: `add_list`, `update_list`, `delete_list`, `reorder_list`.
- Item mutations: `add_item`, `add_items` (bulk), `update_item`, `set_item_due`, `delete_item`, `reorder_item`, `complete_item`, `uncomplete_item`, `move_item`, `clear_done`.
- UI / app: `navigate_to`, `set_theme`, `set_accent`, `get_setting`, `set_setting`, `send_notification`.
- Whole-dataset: `set_state` (replace `{ active, projects, ui }`, normalized).

Projects and lists are addressed by numeric index or by name. Read tools load `data/todo.json` directly; write tools mutate it, normalize (Done list guaranteed), persist atomically, and emit `agent:reload` so the open window re-reads its state at once. The server computes derived stats (open, done, overdue, due-today, health, healthLevel, riskPercent, priority score) using the same risk/threshold model as the frontend (defaults 5/20) via a dependency-free UTC civil-date helper. Auth and transport: bearer token, `127.0.0.1` only, endpoint `POST /mcp`.

## 5§ Constraints and external dependencies

- Windows-only — uses `dwmapi` for native titlebar tinting
- DevTools off in release (the `devtools` Cargo feature is not enabled); still available in dev builds
- Content-Security-Policy set in `tauri.conf.json` (`app.security.csp`): local assets only, no outbound network except Tauri IPC
- No in-app updater — see 3§
- Codesigning not configured — see studio CLAUDE.md 9.6§ when shipping publicly
