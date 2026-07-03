# CLAUDE.md — repo guidance for Claude sessions

This repo holds a self-contained compensation dashboard (`compensation-dashboard.html`)
and documentation for rebuilding the same analysis in Power BI (`POWERBI_PLAYBOOK.md`).

## compensation-dashboard.html

- Single self-contained file: all HTML/CSS/JS inline, data embedded as `const DATA=…`,
  **no external dependencies** (no CDN). It must stay openable from `file://` and offline.
- Two placeholder columns are intentional and **flagged in-UI**: `gender` (inferred from
  first name) and salary `ranges` (mock P5/median/P95 bands). Never present them as real.
- Range math (penetration, compa-ratio, band position, remediation) is computed **in USD**
  internally and converted only for display — do not reintroduce per-currency band pooling.
- Escape any user-supplied string before it enters `innerHTML` (uploaded files can contain
  markup) — use the existing `esc()` helper.
- To regenerate: edit `scratchpad/template.html`, inject `data.json` in place of
  `/*DATA_PLACEHOLDER*/`, and strip the wrapper tags for the Artifact copy.
- Verify changes by driving the file in headless Chromium (Playwright at
  `/opt/node22/lib/node_modules`) — check KPIs, tab switches, and **zero console errors** —
  before committing. Cross-check numbers against Python on the raw data.

## Power BI work — gotchas to state UP FRONT (learned the hard way)

When guiding a non-technical user through Power BI, give beginner-precise clicks and
volunteer these BEFORE they hit them, in this order:

1. **Create the report pages first.** Don't let visuals pile onto Page 1 — set up one page
   per section (Composition / Equity / Pay Structure / …) at the start.
2. **Get Data → Transform Data, not Load** (clean in Power Query first). CSV skips the
   Excel "Navigator" sheet-picker; otherwise identical.
3. **ID-looking columns must be type Text** (GPID, employee IDs). Number-typed IDs throw
   conversion errors and eat leading zeros. Fix at the "Changed Type" step.
4. **Card visual has two wells: Categories and Values.** A field dropped in *Categories*
   makes one mini-card per value ("row per employee"); it must go in **Values**, then set
   the aggregation to **Count**.
5. **Card display units default to "Auto"** and abbreviate (1,742 → "2K"). Set Format →
   Callout value → Display units → **None**.
6. **A measure's name is the text before `=`** in the formula (e.g. `Headcount = …`).
   Don't have the user name it separately.
7. **Level ordering** needs a `Level Rank` column (Conditional Column), then
   field → **Sort by column → Level Rank**, then per-visual **⋯ → Sort axis**.
8. **Matrix "Top N → By value" needs a measure** (e.g. `Headcount`), not the text GPID
   column — a text field only offers First/Last/Count and misleads.
9. **A matrix with a Columns field can't show a single un-split count column** next to the
   row label — every value repeats under each column group. Don't promise it.
10. **Matrix sort only offers fields the visual displays.** To sort rows by headcount you'd
    need headcount in the visual; otherwise sort by an available measure or accept
    alphabetical (Top-N already bounds membership, so order is cosmetic).
11. **Basic filters show a live count next to each value** but it can't be dragged onto the
    canvas — to display a count you use a measure in a visual.
12. **Slicers filter only their own page by default.** Use **View → Sync slicers** (or copy
    the slicer onto each page) to get a global filter bar.
13. **Normalize at ingest**, mirroring the HTML app: currency `"CAD - Canadian Dollar"` →
    first 3 chars; status `"A - Active"` → `Active`; keep FTE `0` as `0` (not null→100);
    decide null-salary rows deliberately (the user chose to drop them).
14. **Default view** = Active, exclude Partner, FTE ≥ 51 — set via *Filters on all pages*
    (FTE via Advanced filtering ≥ 51). Sanity-check headcount against the HTML dashboard on
    the same file + same filters (they must match).

Full step-by-step (for the human) lives in `POWERBI_PLAYBOOK.md` — keep the two in sync.
