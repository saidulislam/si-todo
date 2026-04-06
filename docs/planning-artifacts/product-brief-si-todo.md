---
title: "Product Brief: SI-Todo"
status: "complete"
created: "2026-04-05"
updated: "2026-04-05"
inputs: [user-interview, competitive-landscape-research]
---

# Product Brief: SI-Todo

## Executive Summary

Engineering leaders juggle dozens of workstreams across multiple teams — and the hardest thing to track isn't the team's work, it's *your own*. What did you delegate, to whom, and when is it due? What's slipping? What fell off the radar entirely? The tools that exist are either blocked by corporate IT (Trello), bloated with team features you don't need (Jira, Asana), or too primitive to be useful (sticky notes, text files).

**SI-Todo** is a personal Kanban board web app — radically single-player by design — built for one engineering leader who needs a clean, visual command center for their own tasks, delegations, and deadlines. Two signature features set it apart: a first-class **Delegated** column with assignee tracking (because leaders delegate, and no personal tool handles that well), and a **due-date aging system** where cards automatically shift from green to yellow to red as deadlines approach — turning the entire board into an urgency heat map that tells the truth even when you don't ask.

The app runs locally on your laptop — `localhost`, no deployment, no infrastructure. A lightweight backend persists data to disk so nothing is lost across restarts. No login required. No Jira admin. No IT ticket. No procurement cycle. It's the tool that works precisely because it lives outside the corporate stack — the private operating system for an engineering leader who refuses to drop balls.

## The Problem

You lead multiple engineering teams. Every day, you're assigning work, following up on delegations, context-switching across workstreams, and trying to remember what's due when. The mental overhead is real:

- **Delegations vanish into the ether.** You asked Ahmed to look into the deployment issue on Tuesday. It's now Friday. Did he finish? Did you forget to follow up?
- **Priorities blur.** You have 30 open items. Which ones are P1 fires and which are P3 nice-to-haves? Without visual signals, everything feels equally urgent — or equally forgettable.
- **Deadlines sneak up.** That architecture review was due yesterday. You didn't realize it until someone asked for it.
- **Corporate tools are unavailable or overkill.** Trello is blocked. Jira is for the team, not for you. Notion requires yet another account and a learning curve.
- **The real competitor is the neglected text file.** You end up with a mix of sticky notes, scratch pads, pinned Slack messages to yourself, and a TODO.md that hasn't been updated in weeks.

The cost of the status quo is dropped commitments, delayed follow-ups, and the nagging feeling that something important is slipping through the cracks.

## The Solution

SI-Todo gives you a single-screen Kanban board with six purpose-built columns:

**Backlog → To Do → In Progress → Delegated → Blocked → Done**

Each card captures what matters: title, description, who you delegated to, due date, priority level (P1/P2/P3), and customizable tags. You drag cards between columns as work progresses.

**What makes the experience work:**

- **Due-date aging** — The signature behavioral feature. Cards automatically shift from green (7+ days) → yellow (3–7 days) → red (<3 days) → pulsing red (overdue) as deadlines approach. Your board becomes a heat map that creates urgency without notifications and surfaces risk without nagging. These thresholds should be configurable.
- **User-customizable card colors** — Define your own color rules to denote priority, work type, assignee, or any categorization that fits your mental model. Custom colors apply as card accents/borders, while due-date aging controls the card background — so both signals coexist.
- **Filtering and search** — Slice the board by assignee, priority, due date, or tag. "Show me everything I delegated to Ahmed" is one click.
- **At-a-glance dashboard** — Total items, overdue count, breakdown by assignee and priority. The numbers you need without digging.
- **Light/dark mode toggle** — Clean white or dotted grid background in light mode, with a dark mode option for late nights.
- **Persistent storage** — Data is saved to a backend store on every change. Restart your server, reboot your VDI, close the browser — your board is exactly where you left it.
- **Localhost deployment** — Runs entirely on your laptop. No servers, no infrastructure, no network exposure. Start the app, open the browser, and you're working.

## What Makes This Different

Existing solutions fail this use case in specific ways:

| Tool | Why It Fails |
|------|-------------|
| Trello | Corporate-blocked; bloated with team features and upsell nudges |
| Notion | General-purpose workspace, not a focused board; requires account |
| Jira/Asana/Monday | Enterprise team tools — far too heavy for personal use |
| Lightweight OSS (Nullboard, Kanbany) | No delegation tracking, no due-date aging, no filtering, no dashboard, poor UX |
| Text files/sticky notes | No persistence, no visual signals, no search |

SI-Todo's differentiator is the specific combination: drag-and-drop Kanban + delegation tracking + priority color-coding + due-date aging + filtering/dashboard — in a zero-auth, persistent, localhost package. No existing tool bundles all five.

## Who This Serves

**One user: an engineering leader managing multiple teams and workstreams.**

This is a personal tool, not a team platform. The user needs to:
- Track their own tasks and commitments across workstreams
- Monitor delegations (who, what, when)
- Spot overdue and at-risk items instantly via visual signals
- Access the board from their laptop browser (localhost)
- Trust that data persists across sessions, restarts, and shutdowns

## Success Criteria

This tool is working when:
- **Zero dropped balls** — Nothing assigned or delegated falls through the cracks because it wasn't visible
- **Instant status awareness** — Opening the board immediately tells you what's overdue, what's at risk, and what's on track
- **Reliable persistence** — Data survives server restarts, VDI shutdowns, and browser closures without exception
- **Daily habit** — The board becomes the first tab opened in the morning and the last checked before end of day

## Scope

### V1 — In Scope
- 6-column Kanban board with drag-and-drop
- Card creation/editing (title, description, assignee, due date, priority, tags)
- Due-date aging with configurable thresholds (default: green 7+ days, yellow 3–7 days, red <3 days, pulsing red overdue)
- User-customizable card accent colors (coexists with aging via border/accent vs background)
- Filter/search by assignee, priority, due date, tag
- Summary dashboard (totals, overdue, by assignee, by priority)
- Light/dark mode toggle with dotted grid background option
- Persistent backend storage (file-based or SQLite) on localhost
- Desktop browser UI (no mobile required)
- Card archive/delete (prevent Done column from growing unbounded)
- JSON export/import for backup and data portability
- No authentication required

### V1 — Explicitly Out of Scope
- Multi-user or collaboration features
- User authentication/login
- Sprint management, burndown charts, Gantt views
- Third-party integrations
- Mobile-specific design or responsive layouts
- Cloud sync or multi-device sync
- AI-powered features
- Notifications or reminders

## Risks and Mitigations

| Risk | Mitigation |
|------|-----------|
| **Data loss** — Single-machine storage with no cloud redundancy | JSON export/import in V1 provides manual backup. Future: optional sync layer. |
| **Habit formation** — Single-user tools risk abandonment without external accountability | Due-date aging provides passive urgency signals. The board's visual state creates its own pull. |
| **No-auth security** — If ever moved off localhost, board data is exposed | V1 runs on localhost only. Future: optional lightweight auth if network deployment is needed. |

## Vision

If V1 succeeds — if it becomes the daily command center — future versions could explore:
- **Optional sync** across devices (simple cloud backend or file-based sync)
- **Recurring tasks** for regular responsibilities
- **Quick-capture** via keyboard shortcut or mobile widget
- **Archive and history** — searchable log of completed items for accountability and review
- **1:1 prep view** — Filter delegated items by assignee to generate instant prep sheets for 1:1 meetings
- **Weekly digest export** — Summarize the week's card movements for status reporting

The north star is simple: one screen that tells an engineering leader exactly where everything stands, every time they look at it.
