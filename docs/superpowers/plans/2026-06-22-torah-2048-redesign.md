# Torah 2048 Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restyle the live `torah-2048` game to the "stone tablets / desert" aesthetic (direction C), replace the pale per-book palettes with bold strongly-stepped ramps, and add a favicon.

**Architecture:** Pure visual change to the single `index.html` in the `torah-2048` repo — the 2048 engine, the 54-parashah data, and the menu↔game navigation are untouched. Only CSS (theme), the per-book color ramps, the tile foreground-color rule, and a `<link rel="icon">` change.

**Tech Stack:** Static HTML/CSS/vanilla JS (no build, no test framework). Verification is browser/manual.

## Global Constraints

- File: `C:\Users\shlom\projects\torah-2048\index.html` (separate repo, its own git). Do NOT touch the syllabus repo (the World 8 link is unchanged).
- Hebrew, `dir="rtl"`; do NOT change game logic, parashah data, or navigation.
- Aesthetic = direction C "stone tablets / desert". Visual target reference:
  `C:\Users\shlom\AppData\Local\Temp\claude\C--Users-shlom-projects----------------------\8aec4146-9f91-428b-8fda-79ade7f6e6fa\scratchpad\torah-redesign-mockup.html` (the `.dir-c` panel) — sandstone background `#d9b88a`→`#b98e5e`→`#9c6f43`, board well `#8a653c` with inset shadow, carved-stone tiles (small radius ~6px, inset top-highlight + inset bottom-shadow + drop-shadow bevel), serif display type (`Frank Ruhl Libre`/`Suez One`), brown stone score boxes `#7a5733`.
- Bold per-book ramps (exact, light→deep; array length = parashah count):
  - genesis (12): `#f0eaa8 #d2e07a #a6d44f #5fc04a #28ad5e #119b7a #0e8a86 #0d7480 #125f72 #184a5e #1d3848 #11242e`
  - exodus (11): `#d9ecf2 #aad9ec #6ec3e6 #3aa6de #1f86cf #1769bf #1450a6 #143f88 #15316a #172650 #121a38`
  - leviticus (10): `#f6e7a8 #f4cd6a #efa83f #e8822c #dd5a26 #cc3a2a #b32432 #971a3a #741533 #4d0f24`
  - numbers (10): `#ecd9a0 #e0b95e #d18f3a #c4632e #a83f3e #823a63 #5a3a7e #3b3470 #272a55 #171b38`
  - deut (11): `#ead9f0 #d3a9e6 #bd7ddb #a854c9 #9536b3 #7e2a9c #662586 #4f1f6e #3a1a54 #27143b #180c27`
- Tile fg rule: destination(last) tile = `#f4cd7e`; else luminance(bg) < 0.56 → `#fff`, else `#2a2410`.
- Menu card accent per book (bold mid tone): genesis `#28ad5e`, exodus `#1f86cf`, leviticus `#dd5a26`, numbers `#c4632e`, deut `#9536b3`.
- Favicon: inline SVG data-URI (no external file).
- Deploy (push to torah-2048 `main`) is approval-gated.

---

### Task 1: Redesign — stone-tablet theme + bold palettes + favicon

**Files:**
- Modify: `C:\Users\shlom\projects\torah-2048\index.html`
- Reference (read): the mockup file above (`.dir-c` CSS) for the visual recipe.

**Interfaces:**
- Consumes: existing `BOOKS` array, `bookColors(book)`, `fgFor(bg,isDest)`, `deriveBook`, the engine, and existing CSS class names (`body`, `.board`, `.cells`/`.cell`, `.tile`, `header`/`.title h1`, `.box`, `.btn`, `.goal`/`.dot`, `.ladder`/`.chip`, `.overlay`, `#menu-screen`/`.bookcard`/`.menu-title`, `#game-screen`).
- Produces: same DOM/JS contract; only styles + ramp data + favicon change.

- [ ] **Step 1: Add an explicit `ramp` to every book; remove start/end interpolation**

In the `BOOKS` array, give EACH book a `ramp:[...]` array using the exact hexes above (numbers already had one — keep its values updated to the new bold set above). Remove the now-unused `start`/`end` fields. Update `bookColors(book)` to simply `return book.ramp;` (the `rampColors` interpolation helper can be deleted if no longer referenced). Set each book's `accent` to the bold mid tone listed in Global Constraints (used by the menu cards).

- [ ] **Step 2: Update the tile fg rule**

Change `fgFor` to: `function fgFor(bg,isDest){ return isDest ? '#f4cd7e' : (luminance(bg) < 0.56 ? '#fff' : '#2a2410'); }`

- [ ] **Step 3: Restyle to the stone-tablet/desert theme (direction C)**

Rewrite the CSS to match the `.dir-c` mockup, mapped onto the real classes. Concretely:
- `body` background: `linear-gradient(180deg,#d9b88a,#b98e5e 60%,#9c6f43)` (replace the parchment radial). Keep min-height/centering.
- `.board`: background `#8a653c`; `box-shadow: inset 0 3px 12px rgba(0,0,0,.45)`; keep radius/padding/sizing.
- `.cell` (empty wells): `background:#9c7448; box-shadow:inset 0 2px 5px rgba(0,0,0,.35)`.
- `.tile`: `border-radius:6px`; carved bevel `box-shadow: inset 0 2px 3px rgba(255,255,255,.25), inset 0 -3px 5px rgba(0,0,0,.35), 0 2px 4px rgba(0,0,0,.3)`; keep `font-family:var(--serif)` (Frank Ruhl Libre), weight 900; keep `white-space:nowrap`.
- Header `.title h1`: use a heavier engraved look (`Suez One` if loaded, else Frank Ruhl Libre 900), color `#3a2a16`, subtle light text-shadow; load `Suez+One` in the Google Fonts `<link>`.
- `.box` (score/best): `background:#7a5733; color:#fbe6cf`.
- `.btn`: stone style — `background:#7a5733; color:#fbe6cf` (and the existing hover/active).
- `.goal` text color to suit sand (`#5a4326`); `.dot` keeps the per-book accent already set in startBook.
- `.ladder .chip`: keep per-tile colors (driven by COLORS); ensure chip text uses the same fg rule (already does via COLORS fg).
- `#menu-screen`: same desert background; `.menu-title` in the display serif, color `#3a2a16`; `.bookcard` styled as stone tablets (small radius, carved bevel like `.tile`, the book `accent` as the card background, light text).
- `.overlay`: replace the pale `rgba(246,239,224,.86)` with a sand/stone tone (e.g. `rgba(154,111,67,.92)` with light text), win overlay a darker stone; keep the existing `.btn` styling.
- Keep all layout/sizing vars (`--size`,`--gap`,`--cell`,`--stride`) and the engine-driven inline styles intact.

- [ ] **Step 4: Add the favicon (inline SVG data-URI)**

In `<head>`, add a `<link rel="icon">` with a URL-encoded SVG of a stone tile with a gold aleph (matches the theme):

```html
<link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Crect x='6' y='6' width='52' height='52' rx='10' fill='%23c4632e' stroke='%233a2a16' stroke-width='4'/%3E%3Ctext x='32' y='44' font-size='34' text-anchor='middle' fill='%23f4cd7e' font-family='serif' font-weight='bold'%3E%D7%90%3C/text%3E%3C/svg%3E">
```
(`%D7%90` is the Hebrew letter aleph. Confirm the `<head>` has no other `rel="icon"`.)

- [ ] **Step 5: Verify locally**

Run `cd "C:/Users/shlom/projects/torah-2048" && python -m http.server 8060` (background); `curl -s -o /dev/null -w "%{http_code}" http://localhost:8060/index.html` → 200. Stop server after.
Static-inspect: each book has a `ramp` of the correct length (12/11/10/10/11) with the exact hexes; `bookColors` returns `book.ramp`; `fgFor` threshold 0.56; stone theme CSS applied to board/tiles/menu/overlay; favicon `<link>` present; engine/logic/data untouched. (Full visual check of all 5 books is done by the controller in a browser.)

- [ ] **Step 6: Commit (on a branch in the torah-2048 repo)**

```bash
cd "C:/Users/shlom/projects/torah-2048" && git checkout -b redesign && git add index.html && git commit -m "feat: stone-tablet redesign + bold per-book palettes + favicon"
```

---

### Task 2: Deploy (approval-gated)

**Files:** none; deploy commands only.

- [ ] **Step 1: (Get explicit user approval first.)**

- [ ] **Step 2: Merge to main and push**

```bash
cd "C:/Users/shlom/projects/torah-2048" && git checkout main && git merge --ff-only redesign && git push origin main
```

- [ ] **Step 3: Verify live**

Poll `https://lemons-gaming.github.io/torah-2048/` (GitHub Pages rebuild ~1–2 min); confirm the new stone theme + bold palettes render and a book is playable. The syllabus World 8 link is unchanged (no syllabus push needed).

---

## Self-Review

**Spec coverage:**
- Direction C stone-tablet styling → Task 1 Step 3 ✓
- Bold per-book palettes (5 ramps) → Task 1 Step 1 ✓ (+ fg rule Step 2)
- Favicon → Task 1 Step 4 ✓
- Deploy (approval-gated, torah-2048 only; syllabus link unchanged) → Task 2 ✓
- Out-of-scope (no engine/data/nav change, no syllabus change) respected ✓

**Placeholder scan:** No TBD. Exact ramps, fg rule, favicon data-URI, and concrete CSS mappings given; the mockup `.dir-c` is the visual reference for fine styling. No test framework (browser verification by controller) — expected.

**Type consistency:** `bookColors(book)` returns `book.ramp` (array, length = parashah count); `deriveBook` already maps `cols[i]`→`COLORS[value]={bg,fg}` and `fgFor(cols[i], i===last)`. `accent` per book consumed by `.bookcard` background and the goal dot. No signature changes — only data/style.
