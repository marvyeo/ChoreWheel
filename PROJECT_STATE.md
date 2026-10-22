# ChoreWheel — Project State

> **Next chat name: `ChoreWheel ver 2`**
> Claude: your FIRST reply in a new chat must start with "Rename this chat to: **ChoreWheel ver 2**" (you can't rename chats yourself). In your first commit this session, bump the number here by 1.

**State version:** 1 (built 2026-10-20 from chat "Life Plan Roadmap ver 2")
**Maintenance rule:** When ported to a new chat, name it "ChoreWheel ver X" (X = previous + 1). Update this file at the end of each session. The Life Plan Roadmap chat (conductor) may also change this app; always `git pull` before working.

## ⚙ Session Rules (every chat that works on this app follows these)

**1. Pick the cheapest model that can do the job.** At the start of every session Claude names the model it recommends and the estimated size (below). If the chat is on a pricier model than needed, Claude says so before starting.

| Model | Use for |
|---|---|
| **Haiku** | State-file edits, text or label changes, small data packs, lookups, "where is X" questions |
| **Sonnet** (default) | Features, bug fixes, refactors, data processing (e.g. novel volumes, PullList), most app work |
| **Opus** | Only for new-app builds, cross-app architecture, or a bug Sonnet has failed on twice. Claude says why before using it |

**2. Every push updates this file in the same commit.** No exceptions, including PR branches and one-line fixes.
- `git pull` before starting, so you build on the latest state.
- Add a dated line to the change log (what changed and why, in plain words) and update any section the change made wrong.
- If a session ends without a push, nothing changes here.

**3. Size the work first, and plan the pause points.**
- Before starting, Claude lists the steps and sizes the job: **S** (under ~20k tokens), **M** (~20–60k), **L** (over ~60k).
- **M and L jobs are split into checkpoints.** Each checkpoint ends with a commit and a state update, so a cut-off session loses nothing.
- **Pause** when usage is around half the window, when the chat has grown long (heavy file reads, ~30+ turns), or before a step that won't finish in what's left. Claude can't see the remaining credit directly, so on L jobs it asks Marvin to check usage at each checkpoint.
- **On pause,** Claude writes a `▶ RESUME HERE` block at the top of this file: what's done, the exact next step, branch and files, and the first command to run. It also gives Marvin a one-line restart prompt ("Continue <App> ver X from RESUME HERE"). The next session starts from that block and deletes it once the step is done.

**4. Chat naming.** Use the **Next chat name** line at the top of this file; tell Marvin in the first reply, then bump it.

## What it is
Household chore checklist and reminder for Marvin and Caroline. Covers weekly, monthly, quarterly and yearly chores, plus "as needed" ones (laundry, ironing) that only record when they were last done.

| Item | Value |
|---|---|
| Repo | `marvyeo/ChoreWheel` (public, no personal data in the repo) |
| Live URL | https://marvyeo.github.io/ChoreWheel/ (demo without sign-in: `?demo`) |
| Stack | Single `index.html` (vanilla JS, own CSS, no CDN framework), `sw.js` (network-first, cache `chorewheel-v1`), `manifest.json`, `icons/` |
| Data | Google Drive `chorewheel-data.json`, scope `drive.file`, shared OAuth client `669494100250-…` |
| Household | Same pattern as Financial Planner: file found via remembered id, then shared-with-me, then own. Second person: "Open the shared list" (Google Picker, same `PICKER_API_KEY`). Share from Settings → Household. |
| Sync | Before each save, check Drive `version`; if changed, 3-way merge (chores per field, log = union by id, `lastDone` recomputed from log). Pulls remote changes when the app comes back to the foreground. |

## Data model (`chorewheel-data.json`)
```json
{ "app": "chorewheel", "version": 1,
  "chores": [{ "id", "name", "area", "freq": { "unit": "day|week|month|year|asneeded", "every": 1 },
               "assignee": "marvin|caroline|either", "lastDone": "YYYY-MM-DD|null", "firstDue": "YYYY-MM-DD",
               "snoozedUntil": "YYYY-MM-DD (optional: manual due date or snooze; cleared on tick)", "notes", "archived": false }],
  "log": [{ "id", "choreId", "date", "by": "marvin|caroline", "at": 0, "deleted": false }],
  "settings": { "household": { "members": [{ "email", "who" }] } },
  "upcoming": [ /* Life Plan envelope, type "chore" */ ] }
```
- Next due = `snoozedUntil` if set, else `lastDone + interval`, else `firstDue`. Month and year steps clamp to month end (31 Jan + 1 month = 28 Feb).
- `upcoming[]`: chores due within 30 days. Overdue chores are dated today with priority high. Label includes the assignee unless "Either".

## Features (v1.0)
- **To do:** Overdue, Today, This week, Later this month, As needed. Filter: Everyone, Marvin or Caroline (Caroline's view includes "Either" chores). Tap the circle to tick, with Undo.
- **Chore sheet:** Done today, Done on a date, frequency presets (weekly, fortnightly, monthly, quarterly, yearly) or custom, who, area, due date, notes, snooze 1 day or 1 week, archive.
- **All chores** grouped by area. **History** log with a per-person count for this month. **Settings:** who am I, household share, suggested chores, export JSON.
- **Seeded chores:** Clean sink (weekly), Start robovac (weekly), Washing clothes and Ironing (as needed), Aircon servicing (quarterly), Rotate car tyres (6-monthly). The last two are assigned to Marvin with placeholder first-due dates, so edit them.

## Reminders
- In-app list, plus `upcoming[]` for the Life Plan dashboard.
- Optional: a weekly Claude scheduled task that reads `chorewheel-data.json` and sends a push and email "chores this week" summary. Not created yet; it needs the Drive file to exist first.

## Tested (2026-10-20)
Headless demo: tick and undo, add custom chore, manual due date, month-end interval maths, two-device merge, `upcoming[]` shape, no page errors.

## Open items
- [ ] Marvin: create the empty repo `marvyeo/ChoreWheel` and enable Pages (main / root). Claude pushes the code.
- [ ] First sign-in (Marvin): "Start a new chore list", set real due dates for aircon and tyres, then Settings → Household → share with Caroline.
- [ ] Decide whether to add the weekly push or email summary task.
- [x] Life Plan Dashboard reads `chorewheel-data.json` (type `chore`).

## Change log
- **v1.0 (2026-10-20):** Initial build.
- **2026-10-21:** new icon from the ecosystem icon set (`sw.js` cache bumped).
- **2026-10-22:** manifest id set to `/ChoreWheel/` (the old `./` made Android treat all apps as one).
- **2026-10-22:** added Session Rules and `CLAUDE.md`.
