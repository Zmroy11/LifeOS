# The Pipe — Consolidated To-Do (Rounds 17, batches 1–4 + addenda)

Project folder (on user's computer): `the pipe SEP 25`.
Purpose of this doc: hand to Claude Code in that folder so it has full context, verifies what already exists, and builds only what is missing.

---

## 0. Instructions for the building Claude (READ FIRST — credit-saving rules)

The user is credit-constrained. Many items below MAY already be built (earlier sessions ran out of credits mid-way). Work like this:

1. **Audit first, cheaply.** Do NOT read whole files. Use grep/glob for a keyword per item (e.g. `upside`, `mandate`, `stageLabel`, `relativeTime`, `emailQuote`) and read only the matching ranges. Output a one-table audit: item # → `DONE / PARTIAL / MISSING` + file:line evidence. Show it to the user before building.
2. **Build only PARTIAL/MISSING items**, one item at a time, one commit per item. Run the project's existing tests/checks if any; don't invent new test suites.
3. **No sub-agents / workflows unless the user asks.** Exception: the collaboration split (item B1-9 below) was asked to get ONE accuracy-review pass — a single reviewer, not a fan-out.
4. Don't rewrite files wholesale; use targeted edits. Don't re-derive decisions already listed under "Decisions" below.
5. Keep the round-16 per-deal `mandates` field intact (no data loss).
6. User roles: **DM** (primary focus), **SE** (sales engineer / support), **Admin** (account "Admin Account (zach)"). Tour work is DM-view only for now.

### Decisions already made (don't re-ask)
- State map data: seed **placeholder** values; admin edits them.
- If a state is both "can't write" and "no mandate": **RED (can't write) wins.**
- "TS select" is a **PEO sub-type** (value `TS` or `TS select`), admin-editable per state; user will specify which states later.
- Today tab "Starts on the month" = count of started accounts this month. "Forecasted number" = **locked forecast total**.
- Stats date range: presets **plus** custom picker, applied to all individual stats.
- Undo button: plain, generic undo-arrow icon, styled like the other top-bar icon buttons.

### Known gap
- Batch 2 item "calendar is showing this error: ___" — the **error text was never captured**. Ask the user for it (or reproduce by opening Calendar and reading the console) before fixing.

---

## 1. BUGS (do these first — small, high trust impact)

| ID | Bug | Likely cause / where to look |
|---|---|---|
| BUG-1 | Editing stage names in Settings does not update the DM pipeline. | Org-wide stage-label override isn't read by the DM grid (grid probably uses hardcoded/default stage labels or a cached copy). Make grid, filters, stage dropdowns, and exports read the live settings. |
| BUG-2 | Activity log always shows "just now". | Relative-time renderer: timestamp likely set to `now` on render, stored in wrong units (s vs ms), or missing. Show real elapsed time (min/hr/day ago), updating on re-render. |
| BUG-3 | Calendar throws an error. | Error text not captured — see "Known gap". |
| BUG-4 | SE adds a deal for a DM: DM toggle defaults to the SE's own name, forcing a re-select of "Whole team". | Default must be "Whole team"; a specific DM only if the SE chooses one. |
| BUG-5 | BP not adding to the SE account grid correctly. | BP and payroll RC are independent; a BP forecast entry must not imply payroll RC is counted (BP can hit before payroll). Track separately. |
| BUG-6 | Negative accounts not counted toward forecast; should render **red** like in DM pipeline. | SE grid forecast sum + styling. |
| BUG-7 | "In starts" ≠ "sold this month". | Starts and Sales are independent measures that often flow forward; stop deriving one from the other. |

---

## 2. SETTINGS, TOP BAR, APPEARANCE (do together as one pass)

### 2.1 Top-right toolbar cleanup
Keep ONLY: **Profile (shows user's name)**, **Undo** (generic undo-arrow look), **Bell**, **Ticket**, **Full screen**.
Moved:
- Multi-persona switch icon → stays, **left of the user/profile icon** (multi-persona users only).
- Pipeline grid customization button → **left of the GROUP dropdown** in the pipeline toolbar.
- Dark/light toggle → inside **Pipeline appearance** settings (light and dark are the defaults).
- Power / log-out → inside **Profile / account-information** section (not the top row).

### 2.2 Top bar color
When not in full screen the top bar is always navy. Make it **match the user's chosen header-column color**. (Previously noted as possibly already done — verify.)

### 2.3 Profile menu (click your name) → sections
a. **Profile information:** Name, Title, Email (read-only), Password reset (+ log-out lives here).
b. **Quota / Goals:** Bucket selection, Annual quota, Monthly quota, Annual goal, Monthly goal; also shows **RC TI %** and **per-unit payouts**.
c. **Pipeline appearance:**
   - Themes: Lightning, Fall, Cruise ship, Money, plus general color themes. Light/dark are the defaults.
   - **"Customize pipe" → Designer mode:** main pipeline becomes non-editable/non-movable; a **draggable color wheel** floats on screen; user clicks any element to recolor it; used colors are saved as a palette for quick-apply; everything is editable. Save as a **named style; max 3 saved styles.** (Existing customization is "not quite what was hoped" — replace/rework it.)
d. **Upload / export pipeline:** step-by-step guide. Before downloading the **starter CSV template**, user chooses columns and order. Default left→right: **Follow up date, Account name, Contact, Stage, Engagement, State, EE, Bundle, Frequency, Dis %, Input date, Notes** (add / remove / reorder allowed).

### 2.4 Admin Settings (org-wide, "think BU not team-by-team")
- All admin settings apply to **ALL teams**, organization-wide.
- Admin Settings is a **collapsible/expandable list** of sections: **Price book, Mandates, Stage names, Quotas**, and room to add more.
- Add an **admin view of the Resources tab**.
- When an admin makes an update: auto-generate a **synopsis of what changed**, post it to the **discussion board as an announcement**, and **alert DMs via the notification bell**.

---

## 3. MANDATES → ADMIN STATE MAP

- Account "Mandates" are no longer per-deal; they're sourced from the **admin-defined state + employee-size config**. Keep the old per-deal field in data, but drive the account's mandate display from the admin map.
- **Admin editor = simple spreadsheet**: one row per state; columns = WC, RS, PEO, COMP. Cell values: `NO` or a number (employee threshold: 1, 2, 3…). PEO cell: `NO`, number, and a **TS / TS select** selector. COMP cell: `NO` or number. (Admin can extend with more columns later.)
- **State-specific info resource = interactive US map:**
  - Realistic, recognizable US map (like a standard state-outline map — NOT a blocky tile map). Inline SVG with per-state paths is fine.
  - **Search box** to find a state.
  - Hover → state grows slightly; click → shows the admin-provided info for that state.
  - Lens selector (extensible): **ALL** (everything at once), **WC** (hard-hat icon + EE threshold), **RS** (piggy-bank icon + threshold), **PEO** (“TS” icon + threshold), **COMP** (“COMP” icon + threshold).
  - Colors: WC/RS → **RED = can't write**, **YELLOW = state doesn't mandate**. PEO → RED = can't, YELLOW = TS select. COMP → RED = can't. **Red wins on overlap.**
  - PEO and COMP are not mandatory but need people balancing them out.
- Seed placeholder data for all 50 states + DC.

---

## 4. TODAY TAB (first pass; will iterate)

Show:
- a. Starts this month (count)
- b. Forecasted number (locked forecast total)
- c. Props this week / Sales this week
- d. Follow-ups today (count) + Overdue (count)
- e. Task list
- f. Incomplete accounts
- g. Written-out, more prominent notifications

Individual stats (all driven by a **date-range control: presets + custom picker**): average deal size; **BP attachment %** (share of deals with a BP product); props #; **Started & Sales month-over-month / week-over-week graph**; **progress graphs to year goal and to quota**.

---

## 5. PIPELINE (DM VIEW) FEATURES

| ID | Item |
|---|---|
| P-1 | **Forecast filter:** clicking the locked forecast number currently opens a card of forecasted accounts — wrong. Clicking the forecast requirements should **filter the grid** to forecasted accounts. |
| P-2 | **"Upside" engagement option.** When selected, account is calculated as Upside; must still meet forecast requirements but lands in a separate Upside category. |
| P-3 | **"Started" filter** so DMs can see what started this month. |
| P-4 | **Compound filters:** "Add filter" must support **AND** and **OR** requirements (critical). |
| P-5 | **Auto-filter chips are reorderable** by press-and-hold + drag. |
| P-6 | **Excel-style multi-cell selection:** rectangular X/Y range selection (not whole rows). **Right-click menu:** Copy · Export data · **Mass follow-up change** (only when selection is in the follow-up column). |
| P-7 | **Remove emojis** from the pipeline; use custom **mini icons** (same set serves the Resources app). |
| P-8 | **Account card:** show **per-processing fee** right next to the ARC at the top (integral to client conversations). |

---

## 6. SE ACCOUNT GRID (left side of the Accounts tab)

- S-1 Weekly sales breakdown **for each week, for each DM**.
- S-2 **Monthly total for the whole team**, broken down by week.
- S-3 **Monthly totals per DM**.
- S-4 Ability to **remove a deal from Sales/Starts without affecting the DM pipeline** once it's on the chart.
- (See BUG-4/5/6/7 above for SE grid correctness.)

---

## 7. COLLABORATION SPLIT (B1-9) — needs one accuracy review

Flow: (1) original DM **invites** another DM and sets the **offer**: invitee's **RC %** and **whether invitee receives the unit**. (2) invitee **accepts or denies**. Only the **original DM** can edit the percentages (invitee cannot). Percentages must **always total 100%** — currently 25/75 etc. gets attributed to both DMs wrongly.
Status per last report: **in progress** — viewer-aware crediting (owner = 100 − Σ collaborator %, collaborator = their %), owner-only editing, unit assignment, team-leaderboard crediting; accuracy-review agent pending. Verify the 100% invariant with 25/75, 60/40, 3-way splits, and unit-to-invitee on/off.

---

## 8. TICKETS & DISCUSSION BOARD

- T-1 **Admin can export tickets** in a format easy to paste/upload into a Claude conversation (e.g. Markdown/JSON with id, author, date, status, title, body, comments).
- T-2 **Discussion board shows ALL requests** (currently only new). Add **upvotes, comments, delete post, flag post**.
- T-3 Ticket upload: **public to discussion board by default**, with a **toggle for private message to admin only**.

---

## 9. NEW APP: "Email Quote Form"

Personalized email link tied to each user's account. Each user builds a survey their clients open, fill out, and submit → it **lands in that user's pipeline**.
Default fields: First & last name · Business name · Employee count · Pay frequency · State · Email · Interest in ADP · Schedule a call time with the rep.
**DM can customize** fields (retrieve any info). Fields should map cleanly to pipeline columns.
On submit: **bell notification + popup card** ("you received a form fill-out"). Until acknowledged, the pipeline row is **flagged with follow-up date/time** in the follow-up column so it's obvious.

---

## 10. TAKE-THE-TOUR APP (DM view only; spec first, then user approves)

Entry screen with two options: **(1) Take the whole tour**, **(2) Pick a specific function** (section list to be written up). Whole tour = guided demo of the pipeline: **arrows pointing at parts of the UI + brief captions, Next button, X (top-right) to exit.** Deliver a **written spec** for user approval/tweaks before building.

---

## 11. Suggested order
1. Audit table (cheap grep pass) → user confirms.
2. Bugs (BUG-1…7).
3. Top bar + Settings revamp (2.1–2.4) in one pass.
4. State map + admin spreadsheet (3).
5. Today tab (4).
6. Pipeline features (5), SE grid (6), tickets/discussion (8).
7. Collaboration review (7), Email Quote Form (9), Tour spec (10).
