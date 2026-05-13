# Second Brain — Session Handoff

**Last updated**: 2026-05-13
**Live URL**: https://second-brain-app-self.vercel.app
**GitHub**: https://github.com/yogeshb-97/second-brain-task-hub
**Local path**: `/Users/yogeshb/Desktop/second-brain-app/`

## What this is
Single-file HTML/CSS/JS gamified task PWA. No build step, no framework. Tailwind CDN + Geist + Material Symbols. Deployed on Vercel (auto from GitHub main) and as a PWA (installable on iPhone home screen).

## Current state — v4 shipped today

### Views (5 tabs)
1. **Today** — `list==='today'` tasks, grouped undone → done, plus a "Tomorrow" section below for `list==='tomorrow'` tasks
2. **Pending** — backlog with search + difficulty-count pills
3. **Bucket** — someday/maybe goals
4. **Habits** — weekly streak dots, habit ring cards, consistency heatmap
5. **Kanban** — To Do / In Progress / Done columns with drag-drop on desktop, vertical stack + Start/Done/Reset buttons on mobile

### Data model
```js
state.tasks[i] = {
  id, title,
  desc,              // optional multi-line description
  diff: 'Medium',    // defaulted, no UI control
  xp: 150,           // defaulted, no UI control
  tag,
  list: 'today' | 'tomorrow' | 'pending' | 'bucket',
  status: 'todo' | 'inprogress' | 'done',
  dueDate: 'YYYY-MM-DD' | '',
  xpAwarded: bool    // XP credited only on first completion (no farming)
}
```
localStorage key: `sb_v4`. Theme key: `sb_theme`. Migration from `sb_v3` lives in `load()`.

### Features
- **Dark / light theme** — softer slate dark palette (`#1c2030` bg / `#272c42` card), persists in `sb_theme`. Anti-flash sync script in `<head>` before CSS.
- **Tomorrow auto-promote** — on date rollover, all `list:'tomorrow'` tasks flip to `'today'`. Toast shown if N>0.
- **Add Task modal** simplified:
  - Title
  - Description (multi-line, optional)
  - Add To (Today / Tomorrow / Pending / Bucket)
  - Tag
  - Due Date — dropdown quick picks (No date / Today / Tomorrow / This weekend / Next week / Pick a date…)
  - Difficulty + XP fields **removed** from UI (silently default to Medium/150)
- **Header** stripped — no more "Level X · Navigator · 3,250 XP" or progress bar. Just ⚡ Second Brain + theme btn + Add Task.
- **Habits Consistency** heatmap card — clean `1px solid var(--border)` (was heavy indigo).
- **Mobile Kanban** — stacks columns vertically (was unusable 85vw scroll).
- **PWA** — `manifest.json`, `sw.js` (cache `sb-v2`), icons 180/192/512.

## File map
| Path | Purpose |
|---|---|
| `index.html` | entire app, ~1080 lines |
| `sw.js` | service worker, cache-first |
| `manifest.json` | PWA manifest |
| `icon-{180,192,512}.png` | app icons |
| `vercel.json` | static deploy config |
| `.vercel/project.json` | linked project (gitignored) |
| `README.md` | basic project description |

## Deploy flow
```bash
cd /Users/yogeshb/Desktop/second-brain-app
git add . && git commit -m "..." && git push
/opt/homebrew/bin/vercel --prod --yes
```
GitHub Pages also auto-builds at `yogeshb-97.github.io/second-brain-task-hub`.

## Quirks / gotchas
- After deploy, the PWA on iPhone may serve cached old `index.html` — bump `sw.js` `CACHE` string to force refresh, or have user long-press → Remove App → reinstall.
- `state.lastOpenDate` drives tomorrow→today promotion. Don't remove.
- Drag-drop uses Pointer Events API; cards have `touch-action:none` on desktop, `touch-action:auto` on mobile (<640px) so page scrolls work.
- Task `desc` field added in last commit — old tasks in localStorage won't have it; the render checks `task.desc?...:''`.

## Open requests (not yet built)
1. **Google Calendar two-way sync** — push tasks with `dueDate` (+ optional `dueTime`) to user's calendar, surface calendar events in app.
2. **Task time field** — pair with `dueDate` to support scheduling at e.g. 14:30. Required before any calendar push has meaning.

See "Next ideas" below.

## Next ideas (in priority order)
1. **Add `dueTime` field** — second input in modal (alongside date dropdown). Pure local change, low risk. Renders inline with the date pill. ~30 lines.
2. **Google Calendar integration** — see feasibility notes below.
3. **Recurring tasks** — daily/weekly repeat for habits-adjacent tasks
4. **Edit task** — currently only add/delete; no inline edit
5. **Reorder tasks** — drag within a list
6. **Bulk actions** — multi-select + move/delete
7. **Better app icons** — current ones are programmatic lightning bolts
8. **Notifications** — local reminders at `dueTime` (needs Web Push or local schedule)

## Google Calendar — feasibility notes
- **Required**: OAuth 2.0 client in Google Cloud Console, scope `https://www.googleapis.com/auth/calendar.events`. The redirect URI must be a real domain — works on Vercel, won't work from `file://`.
- **Auth flow**: Google Identity Services JS library, popup-based consent. Token is short-lived; refresh handled client-side or via a tiny serverless function.
- **Cost**: Calendar API is free for personal use. No server needed if we accept tokens-in-localStorage (acceptable for a personal-use PWA, not for multi-user).
- **Sync model**: one-way push first (Second Brain → Calendar) is ~150 lines. Two-way (also pull events as read-only tasks) doubles that and introduces conflict resolution. Recommend one-way push to start — every task with `dueDate` creates/updates a Google Calendar event keyed by task ID. Delete in app → delete in calendar.
- **Prereq**: `dueTime` field, otherwise events become all-day blobs which is rarely useful.
- **Estimated work**: 1 session for the GCal client wrapper + auth UI + push-on-save logic. Add an "API key / OAuth setup" page once.

## Memory references
- Project memory: `~/.claude/projects/-Users-yogeshb/memory/project_second_brain_app.md`
- Plan file (v4 build): `~/.claude/plans/federated-marinating-hollerith.md`
