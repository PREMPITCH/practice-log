# Handoff — บันทึกการซ้อม (Spec 02 · Practice Timer & Log)

**Date:** 9 November 2026 (2569)
**Workspace:** `/Volumes/Crucial2TB/All Codes/FAA/Arttechno/App for building/02 Practice Log `
**Repository:** [yos2568/practice-log](https://github.com/yos2568/practice-log) · public · branch `main`
**Shipped commit:** `aa7d391` — “Add practice timer and log app” — 8 Nov 2026 23:53 +0700
**Spec:** `deploy-projects/specs/02-practice-log-spec.md` (FAA / 3500230)
**Status:** Built, browser-tested, committed, and pushed. GitHub Pages is **not enabled** yet. This handoff is committed to the repo.

---

## 0. Resume in 60 seconds

| If you want to… | Do this |
|---|---|
| Open or use the app | Double-click `index.html` (works from `file://`) |
| Deploy it | Upload `index.html` alone to GitHub Pages, Netlify Drop, or Vercel; see §7 |
| Rebuild from the source spec | Paste `deploy-projects/specs/02-practice-log-spec.md` into a new AI chat |
| Re-run the acceptance checks | §5 and the spec’s §11 |
| Change goal limits, copy, or colors | §8 |
| Know what is imperfect | §6 |
| Update this handoff | Edit `handoff.md`, then commit and push it with the app change |

The app is one self-contained file. There is no build step, no dependency install, no server, and no network request at runtime.

---

## 1. What this is

A daily practice stopwatch and log for first-year Western music performance students (Chulalongkorn University, Faculty of Fine and Applied Arts, course 3500230).

Core loop: **start → practice → stop → pick/type what you practiced → save → see the last 7 days against a daily goal.**

It deliberately makes no sound, calls no AI, collects no personal data, and stores everything in the browser’s `localStorage`.

---

## 2. Current state

| Item | Value |
|---|---|
| App file | `index.html` — 51,145 bytes, 1,440 lines |
| Tracked files | `index.html`, `.gitignore`, `handoff.md` (this file) |
| Working tree | Clean |
| Remote | `origin` → `https://github.com/yos2568/practice-log.git` |
| Branch | `main` tracking `origin/main` |
| App commit | `aa7d391` — app + `.gitignore` |
| Branch head | app commit + the handoff commit that added this file (see `git log`) |
| App data in repo | None — the app opens empty; test entries were cleared from the test browser |
| GitHub Pages | 404 / not configured |

`.gitignore` ignores macOS AppleDouble files (`._*`) and `.DS_Store`; this volume produces `._index.html` next to edited files.

### Verified environment

- **Chrome 157 headless on macOS**, tested through the Chrome DevTools Protocol against the local `file://` URL.
- Layout checked at **320, 360, 390, 430, and 844×390 landscape**.
- Real iOS Safari and Android Chrome were **not** available in this environment; see §6.
- The file was **not** uploaded to GitHub Pages / Netlify / Vercel in this session.

---

## 3. Behavior map (for future edits)

### Timer

- States: `idle → running ⇄ paused → saving → idle`.
- Elapsed time is **always derived from timestamps**, never incremented:
  - running: `accumulatedMs + (Date.now() − startedAtMs)`
  - display refresh every 250 ms only while running.
- Pauses add the running delta to `accumulatedMs`; clock jumps backward are clamped to `lastElapsedMs`.
- Persisted on every state change to `practice-log-timer-v1`.
- On load:
  - `paused` restores silently;
  - `saving` restores the save sheet;
  - `running` restores and shows `พบเวลาที่ค้างอยู่ (mm:ss)` with two choices.
- While the restore notice is visible, the normal timer buttons are hidden; the timer is already running. `ซ้อมต่อจากเดิม` dismisses the notice; `หยุดและบันทึกตอนนี้` stops into the sheet.
- A running state older than 12 hours is capped at 12 hours (`12:00:00`), shows `เวลานานผิดปกติ ตัดที่ 12 ชั่วโมง`, and focuses `หยุดและบันทึกตอนนี้`.
- Clock format: `mm:ss` below one hour, `h:mm:ss` from one hour.

### Save sheet

- Appears inline after Stop; timer freezes while `saving`.
- Presets: `สเกล` `อาร์เปจโจ` `เพลงหลัก` `อ่านโน้ต` `ฝึกหู` `เทคนิค`.
- Plus up to 5 most recent distinct non-preset labels (case-insensitive dedupe).
- Label ≤ 60 chars, required, trimmed; note ≤ 200 chars, optional.
- Under 60 s shows `ซ้อมไม่ถึง 1 นาที จะบันทึกก็ได้` and still allows saving.
- `ไม่บันทึก` uses a 4 s two-step confirmation.
- Save writes an entry, toasts `บันทึกแล้ว` for 2.5 s, returns to idle, and re-renders chart + log immediately.

### Week chart

- Seven local calendar days ending today; weekday labels `อา จ อ พ พฤ ศ ส`.
- Minutes = sum of `seconds` per day, divided by 60 and rounded for display.
- Bar scale = max(goal, week max); zero days show a baseline dot; taller-than-goal days get `✓`.
- Goal line is 1 px dashed `--line-strong`, labelled `เป้า {n} นาที`.
- Today’s bar uses `--action`; other bars use `--blue`.
- Today progress text + 6 px progress bar (`--good` when met, `--action` otherwise, visually capped at 100%).
- Summary: `รวม {n} นาที · ซ้อมติดกัน {n} วัน`.
- Streak: consecutive local days with ≥60 s, ending today, or yesterday when today has none.
- Hidden `<table>` carries the text alternative (day + minutes).

### Log

- 30 most recent entries, grouped by local date: `วันนี้`, `เมื่อวาน`, then `จ. 9 พ.ย. 2569` style.
- Row: duration, label, optional note, `HH:mm`, delete button.
- Durations: `< 1 นาที`, `{m} นาที`, or `{h} ชม. {m} นาที` from 60 minutes up.
- Delete uses a 4 s two-step confirmation.

### Settings and data (`<details>`)

- Goal stepper 5–300, step 5, default 30; updates the chart immediately and persists.
- Wake-lock switch, hidden when `navigator.wakeLock` is unavailable; re-requested on `visibilitychange` while running.
- Copy weekly summary (exact text format from spec §7); copy full backup JSON.
- Paste-import: validates `{ entries: [...] }`, merges by `id`, applies valid settings, reports `นำเข้า {n} รายการ` or `ข้อมูลไม่ถูกต้อง ลองคัดลอกใหม่`; caps at newest 500.
- Clear all: two-step `ยืนยันล้างทั้งหมด`; clears entries and the saved timer, keeps settings.
- Privacy line included.

### Storage

| Key | Shape |
|---|---|
| `practice-log-entries-v1` | `[{ id, endedAt, seconds, label, note }]`, newest first, max 500 |
| `practice-log-timer-v1` | `{ state, startedAtMs, accumulatedMs }` |
| `practice-log-settings-v1` | `{ goalMinutes, wakeLock }` |

- Every read/write is wrapped in `try/catch`, with an in-memory fallback.
- When storage is blocked, the app shows `เบราว์เซอร์นี้เก็บบันทึกไม่ได้ ข้อมูลจะหายเมื่อปิดหน้านี้` and keeps working for the session.
- The fallback only supplies data when storage is genuinely unavailable; when storage works, reads always go to `localStorage` so a clear/delete from another tab is respected.
- Two tabs: re-read entries/settings on `visibilitychange` and `storage`; last write wins.

---

## 4. Design and hard constraints

- `<html lang="th">`, every user-facing string Thai, `<title>บันทึกการซ้อม`.
- Color tokens exactly as the shared design system, light + dark under `prefers-color-scheme`; no hard-coded component colors.
- Manuscript-paper staff: five 1 px `--line-strong` hairlines, 5 px apart, 21 px tall — in the header and at the top of the week and log cards.
- Timer numerals are the hero: 64 px / 700 / tabular-nums. They become 56 px at ≤340 px so `12:00:00` cannot overflow the card.
- Cards: `--surface`, 1 px `--line`, 18 px radius, 16 px padding, 16 px gap.
- Max content 560 px; at >720 px the week and log cards sit in two columns and the timer spans both (app widens to 900 px to make that possible).
- No network requests, no external fonts/images, no audio, no AI, no geolocation/camera/mic/notifications.
- No `alert` / `confirm` / `prompt` / `print`; confirmations are in-page.
- No infinite animation; transitions ≤200 ms; `prefers-reduced-motion` removes the bar transition.
- User-supplied labels, notes, and imported data are rendered with `textContent`, never `innerHTML`.
- Local dates are computed from `endedAt` with local `Date` fields; never UTC string comparison. Thai Buddhist-era dates and month abbreviations are hand-written (no `Intl`).

---

## 5. Verification performed

All 14 acceptance tests from spec §11 were checked in headless Chrome; all **PASS**.

| # | Check | Evidence |
|---|---|---|
| 1 | Opens from `file://`, no console errors, idle copy + empty chart | `พร้อมเริ่มซ้อม`, `รวม 0 นาที`, 7 dots; console/errors empty |
| 2 | Start/pause/resume/stop; pause excluded; background-safe | +30 s simulated `Date.now` jump advanced the clock by 30 s; paused clock stayed frozen |
| 3 | Reload while running restores + notice | `พบเวลาที่ค้างอยู่ (…)` with both actions; clock continued |
| 4 | Save disabled until label; chips fill; trim + 60 cap | blank/space label disabled; 70-char input stored as 60 chars, trimmed |
| 5 | Save writes storage, toast, chart/log update | entry in `localStorage`, `บันทึกแล้ว`, today/bar/total/log changed |
| 6 | 7 columns, Thai labels, dashed goal line + label, `✓` | 7/7 labels `จ อ พ พฤ ศ ส อา`; computed 1 px dashed `--line-strong`; checks on ≥goal days |
| 7 | Streak 3 / 2 / 0 / gap reset | tested all four cases |
| 8 | `วันนี้`, `เมื่อวาน`, BE date | 9 Nov 2026 → `จ. 9 พ.ย. 2569` |
| 9 | Delete + clear two-step, updates | armed text reverted after 4 s; second tap deleted/cleared |
| 10 | Summary copy + backup round-trip | exact 9-line summary; re-import of 6 entries reported `นำเข้า 0 รายการ` |
| 11 | Storage blocked still works | CDP init script made `localStorage` throw; notice shown, timer/save/log worked in memory |
| 12 | No h-scroll at 320, readable chart, 44 px targets | `scrollWidth === innerWidth` at 320–430 + landscape; all visible controls ≥44×44 |
| 13 | Light/dark contrast, reduced motion | used token pairs meet 4.5:1; reduced motion → `transition-duration: 0s` |
| 14 | No network / banned APIs, textContent | only the document request; no alert/confirm/prompt/infinite/CDN; dynamic text via `textContent` |

**Additional checks:** 600-entry import kept newest 500 and remained sorted; cross-tab clear was reflected without resurrecting memory data; stale >12 h restore capped and focused the right button; 320 px with save sheet and settings open had no horizontal overflow; console/error logs stayed empty through all flows.

### Reproduce the main tests manually

1. Open `index.html` from disk in Chrome/Edge.
2. Press `เริ่มซ้อม`, wait, `พัก`, wait, `ซ้อมต่อ`, `หยุดและบันทึก`; check the paused time is not counted.
3. Reload during `running`; confirm the restore notice and elapsed time.
4. Pick a chip or type a label; save; confirm toast and chart/log updates.
5. Open Settings, change the goal, copy the summary, copy the backup, paste it back, import.
6. Resize DevTools to 320 px wide and check no horizontal scrollbar.

For storage-blocked testing, inject a script before load that redefines `window.localStorage` getter to throw, e.g. via CDP `Page.addScriptToEvaluateOnNewDocument`.

---

## 6. Known caveats and deliberate deviations

1. **Phone chart width at 320 px.** The plot is 254 px wide there because 16 px page gutters + 16 px card padding leave 256 px. Each of the 7 columns is still 34 px (≥32 px) and the chart is readable. From 360 px up the plot is 294 px, meeting the spec’s 280 px target.
2. **Clock at ≤340 px is 56 px**, not 64 px, so a 12-hour `12:00:00` cannot overflow at 320 px. At 390 px it is 64 px as specified.
3. **Goal label over bars.** `เป้า {n} นาที` is kept at the right end of the dashed goal line as the spec asks. On weeks where the rightmost bars reach or pass the goal, the label’s opaque background can cover a small slice of a bar. The numeric value and `✓` sit above the bar and remain readable.
4. **Stale-restore warning color.** In light mode `--warn` on `--blue-soft` measures 4.0:1, below 4.5:1. The `เวลานานผิดปกติ…` line therefore uses `--bad` (5.54:1 on `--blue-soft`); `--warn` is still used for the short-session warning on `--surface` (4.71:1).
5. **Restore notice hides normal timer controls** to avoid four competing buttons. The timer itself keeps running behind the notice.
6. **Not tested on real phones or real hosting.** The file is dependency-free and should behave the same, but iOS Safari / Android Chrome and GitHub Pages / Netlify / Vercel were not exercised.
7. **GitHub Pages is off.** The repo is public, so enabling Pages from `main` / root is all that is needed for `https://yos2568.github.io/practice-log/`.
8. **Handoffs are committed in this series.** This repository is public; this file contains no secrets, credentials, or student data. Keep it that way when updating it.

---

## 7. Run and deploy

### Locally

```bash
open "index.html"          # macOS
# or double-click index.html in Explorer / Finder
```

No server, build, or install. Chrome and Edge accept `file://`; the app only needs to read `localStorage` (guarded).

### GitHub Pages (not enabled)

If you want it:
1. Repo → Settings → Pages.
2. Source: **Deploy from a branch** → `main` → `/ (root)`.
3. Wait for the Pages build; URL should become `https://yos2568.github.io/practice-log/`.
4. Open that URL on a phone and repeat the 14 checks.

### Netlify Drop / Vercel

- Netlify Drop: drag the folder containing `index.html` (the file must be named `index.html`).
- Vercel: import the repo; there is no framework/build step, output is the repo root.
- The page works at `/` and at a subpath such as `/practice-log/`; there are no root-relative URLs.

### Git

```bash
cd "/Volumes/Crucial2TB/All Codes/FAA/Arttechno/App for building/02 Practice Log "
git status
git add index.html
git commit -m "…"
git push origin main
```

Remote: `https://github.com/yos2568/practice-log.git`.

---

## 8. Maintenance notes

| Want to change | Where |
|---|---|
| Any Thai UI copy | Search inside `<script>`; copy strings are centralized in `render*`, `summaryText`, `durationText`, `thaiDate` |
| Colors / dark theme | `:root` tokens and the `@media (prefers-color-scheme: dark)` block at the top of `<style>` |
| Goal bounds | `GOAL_MIN`, `GOAL_MAX`, and the `min`/`max`/`step` attributes on `#goalInput` |
| Entry cap / log cap | `MAX_ENTRIES = 500`; `entries.slice(0, 30)` in `renderLog` |
| Chart height / bar scale | `CHART_H`, `BAR_MAX`, and `renderWeek()` |
| Storage keys / import format | `KEY_*` constants, `sanitizeEntry`, `normalizeEntries`, `importBackup` |
| Study timer rules | `currentElapsed`, `startTimer`, `pauseTimer`, `resumeTimer`, `stopTimer`, `initTimer` |

Rules to keep if you edit:

- Keep it a **single file** for deployment; no CDN, no fonts, no images, no modules.
- Keep every runtime request out; test with DevTools Network.
- Keep user text on `textContent`; imported data must stay sanitized.
- Keep local-date logic; do not switch to UTC date strings.
- Keep the 44 px tap-target floor and the 16 px gutters.
- If you change the timer formula, test reload, clock jump backward, and 12-hour stale restore together.

There is no test suite in the repo. The acceptance tests live in the spec and in §5 above.

---

## 9. Next steps

1. **Decide on Pages.** Enable it if the goal is a phone link from the repo; otherwise use Netlify Drop for the class demonstration.
2. **Real-device pass.** Open the app on one iPhone and one Android phone: keyboard, safe areas, `12:00:00` clock, save sheet reachability, and clipboard fallback.
3. **Keep this handoff current.** Add the Pages URL, real-device results, and any new caveats here when they change.
4. Optional: add a short README to the repo (what it is, one-line open/deploy, spec link).
5. Optional: enable GitHub Pages and add the live link to the course site / student handout.
6. The sister apps are separate repos — [rhythm-tap](https://github.com/yos2568/rhythm-tap) and [note-catcher](https://github.com/yos2568/note-catcher). This handoff covers only `practice-log`.

---

## 10. Session log (what was built and fixed)

1. Read spec 02, scaffolded a single-file app with the shared tokens, staff motif, timer state machine, save sheet, week chart, log, settings, import/export, and storage fallback.
2. Browser-tested in an isolated headless Chrome instance at multiple widths, in light/dark, with reduced motion, and with storage blocked.
3. Fixes made after visual/behavioral review:
   - spacing between the label hint and the note field;
   - clipboard fallback now clears stale toasts and tries `execCommand` before showing the read-only textarea;
   - stale-restore warning switched from `--warn` to `--bad` for contrast;
   - restore notice now hides the normal timer buttons;
   - increased value/check clearance above bars so the goal label cannot clip the numbers;
   - fixed the in-memory storage fallback so a cross-tab clear is not resurrected;
   - 56 px clock at ≤340 px;
   - delete during saving refreshes recent-label chips.
4. Cleared all test data from the test browser; the committed file has no data.
5. Initialized Git, added `.gitignore` for AppleDouble/`.DS_Store`, committed `aa7d391`, created the public repo `yos2568/practice-log`, and pushed `main`.
6. Wrote and committed this handoff.
