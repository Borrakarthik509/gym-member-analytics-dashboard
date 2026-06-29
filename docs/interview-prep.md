# 🎤 Interview Preparation — Technical Q&A

[← Back to README](../README.md)

> 16 prepared answers to the hiring questions a BI / Data Analyst interviewer is most likely to ask about this specific dashboard. Each answer references actual implementation details from the project.

---

## Table of Contents

- [Section 1: Data Modelling](#section-1-data-modelling) — Q1–Q4
- [Section 2: DAX](#section-2-dax) — Q5–Q7
- [Section 3: Visualisation & UX](#section-3-visualisation--ux) — Q8–Q11
- [Section 4: Interactivity & Performance](#section-4-interactivity--performance) — Q12–Q14
- [Section 5: Deployment & Governance](#section-5-deployment--governance) — Q15–Q16
- [Quick-Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Section 1: Data Modelling

### Q1. Walk me through your data model. Why did you choose this structure?

> The model follows a **star schema**: one fact table (`Gym Dataset`) at the centre, surrounded by seven dimension tables (`Mood`, `Gender`, `Steps bucket`, `Steps Category`, `Group lesson`, `Exercise`, `Birth data`) and one isolated measure table (`Measure_table`). I chose this structure because star schemas minimise the number of joins Power BI's engine has to resolve at query time, which keeps visual rendering fast — especially important when all three slicers are applied simultaneously and the engine must cross-filter six visuals at once.

---

### Q2. Why did you create a separate `Measure_table` instead of writing measures on the fact table?

> Storing measures in a disconnected table is a widely adopted Power BI best practice. It has three concrete benefits:
>
> 1. The fact table's field list stays clean (only raw columns, no mixed measures)
> 2. All business logic is in one findable location when you need to edit or audit it
> 3. The measure table can be hidden from report view while still being accessible to visuals
>
> If this model were to scale to multiple fact tables, the measure island also allows a measure to reference multiple tables without being "owned" by one of them.

---

### Q3. How do the `Links` columns in your dimension tables work, and why did you use them?

> Each dimension table that powers an advanced slicer has a `Links` (or `Link`) column that is used as the slicer's binding field rather than the raw data column. This **decouples the filter display from the filter value**. It means I can write custom button labels in the slicer, control the sort order independently, and style the buttons — none of which is possible when you bind a slicer directly to a fact column. The dimension table still joins to the fact table on the actual data column, so filtering still propagates correctly.

---

### Q4. The `Steps bucket` and `Steps Category` appear to be two separate dimension tables. Why not just one?

> They serve different analytical levels:
>
> - **`Steps bucket`** — provides granular numeric ranges (e.g., `"0–5k steps"`) used on the x-axis of the line chart to show the calories-vs-steps relationship at a fine level
> - **`Steps Category`** — provides a broader activity tier label (e.g., `"Sedentary"`, `"Active"`) used as a slicer for filtering the whole report
>
> Keeping them separate allows each to have its own sort order and display logic without one contaminating the other.

---

## Section 2: DAX

### Q5. What does `avg_visit` actually measure, and how does it respond to slicer selections?

> `avg_visit` is `AVERAGE('Gym Dataset'[visits])`. When a slicer selection is made — say, `"Female"` in the gender slicer — Power BI applies that filter to the `Gym Dataset` table through the relationship between `Gender` and `Gym Dataset`. The `AVERAGE` function then evaluates only over the rows that survive the filter, so the card automatically shows the average visit count for female members specifically. No explicit `CALCULATE` or `FILTER` is needed because the slicer context propagates through the relationship.

---

### Q6. What is the difference between `AVERAGE` and `AVERAGEX` in DAX, and when would you use each?

> - **`AVERAGE`** — shorthand that works directly on a single numeric column
> - **`AVERAGEX`** — an iterator that evaluates an expression row by row, then averages the results
>
> For this dashboard, `AVERAGE(visits)` is sufficient because the calculation is straightforward. I would use `AVERAGEX` if I needed to compute something like *average revenue per member*, where revenue requires multiplying two columns:
>
> ```dax
> AVERAGEX('Gym Dataset', 'Gym Dataset'[price] * 'Gym Dataset'[quantity])
> ```

---

### Q7. If you wanted to show how the current month's average session time compares to the previous month, what DAX pattern would you use?

> I would use `CALCULATE` with `DATEADD` from the time intelligence functions, which requires a proper date table:
>
> ```dax
> Avg Time PM =
> CALCULATE(
>     [avg_time],
>     DATEADD('Date'[Date], -1, MONTH)
> )
> ```
>
> A date table does not currently exist in this model — `Birth data` provides birthday information, not a time axis. Adding a proper `Date` table marked as a date table in Power BI would be the prerequisite for this enhancement.

---

## Section 3: Visualisation & UX

### Q8. Why did you choose a dark canvas background for this dashboard?

> Three reasons:
>
> 1. **Contrast** — dark backgrounds increase the perceptual contrast of coloured data series. The primary accent blue (`#118DFF`) reads significantly more clearly against `#1D1E23` than against white
> 2. **Environment** — gym environments often feature wall-mounted displays in low-light settings; a dark background reduces screen glare and eye fatigue
> 3. **Brand alignment** — it aligns the visual identity with modern fitness brand aesthetics, making it feel purpose-built rather than generic

---

### Q9. You used named visual groups with ScaleMode. What problem does that solve?

> Without groups, if the canvas is displayed on a screen with a different resolution, Power BI scales each visual independently, which can break the relative positioning of overlapping elements — for example, a KPI icon shifting relative to its card background. By wrapping each card and its icon into a `ScaleMode` visual group, Power BI scales all child visuals as a single unit, preserving spatial relationships regardless of display size. It also simplifies the Selection Pane from ~60 individual entries to ~15 logical groups.

---

### Q10. The combo chart uses a dual axis. What is the risk and how did you mitigate it?

> Dual-axis charts risk **misleading the reader** by implying a correlation between two metrics that share no real relationship, simply because the Y-axis scales make them appear aligned. I mitigated this by:
>
> 1. Using **different visual encodings** — columns for one metric, a line for the other — so the visual form signals they are distinct measures
> 2. Keeping **axis labels visible** on both sides so a careful reader can verify the scales independently
> 3. In a production context, I would add a clear chart title naming both metrics to eliminate ambiguity

---

### Q11. The mood bar chart uses `Mood Display` for labels rather than the raw `mood` column. Why?

> Raw mood values are likely stored as numeric codes or short text identifiers that are meaningful to the system but not to a gym manager reading the dashboard. `Mood Display` contains human-readable labels. This separation also allows the **sort order to be set explicitly** — e.g., ordering from `"Very Happy"` to `"Very Tired"` on a logical scale rather than alphabetically — without modifying the fact table or the join key.

---

## Section 4: Interactivity & Performance

### Q12. Cross-filtering is enabled globally. When would you disable it for specific visuals?

> If a visual serves as a **constant reference** — for example, a "total members" headline card that should always show the all-up figure regardless of what other visuals are clicked — you would disable cross-filtering via the **Edit Interactions** panel. In this dashboard, all visuals respond to cross-filter because each is intended to update contextually. If I added a target or benchmark card, I would exempt it from interactions so it always shows the absolute goal.

---

### Q13. What is the `exportDataMode: AllowSummarized` setting?

> This controls what data users can export when they right-click → **Export data**. `AllowSummarized` means they can export the aggregated table the visual shows (e.g., trainer name + member count), but **not the raw underlying rows**. This is a data governance decision — it prevents downloading the complete member dataset via the report UI while still allowing summary extraction for further analysis.

---

### Q14. What would you do to improve performance if this model grew to 500,000 rows?

> Four actions in priority order:
>
> 1. **Import mode** — ensure the data model uses Import so queries run against the in-memory VertiPaq engine rather than DirectQuery
> 2. **Remove unused columns** — every column in Import mode is compressed and held in memory
> 3. **Audit relationships** — all dimension-to-fact joins should be many-to-one with single-direction filtering; disable bidirectional unless explicitly needed
> 4. **Optimise measures** — ensure `avg_visit`, `avg_age`, and `avg_time` use column-level aggregations rather than row-level iterators

---

## Section 5: Deployment & Governance

### Q15. This report is a `.pbix` file on GitHub. What are the limitations and how would you improve it for production?

> **Limitations:**
>
> | Issue | Impact |
> |-------|--------|
> | Manual refresh only | Data goes stale between opens |
> | No access control | Anyone with the file sees all member data |
> | Git-based versioning | No built-in Power BI dataset versioning |
>
> **Production upgrade path:**
>
> 1. Publish the dataset to **Power BI Service**
> 2. Configure **scheduled refresh** (triggered when source is updated in SharePoint/OneDrive)
> 3. Implement **Row-Level Security** to restrict trainers to their own members
> 4. Distribute via a **Power BI App** — read-only web view, no `.pbix` exposure

---

### Q16. If the gym expanded to five branches, how would you implement Row-Level Security?

> 1. Add a `branch_id` column to the fact table and a `Branch` dimension table
> 2. In Power BI Desktop → **Modelling → Manage Roles**, create a role per branch:
>
> ```dax
> [branch_id] = "BRANCH_001"
> ```
>
> 3. In Power BI Service, assign each branch manager's email to their corresponding role
> 4. When they access the report, the RLS filter silently applies before any visual query runs — the report URL and layout are identical across all managers

---

## Quick-Reference Cheat Sheet

| Topic | Key Point |
|-------|-----------|
| Data model | Star schema — 1 fact, 7 dims, 1 measure island |
| Measure storage | Disconnected `Measure_table` — best practice |
| Slicer binding | `Links` columns decouple display from filter value |
| Cross-filtering | Enabled globally via `defaultDrillFilterOtherVisuals` |
| Canvas size | 1280 × 720, dark bg `#1D1E23`, Fit to Page |
| Visual groups | ScaleMode — preserves layout at any display resolution |
| Export control | `AllowSummarized` — aggregates only, no raw row export |
| DAX level | AVERAGE functions; no CALCULATE / time intelligence yet |
| RLS | Not implemented — single-owner scope; roadmap item |
| Deployment | `.pbix` on GitHub; upgrade path → Power BI Service + App |
