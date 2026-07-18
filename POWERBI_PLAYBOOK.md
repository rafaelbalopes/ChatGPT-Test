# Power BI Rebuild Playbook — KPMGI Compensation Dashboard

A click-by-click guide to rebuilding the compensation dashboard in Power BI Desktop,
with every trap we hit already corrected. Written for a non-technical analyst.

**Golden rule that got missed the first time:** create your report **pages first**, then
build visuals onto the right page. Don't cram everything onto Page 1.

**Confidentiality:** with real pay data, keep the `.pbix` local — don't Publish to the
Service or hit Share until it's meant for an audience (then use Row-Level Security).

---

## Part 0 — Set up your pages (do this first!)

Bottom-left of the window, next to the **Page 1** tab, click **+** four times. Double-click
each tab to rename:

- **Composition** · **Equity** · **Pay Structure** · **Gender** *(delete if no gender data)* · **Salary Ranges**

---

## Part 1 — Get the data in

1. **Home → Get data → Text/CSV** (or **Excel workbook**). Pick your export → **Open**.
   - CSV goes straight to a preview. Excel first shows a **Navigator** — tick the data sheet.
2. Click **Transform Data** — **not** "Load." (We clean first.)

✅ The **Power Query Editor** opens.

---

## Part 2 — Clean it (Power Query, no code)

1. **Headers:** if columns read "Column1, Column2…", click **Use First Row as Headers**
   (Home). CSV usually does this automatically.
2. **Rename the query:** right-click it (left panel) → **Rename** → `Employees`.
3. **ID columns → Text:** click the type icon (`ABC`/`123`) on **GPID** and **Country
   Employee ID** → **Text**. *(This clears the "12 errors" — number-typed IDs fail to
   convert and lose leading zeros.)* If asked, **Replace current** step.
4. **Currency normalize:** select **Local Currency** header → **Transform → Extract →
   First Characters → 3**. ("CAD - Canadian Dollar" → "CAD".)
5. **Null salaries:** decide deliberately. To drop them: dropdown on **Local FTE Base Pay**
   → untick **(null)** and **(blank)**. Rename the step (right-click → Rename →
   `Removed null-salary employees`). *We reviewed ours and chose to drop them.*
6. **Level Rank column:** **Add Column → Conditional Column**, name `Level Rank`, one clause
   per level (`If Global Level equals <name> then <n>`): Partner→1, Senior Leader→2,
   Director…(3)→3, Senior Manager…(4)→4, Manager…(5)→5, Senior Team Member…(6)→6,
   Team member…(7)→7, **Else 99**. Copy level names exactly from the column's filter list.
   Then set the new column's type to **Whole Number**.
7. **Home → Close & Apply.**

✅ The Data pane on the right lists **Employees** with all fields.

---

## Part 3 — Composition page visuals

Build these on the **Composition** page.

- **Headcount card:** **Card** visual → drag **GPID** into the **Values** well (NOT
  Categories — Categories makes one mini-card per person) → set aggregation to **Count**.
  If it shows "2K", Format visual (paintbrush) → **Callout value → Display units → None**.
- **Headcount by level:** **Clustered bar chart** → **Global Level** to Y-axis, **GPID** to
  X-axis.
- **Median pay by level:** another bar chart → **Global Level** to Y, **Local FTE Base Pay**
  to X → click the pill's dropdown → **Median**.
- **Slicers:** **Slicer** visual × 3 → **Employee Status**, **Country**, **Global Level**.

**Sort levels correctly (do once, then per chart):**
1. Data pane → click the **Global Level** field → **Column tools → Sort by column →
   Level Rank**.
2. On each level chart: **⋯ → Sort axis → Global Level**, then **⋯ → Sort axis → Sort
   ascending**.

**Save:** File → Save → `Comp Dashboard.pbix` (local).

---

## Part 4 — Default view (Filters on all pages)

In the **Filters pane**, under **Filters on all pages**:
1. Drag **Global Level** → tick every level **except Partner**.
2. Drag **KPMGI FTE Percent** → **Advanced filtering** → *is greater than or equal to* **51**
   → **Apply filter**.

✅ **Sanity check:** open the same file in the HTML dashboard with default filters; the
Power BI headcount (Status = Active) must match.

---

## Part 5 — Measures (DAX)

Ritual for each: right-click **Employees → New measure**, then paste the whole line
(the name is the text before `=`). Format via the **Measure tools** ribbon.

```DAX
Headcount = COUNTROWS(Employees)
```
```DAX
FTE-Adjusted HC = SUMX(Employees, Employees[KPMGI FTE Percent] / 100)
```
```DAX
Total Base = SUMX(Employees, Employees[Local FTE Base Pay] * Employees[KPMGI FTE Percent] / 100)
```
```DAX
Bonus % of Base = MEDIANX(Employees, DIVIDE(Employees[Local FTE Bonus], Employees[Local FTE Base Pay]))
```

- Format counts with the comma button; `Bonus % of Base` with the **%** button.
- Put `Bonus % of Base` on the **Pay Structure** page as a bar chart by Global Level — it
  should form a staircase (highest at Senior Leader). A broken step is a real finding.
- Total Base sums local currency; only meaningful for a single-currency population. Add an
  FX table before mixing countries.

---

## Part 6 — Equity page: Country × Level heatmap

1. **Matrix** visual → **Country** to Rows, **Global Level** to Columns, **Local FTE Base
   Pay** to Values → set the value to **Median**.
2. **Heat colour:** Format visual → **Cell elements → Background color → On → fx →** color
   scale (light→dark).
3. **Limit countries:** Filters pane (this visual) → **Country → Top N → Top 10 →
   By value: `Headcount` measure** (NOT the GPID text column) → Apply.
4. Row order is alphabetical by default; Top-N already bounds it to your biggest countries,
   so order is cosmetic. Optional: **⋯ → Sort by → Median of Local FTE Base Pay → Sort
   descending**.

**Known limits (don't fight these):** a matrix with a Columns field can't show a single
count column beside the row label, and its sort only offers fields shown in the visual.

---

## Part 7 — Pending / later

- **Level & person-level inversion, compression** — DAX measures on the Equity page; the
  person-level one needs a duplicated `Managers` table related to `Employees[PM User ID]`.
- **Salary ranges / compa-ratio / penetration** — a grouped `Bands` query (Country+Level →
  P5/median/P95 in USD), related back to Employees.
- **Gender gap & pay quartiles** — only when a real Gender column exists. Do not infer.
- **Sync slicers** — View → Sync slicers, so one filter bar drives every page.

---

*Keep this in sync with the gotchas section of `CLAUDE.md`.*
