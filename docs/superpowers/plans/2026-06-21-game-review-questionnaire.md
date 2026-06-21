# Game Review Questionnaire Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Hebrew/RTL "ביקורת משחק" (Game Review) questionnaire page to the syllabus site that emails each submission to the teacher as a table and accumulates rows in a Google Sheet, reachable from a chip in lesson 42.

**Architecture:** A new standalone `game-review.html` (themed like `assessment.html`) walks the student through name+game + 6 questions, then POSTs to the **existing** Google Apps Script web app. The script gets a new `review` action that emails an HTML table to the teacher and appends a row to a new "ביקורות משחק" tab in the same results spreadsheet. A link-chip in the lesson-42 card of `index.html` points to the page.

**Tech Stack:** Static HTML/CSS/vanilla JS (no build, no test framework — this is a GitHub Pages static site). Google Apps Script (`private/Code.gs`) for the backend. Fonts via Google Fonts. Verification is manual (browser locally; end-to-end only after deploy).

## Global Constraints

- Language/direction: Hebrew, `dir="rtl"`, `lang="he"`. All UI copy in Hebrew.
- Visual theme: neo-brutalist — copy the `:root` tokens, fonts, and component
  styles verbatim from `assessment.html` (cream `#fdf4e3`, ink `#1a1a1a`,
  pink/yellow/mint/orange/sky, thick black borders, offset shadows; fonts
  Rubik/Heebo/Space Grotesk/Bungee).
- Backend contract: same Web App URL and `API_TOKEN` as `assessment.html`
  (`API_URL = "https://script.google.com/macros/s/AKfycbzmgmK8D84lfwPcGhnsPT5eg9M35tZ_tGSmKzw4NGKs8wsynM6PQeat4dyYAUfn73RLug/exec"`,
  `API_TOKEN = "2f8ea392126249cd2144433b"`).
- Server token in `private/Code.gs`: `TOKEN = '2f8ea392126249cd2144433b'`,
  `TEACHER_EMAIL = 'shlomin@hityash.org'`.
- `api()` calls use `Content-Type: text/plain;charset=utf-8` (simple request,
  no CORS preflight) with a JSON body containing `token`.
- `private/` is gitignored — **never** commit or push `Code.gs`.
- Illustrations are inline SVG in the site style (thick black stroke, flat
  palette fills), `aria-hidden="true"`. No external images, no AI rasters.
- Survey, not a test: no scoring, no right/wrong, no school/class fields.
- Deploy (Apps Script redeploy + push to `main`) only on explicit user approval.

---

### Task 1: Build `game-review.html` front-end (screens, theme, flow) — no server call yet

Build the full page UI and navigation. Submission is stubbed (logs to console /
shows the finish screen) so the flow is verifiable in the browser before the
backend exists. A working scaffold already exists at
`%scratchpad%/game-review-mockup.html` — adapt it (it has the theme tokens,
intro, 6-question flow, progress bar, finish screen already implemented).

**Files:**
- Create: `game-review.html` (repo root)
- Reference (do not modify): `assessment.html` (theme + `api()` pattern),
  the scratchpad mockup `game-review-mockup.html`.

**Interfaces:**
- Produces (JS state object, consumed by Task 2's submit):
  `data = { name, game, q1, q2, q3, q4, q5, q6 }` where `q1` ∈ {"כן","לא"},
  `q2..q5` are strings, `q6` is a string "1".."10".

- [ ] **Step 1: Create the page from the mockup scaffold**

Copy the structure/CSS/flow JS from the scratchpad mockup into `game-review.html`,
with these production changes:
- Add full `<head>`: charset, viewport, `<title>שאלון ביקורת משחק · גיימינג בחינוך</title>`,
  the favicon `<link>` set copied from `assessment.html` (lines 8–12), and the
  Google Fonts `<link>` (assessment.html line 29).
- Add OG/Twitter meta mirroring `assessment.html` (lines 14–25) but for this page:
  `og:title`/`twitter:title` = "שאלון ביקורת משחק · גיימינג בחינוך",
  description = "מבקרים משחק כמו מקצוענים — שאלון קצר שהביקורת בו נשלחת למורה.",
  `og:url` = `https://lemons-gaming.github.io/gaming-syllabus/game-review.html`.
  (Reuse `assets/assessment-og.png` for the image for now.)
- Remove the yellow `.mockbar` banner from the mockup.
- Keep the masthead, progress bar, intro, 6 question screens, finish screen,
  prev/next nav, and the disabled-until-filled intro validation.

- [ ] **Step 2: Add a `<textarea>` autosize-free style** (already in mockup as `textarea.answer`) and confirm Q2–Q5 use it; Q1 uses two `.opt` buttons (כן/לא); Q6 uses ten `.rbtn` buttons.

- [ ] **Step 3: Verify the flow in a browser**

Run: `python -m http.server 8000` in the repo root, open
`http://localhost:8000/game-review.html`.
Expected: intro gates the start button until both fields are filled; each of the
6 questions shows on its own screen; progress bar advances; prev/next work;
selections persist when navigating back; finish screen shows the entered name+game.

- [ ] **Step 4: Commit**

```bash
git add game-review.html
git commit -m "feat: add Game Review questionnaire page (UI + flow, no backend yet)"
```

---

### Task 2: Wire submission to the existing Apps Script backend

Replace the stubbed finish with a real POST, mirroring `assessment.html`'s
`api()`/`sendReport()` pattern (assessment.html lines 796–803, 968–992).

**Files:**
- Modify: `game-review.html` (script section)

**Interfaces:**
- Consumes: `data` from Task 1.
- Produces (the JSON POST body — Task 3's server consumes this exact shape):
  `{ token, action: "review", name, game, q1, q2, q3, q4, q5, q6 }`.

- [ ] **Step 1: Add the `api()` helper and constants**

```javascript
const API_URL = "https://script.google.com/macros/s/AKfycbzmgmK8D84lfwPcGhnsPT5eg9M35tZ_tGSmKzw4NGKs8wsynM6PQeat4dyYAUfn73RLug/exec";
const API_TOKEN = "2f8ea392126249cd2144433b";
async function api(payload){
  const res = await fetch(API_URL, {
    method: "POST",
    headers: { "Content-Type": "text/plain;charset=utf-8" },
    body: JSON.stringify(Object.assign({ token: API_TOKEN }, payload))
  });
  if (!res.ok) throw new Error("HTTP " + res.status);
  const json = await res.json();
  if (!json.ok) throw new Error(json.error || "server error");
  return json;
}
```

- [ ] **Step 2: Make the last "שליחת הביקורת" click submit instead of jumping straight to finish**

On the final question's Next:
- disable the button + show "שולח ביקורת…",
- call `await api({ action:"review", name:data.name, game:data.game, q1:data.q1, q2:data.q2, q3:data.q3, q4:data.q4, q5:data.q5, q6:data.q6 })`,
- on success → render finish screen,
- on failure → re-enable, show an error line with a "נסו שוב" button that
  retries the same submit; the entered answers stay in `data` (not lost).

```javascript
async function submitReview(btn){
  btn.disabled = true; const old = btn.textContent; btn.textContent = "שולח ביקורת…";
  try {
    await api({ action:"review", name:data.name, game:data.game,
      q1:data.q1, q2:data.q2, q3:data.q3, q4:data.q4, q5:data.q5, q6:data.q6 });
    step = 7; render();
  } catch (err) {
    btn.disabled = false; btn.textContent = old;
    showSubmitError();   // renders a .fb.bad block with a "נסו שוב" button → submitReview
  }
}
```

- [ ] **Step 3: Warm up the server on intro (optional parity)**

In `renderIntro`, after focusing the name field: `api({ action:"ping" }).catch(()=>{});`

- [ ] **Step 4: Verify graceful failure locally**

Run the local server, complete the form, click send. Expected (backend not yet
deployed for `review`, so it returns `{ok:false,error:"unknown action"}`): the
error line + "נסו שוב" appears, answers are preserved. This confirms error handling.

- [ ] **Step 5: Commit**

```bash
git add game-review.html
git commit -m "feat: submit Game Review to Apps Script backend with retry-on-failure"
```

---

### Task 3: Add per-question inline-SVG gaming illustrations

Author 7 small inline SVG illustrations in the site's neo-brutalist style and
render one above each question's text (and on intro). Verify visually.

**Files:**
- Modify: `game-review.html`

**Interfaces:**
- Consumes: the per-question render in Task 1.
- Produces: an `ILLO[stepIndex]` lookup (0=intro … 6=Q6) returning an SVG string;
  rendered inside a `figure.shot`-style block above `h2.qtext`.

- [ ] **Step 1: Add an illustration container style**

Reuse `assessment.html`'s `figure.shot` look (white bg, 3px ink border, 14px
radius, `6px 6px 0 var(--yellow)` shadow) sized for an icon (e.g. height ~150px,
centered SVG). Add `aria-hidden="true"` on the figure.

- [ ] **Step 2: Author the 7 SVGs** (thick black `stroke:#1a1a1a` ~2.5–3, flat fills from the palette). Worked example for the intro (game controller + star) — follow this convention for the rest:

```html
<svg viewBox="0 0 120 80" width="150" height="100" aria-hidden="true">
  <rect x="14" y="26" width="92" height="40" rx="20" fill="#38bdf8" stroke="#1a1a1a" stroke-width="3"/>
  <circle cx="38" cy="46" r="4" fill="#1a1a1a"/><rect x="32" y="42" width="12" height="8" rx="2" fill="none" stroke="#1a1a1a" stroke-width="3"/>
  <rect x="34" y="38" width="8" height="16" rx="2" fill="#1a1a1a"/><rect x="30" y="42" width="16" height="8" rx="2" fill="#1a1a1a"/>
  <circle cx="80" cy="42" r="5" fill="#ff4f93" stroke="#1a1a1a" stroke-width="2.5"/>
  <circle cx="92" cy="50" r="5" fill="#ffd23f" stroke="#1a1a1a" stroke-width="2.5"/>
  <path d="M60 4 L64 14 L75 14 L66 21 L69 32 L60 25 L51 32 L54 21 L45 14 L56 14 Z" fill="#ffd23f" stroke="#1a1a1a" stroke-width="2.5" stroke-linejoin="round"/>
</svg>
```

Mapping (each its own SVG in the same style):
- intro: controller + star (above)
- Q1 (האם זה משחק?): arcade/controller with a bold "?" badge
- Q2 (מה הופך למשחק?): interlocking puzzle pieces / building blocks
- Q3 (מכניקה): two gears + a joystick stick
- Q4 (שלבים): three ascending platform blocks with a checkered flag
- Q5 (מה לימד): light bulb with rays
- Q6 (דירוג): trophy with stars

- [ ] **Step 3: Render the illustration per screen**

In `renderIntro` and `renderQ`, inject `ILLO[step]` inside the figure block
above the heading.

- [ ] **Step 4: Visual verification**

Run the local server and click through every screen. Expected: each screen shows
its themed SVG; nothing overflows on mobile width (resize to ~360px).
**Checkpoint:** show the user (screenshot or local URL) for sign-off on the art.

- [ ] **Step 5: Commit**

```bash
git add game-review.html
git commit -m "feat: add per-question neo-brutalist SVG illustrations"
```

---

### Task 4: Extend the Apps Script backend (`private/Code.gs`)

Add the `review` action without touching the existing assessment logic. Writes
to a new tab in the SAME spreadsheet (`SHEET_ID`).

**Files:**
- Modify: `private/Code.gs` (gitignored — edited locally, pasted into the Apps
  Script editor by the teacher; never committed)

**Interfaces:**
- Consumes: the POST body from Task 2:
  `{ token, action:"review", name, game, q1..q6 }`.
- Produces: `{ ok:true }` on success; appends a row; sends an email.

- [ ] **Step 1: Add the `review` case to `doPost`'s switch**

```javascript
    case 'review':
      return submitReview(req);
```
(place alongside the existing `submit` case)

- [ ] **Step 2: Add `submitReview`**

```javascript
function submitReview(req) {
  const name = String(req.name || 'ללא שם').slice(0, 80);
  const game = String(req.game || 'ללא שם').slice(0, 120);
  const q1 = String(req.q1 || '—').slice(0, 10);
  const q2 = String(req.q2 || '—').slice(0, 2000);
  const q3 = String(req.q3 || '—').slice(0, 2000);
  const q4 = String(req.q4 || '—').slice(0, 2000);
  const q5 = String(req.q5 || '—').slice(0, 2000);
  const q6 = String(req.q6 || '—').slice(0, 10);
  const when = Utilities.formatDate(new Date(), 'Asia/Jerusalem', 'dd/MM/yyyy HH:mm');

  const rows = [
    ['1. האם זה משחק?', q1],
    ['2. מה הופך את זה למשחק?', q2],
    ['3. מכניקה', q3],
    ['4. שלבים', q4],
    ['5. מה המשחק לימד', q5],
    ['6. דירוג (1-10)', q6 + '/10']
  ].map(function (r) {
    return '<tr><td style="padding:6px 12px;border:1px solid #ddd;font-weight:bold;vertical-align:top">' +
      esc(r[0]) + '</td><td style="padding:6px 12px;border:1px solid #ddd">' + esc(r[1]) + '</td></tr>';
  }).join('');

  const html =
    '<div dir="rtl" style="font-family:Arial,sans-serif;max-width:680px">' +
    '<h2 style="margin:0 0 4px">ביקורת משחק · גיימינג בחינוך</h2>' +
    '<p style="margin:0 0 16px;color:#666">' + when + '</p>' +
    '<table style="border-collapse:collapse;margin-bottom:16px">' +
    '<tr><td style="padding:6px 12px;border:1px solid #ddd;font-weight:bold">שם התלמיד</td><td style="padding:6px 12px;border:1px solid #ddd">' + esc(name) + '</td></tr>' +
    '<tr><td style="padding:6px 12px;border:1px solid #ddd;font-weight:bold">שם המשחק</td><td style="padding:6px 12px;border:1px solid #ddd">' + esc(game) + '</td></tr>' +
    '</table>' +
    '<table style="border-collapse:collapse">' + rows + '</table></div>';

  const plain = 'ביקורת משחק · ' + name + ' · ' + game + '\n' +
    'האם משחק: ' + q1 + ' | דירוג: ' + q6 + '/10';

  try {
    logReviewToSheet({ when: when, name: name, game: game,
      q1: q1, q2: q2, q3: q3, q4: q4, q5: q5, q6: q6 });
  } catch (err) { /* still email even if the sheet write fails */ }

  MailApp.sendEmail({
    to: TEACHER_EMAIL,
    subject: 'ביקורת משחק · ' + name + ' · ' + game,
    body: plain,
    htmlBody: html
  });
  return out({ ok: true });
}
```

- [ ] **Step 3: Add `getReviewsSheet` + `logReviewToSheet`**

```javascript
/** לשונית הביקורות בתוך אותו קובץ גיליון של שאלון הסיכום. */
function getReviewsSheet() {
  const ss = getResultsSheet().getParent();   // same spreadsheet (creates it if needed)
  let sheet = ss.getSheetByName('ביקורות משחק');
  if (!sheet) {
    sheet = ss.insertSheet('ביקורות משחק');
    const header = ['תאריך ושעה', 'שם התלמיד', 'שם המשחק', 'האם משחק?',
      'מה הופך למשחק', 'מכניקה', 'שלבים', 'מה לימד', 'דירוג (1-10)'];
    sheet.appendRow(header);
    sheet.setFrozenRows(1);
    sheet.getRange(1, 1, 1, header.length).setFontWeight('bold');
  }
  return sheet;
}

function logReviewToSheet(r) {
  getReviewsSheet().appendRow([r.when, r.name, r.game, r.q1, r.q2, r.q3, r.q4, r.q5, r.q6]);
}
```
(Note: `getResultsSheet()` already exists and returns the first sheet of the
shared spreadsheet; `.getParent()` gives the spreadsheet so the new tab lands
in the same file.)

- [ ] **Step 4: Static sanity check**

Re-read the edited `Code.gs`: confirm `review` case present; `submitReview`,
`getReviewsSheet`, `logReviewToSheet` defined; `esc`, `out`, `getResultsSheet`,
`TEACHER_EMAIL`, `TOKEN` referenced exist; existing `submit`/`check`/`ping`
untouched. (Cannot run Apps Script locally — real verification is in Task 6.)

- [ ] **Step 5: No commit** (file is gitignored). Note the change is ready to paste into the editor.

---

### Task 5: Add the entry chip in lesson 42 (`index.html`)

**Files:**
- Modify: `index.html` (the lesson-42 `<article>`, around lines 3246–3266, inside
  its `.session-links`)

**Interfaces:**
- Consumes: `game-review.html` at the site root.

- [ ] **Step 1: Add a link-chip to lesson 42's `.session-links`**

In the article whose `.session-num` is `42` ("רפלקסים — מצא את הצורה"), inside
`<div class="session-links">`, after the existing play chip, add:

```html
<a class="link-chip" href="game-review.html" target="_blank" rel="noopener noreferrer"><svg class="icon"><use href="#i-play"/></svg>ביקורת משחק</a>
```
(Pick an existing `<symbol>` id that fits — `#i-play` or another already defined
in the file's `<svg>` sprite. Verify the chosen id exists before using it.)

- [ ] **Step 2: Verify locally**

Run the local server, open `index.html`, expand World 8, scroll to lesson 42,
confirm the "ביקורת משחק" chip appears and links to `game-review.html`.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: link Game Review questionnaire from lesson 42 card"
```

---

### Task 6: Deployment instructions + end-to-end verification (requires explicit approval)

**Files:**
- Create/Update: `private/הוראות-חיבור.md` (append a "ביקורת משחק" section) —
  gitignored, for the teacher.

- [ ] **Step 1: Write redeploy instructions**

Append to `private/הוראות-חיבור.md`: paste the updated `Code.gs` into the Apps
Script editor → Deploy → Manage deployments → Edit (pencil) → Version: New
version → Deploy. Same URL/token, no front-end change needed. The "ביקורות משחק"
tab is created automatically on the first submission.

- [ ] **Step 2: (User) redeploy the Apps Script** — only after explicit OK.

- [ ] **Step 3: End-to-end test** (after redeploy)

Open the deployed/local `game-review.html`, submit one test review. Expected:
finish screen shows; an email arrives at shlomin@hityash.org with the table; a
new row appears in the "ביקורות משחק" tab of the results spreadsheet.

- [ ] **Step 4: (User) approve deploy of the site**

Merge `feature/game-review-questionnaire` → `main` and push (GitHub Pages goes
live in 1–2 min). Only on explicit confirmation per the deploy-confirmation rule.

---

## Self-Review

**Spec coverage:**
- Front-end page (themed, RTL, intro+6Q+finish) → Task 1 ✓
- Server submission + error handling → Task 2 ✓
- Illustrations → Task 3 ✓
- Backend `review` action + new tab in shared sheet + email table → Task 4 ✓
- Lesson-42 chip integration → Task 5 ✓
- Deployment (Apps Script redeploy + site push, approval-gated) + e2e → Task 6 ✓
- Out-of-scope items (no scoring, no school/class, no homepage banner) respected ✓

**Placeholder scan:** No TBD/TODO; concrete code for page wiring and full
`Code.gs` functions. SVG art is specified with a worked example + per-question
mapping and a visual-verification checkpoint (creative assets are iterated with
the user, not pre-frozen) — acceptable given no test framework and a visual gate.

**Type consistency:** POST body `{token, action:"review", name, game, q1..q6}`
produced in Task 2 matches `submitReview(req)`'s reads in Task 4. Sheet header
(9 cols) matches `logReviewToSheet` row (9 values). `getReviewsSheet` reuses
existing `getResultsSheet().getParent()`.
