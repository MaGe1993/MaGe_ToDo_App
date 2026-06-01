# MaGe ToDo — User Guide

[English](README.md) | [Deutsch](README.de.md)

MaGe ToDo is a lightweight Windows desktop app for organising daily work across several projects as kanban boards. It tracks due dates with relative labels, derives a per-project health score, and shows everything on a dashboard with KPIs, trend sparklines and charts. It runs entirely offline — data lives next to the app, there is no account and no backend. A built-in MCP server lets AI agents (e.g. Claude Code) read and drive the app locally. Built with Tauri, Rust and SolidJS; installs via a small NSIS setup.

A personal, multi-project to-do app for daily planning. It installs from a setup wizard (English or German), stores all its data locally on your machine, and needs no account or internet connection. The app opens on the Dashboard and comes pre-loaded with a set of example projects so you can explore it right away. This guide explains how to work with the app: what each button does and what every screen offers.

## 1§ The window

The title bar across the top carries, from left to right: the app logo and name, and a row of buttons.

- Settings (gear) — opens the Settings page.
- Documentation (question mark) — opens a menu: About, Readme, Documentation, MCP Server, License. Hover or click to open it.
- Support me (heart) — opens the developer's Ko-fi page in your browser.
- Minimize, Maximize/Restore, Close — standard window controls. Double-clicking an empty part of the title bar also toggles maximize.

## 2§ Projects and the sidebar

The left sidebar lists your projects. The order is meaningful: a project's position is its priority, top = 1. Reorder by dragging a project up or down — an insertion line shows where it will land.

- Dashboard — the top entry opens the overview across all projects (see 5§).
- Filter projects — the search field filters the sidebar by name as you type. On a collapsed sidebar, click the magnifier to expand and focus it.
- Open a project — click it to show its board.
- New project (plus) — adds a new project and opens it.
- Collapse the sidebar — the chevron button at the top narrows the sidebar to icons only.

## 3§ The board

A board shows one project as a set of lists (kanban columns). The last list is always `Done` and cannot be removed.

### 3.1§ Board header

The header shows the project emoji and name, its priority number, a health LED, a completion bar, and two-line counts for tasks due today and overdue. On the right are the board controls:

- Group by prefix — groups tasks in each list by their first word when it repeats.
- Group by due date — groups tasks by due date instead.
- Expand / collapse all — opens or closes every group at once.
- Search — filters tasks across the board by text.
- Add list — adds a new list (inserted just before `Done`).
- Edit project (pencil) — turns on inline edit mode (see 3.4§).
- Delete project (trash) — deletes the project after a confirmation. Disabled when only one project remains.

### 3.2§ Tasks

Each task carries a text, an optional due date, and a completion date once finished.

- Add a task — type into the `Add item…` field at the bottom of a list and press Enter. Optionally pick a due date first in the date field next to it.
- Due date field — type the date directly (it formats itself as you type, e.g. `12032026` becomes `12.03.2026`), or click the calendar icon to pick it. The format follows your Settings choice.
- Complete a task — click its check-circle, or drag it into the `Done` list. This stamps today as the completion date.
- Reopen a task — drag it out of `Done`; the completion date is cleared.
- Edit a task — click it to edit its text and due date inline. Clearing the text deletes the task.
- Move a task — drag it to another list; an insertion line shows the drop position.

### 3.3§ Due labels

A task's due date is shown relative when it is close: `Today`, `in N days`, or `+N days` when overdue. Dates further out show as an absolute date in your chosen format. Today is highlighted in the warning color, overdue in the error color.

### 3.4§ Edit mode

The pencil button turns the project title, emoji, and every list title and emoji into editable fields. Each list gets a delete button (except `Done`). Click the pencil again to finish.

## 4§ Reordering by drag-and-drop

Three things reorder by dragging, each with an insertion-line indicator:

- Projects in the sidebar (changes priority).
- Lists within a project (the `Done` list always stays last).
- Tasks between lists.

## 5§ The dashboard

The dashboard summarizes all projects. Nothing here is stored separately — every figure is recomputed from your task dates.

### 5.1§ Top row

- Overall health — a gauge showing the average project status (all healthy reads green, all critical reads red).
- No due date — a ring showing how the open tasks without a due date split across projects; the center shows the total.
- Clocks — a large local clock pinned on the right, plus any extra world clocks you add in Settings.

### 5.2§ KPI strip

Six metrics, each with a day-over-day change arrow and a 7-day sparkline: Completion, Total Tasks, Overdue, Due Today, Total Delay, Average Delay. The arrow and sparkline are colored by whether the change is good or bad, not merely by its direction.

### 5.3§ Charts

One panel with a chart selector and a shared date range (1M / 3M / 1Y / MAX):

- Tasks — a stacked bar of upcoming open tasks by due day, one color per project.
- Progress — each project's completion percentage over time.
- Priority — a ranking bump chart of project priority over time. A project's line starts on the day it first appeared; today's rank follows the current sidebar order.

### 5.4§ Project cards

A card per project with its health LED, completion bar, and a per-list breakdown of open-task counts (badges turn warning or error when a list has tasks due today or overdue). Click a card to open that project.

## 6§ Settings

Reach Settings from the gear in the title bar. Sections:

- Appearance — theme (Dark / Light / System), accent color (swatch plus HEX/RGB/HSL fields), and an animation toggle.
- Language & region — interface language (English / German), time zone, date format, number format, and the Dashboard clocks list (add named world clocks; each row has a delete button, a label, and a zone).
- Notifications — a master switch for desktop notifications, a sounds toggle (plays a soft chime), do-not-disturb hours, and per-event toggles: Tasks due today, Tasks overdue, Daily summary, Task completed. Each timed event has a time-of-day picker for when it fires (defaults: due today 09:00, overdue 10:00, daily summary 14:00). Completing a task notifies immediately; the timed ones fire once per day at their set time. Every notification honours the master switch, its own toggle, and do-not-disturb.
- Projects — configures project health. Completed tasks do not count toward health. The driving figure is Risk = the worse of two percentages: overdue open tasks vs. all open tasks, and total delay of open tasks vs. the project's planned span. Two steppers set the thresholds (defaults 5 and 20): Risk at or below the first is success, above the second is error, in between is warning. The card shows the three states with their LEDs.

## 7§ Documentation pages

The question-mark menu in the title bar opens:

- About — app name, version, and credits.
- Readme — this guide.
- Documentation — the technical project specification.
- MCP Server — how an external AI agent connects to this app.
- License — the license text.

## 8§ AI agent control

Every running instance also exposes a local control server so an external AI agent (such as Claude Code) can read your projects and drive the app — add and complete tasks, reorder projects, change settings, and more. It listens only on your own machine and requires a token. See the MCP Server page in the app for details.
