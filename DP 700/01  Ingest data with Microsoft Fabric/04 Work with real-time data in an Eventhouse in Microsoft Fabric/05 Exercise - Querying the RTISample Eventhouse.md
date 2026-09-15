
# Exercise Notes — Querying the RTISample Eventhouse (Bikestream)

Hands-on exercise notes from actually running the "Explore Real-Time Intelligence Sample" lab. These are the facts and gotchas worth keeping, not a step-by-step replay — see [02 Get Started with an Eventhouse](02%20Get%20Started%20with%20an%20Eventhouse.md) and [03 Use KQL Effectively](03%20Use%20KQL%20Effectively.md) for the conceptual background.

---

## 1. Fastest Path to Sample Data: "Explore Real-Time Intelligence Sample"

* A workspace needs a **paid or trial Fabric capacity** before an Eventhouse can be created in it — this isn't optional, unlike some other Fabric items.
* On the Real-Time Intelligence home page, the **Explore Real-Time Intelligence Sample** tile is a one-click shortcut that provisions a fully populated Eventhouse (named **RTISample**) for you — no Eventstream, no manual ingestion setup.
* This is notably different from every ingestion path described in [02 Get Started with an Eventhouse §2](02%20Get%20Started%20with%20an%20Eventhouse.md) (direct ingestion, shortcuts, OneLake availability) — it's a demo-data shortcut that skips all of that.
* Creating the Eventhouse creates a **same-named KQL database** automatically (confirms the fact from note 02 §1) — this instance came with a `Bikestream` table already populated (NYC-style bike-share station data: `Street`, `No_Bikes`, `No_Empty_Docks`, `Neighbourhood`, etc.).

---

## 2. KQL Facts Confirmed by Running Queries

* `take N` behaves like `SELECT TOP N *` — cheap sampling, no guaranteed ordering.
* `project` can **rename a column inline** using bracket syntax when the new name has spaces or special characters:
  ```kql
  Bikestream
  | project Street, ["Number of Empty Docks"] = No_Empty_Docks
  ```
  Plain SQL-style `AS` doesn't exist in KQL for this — the `["Name"] = expr` form is the only way to alias.
* `sort by` and `order by` are genuinely **interchangeable synonyms** in KQL — confirmed both produce identical results on the same query. SQL only has `ORDER BY`, so this is a KQL-specific redundancy, not a trick.
* `where` uses `==` for equality (not `=`), and is **case-sensitive** on string comparisons (`"Chelsea"` won't match `"chelsea"`) — consistent with note 02's point that KQL is case-sensitive everywhere, including string values, not just identifiers.
* The `case()` + `isempty()` / `isnull()` pattern is the standard KQL idiom for bucketing missing/blank grouping keys into a labeled catch-all (e.g. `"Unidentified"`) instead of letting them render as blank rows in results.

### Subtlety worth remembering: where the `case()` runs relative to `summarize`

In the KQL version used in this exercise, the pipeline is:

```kql
Bikestream
| summarize ["Total Number of Bikes"] = sum(No_Bikes) by Neighbourhood
| project Neighbourhood = case(isempty(Neighbourhood) or isnull(Neighbourhood), "Unidentified", Neighbourhood), ["Total Number of Bikes"]
```

The `summarize ... by Neighbourhood` groups on the **raw** column *before* the `case()` relabeling happens in `project`. That means if the source data had **both** `NULL` and `""` (empty string) as distinct values for `Neighbourhood`, this query would produce **two separate "Unidentified" rows** (one per distinct raw value being summed independently), not one merged row — the relabeling happens too late to affect the grouping.

The T-SQL version avoids this because it puts the `CASE` expression directly inside `GROUP BY`, so the grouping itself happens on the normalized label:

```sql
GROUP BY CASE
    WHEN Neighbourhood IS NULL OR Neighbourhood = '' THEN 'Unidentified'
    ELSE Neighbourhood
END
```

**Takeaway:** if you need null/empty values *merged* into one group in KQL (not just relabeled after grouping), do the `case()` normalization **before** `summarize`, e.g. as an `extend` step first — don't rely on a post-`summarize` `project` to merge groups.

---

## 3. T-SQL Endpoint — What It Actually Is

* Every KQL database exposes a **T-SQL endpoint** that emulates SQL Server well enough for tools that only speak T-SQL (not KQL) to query the same data.
* It is **read-only in the schema/data-mutation sense**: no `CREATE`/`ALTER`/`DROP TABLE`, no `INSERT`/`UPDATE`/`DELETE`. You can only `SELECT`.
* It doesn't support the full T-SQL surface — only a compatible subset, plus common aggregates (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) that map cleanly onto KQL's own aggregation functions.
* Microsoft's own recommendation (per the exercise): **prefer KQL as the primary language** — the T-SQL endpoint exists for interoperability with SQL-only tooling, not as a first-class equal to KQL. KQL has more capability and better performance against these tables.
* Practical corollary: the same `case()`/null-bucketing logic that's a single `project` step in KQL requires repeating the full `CASE WHEN` expression in **both** the `SELECT` list and the `GROUP BY` clause in T-SQL — SQL has no way to reference a `SELECT`-list alias inside `GROUP BY`.
* T-SQL's `HAVING` is the mechanism for filtering on a `GROUP BY`-derived/aliased column (e.g. filtering to `Neighbourhood = 'Chelsea'` after grouping) — `WHERE` can't reference the post-aggregation alias.

---

## 4. Cleanup Reality Check

* An Eventhouse (and its KQL database) lives inside — and bills against — the workspace's Fabric capacity. There's no separate "delete eventhouse" cleanup step emphasized in the exercise; the recommended cleanup is **deleting the whole workspace** (Workspace settings → General → Remove this workspace).
* Worth remembering for trial capacities specifically: leaving an unused RTISample workspace around is the thing that quietly burns trial capacity time, not just the Eventhouse item itself.

---

## 5. Quick Reference: KQL vs T-SQL for the Same Task

| Task | KQL | T-SQL (endpoint) |
| --- | --- | --- |
| Sample rows | `\| take 100` | `SELECT TOP 100 *` |
| Rename column | `project ["New Name"] = col` | `col AS [New Name]` |
| Aggregate | `summarize x = sum(col) by g` | `SELECT g, SUM(col) AS x ... GROUP BY g` |
| Null/empty bucketing | `case(isempty(g) or isnull(g), "Unidentified", g)` | `CASE WHEN g IS NULL OR g = '' THEN 'Unidentified' ELSE g END` |
| Sort | `sort by col asc` **or** `order by col asc` (same thing) | `ORDER BY col ASC` (only option) |
| Filter post-aggregation | `where` *before* `summarize`, or filter the `project`ed result | `HAVING` (after `GROUP BY`) |
| Schema/data mutation | Full DDL/DML support natively | **Not supported** — read-only `SELECT` |

---
