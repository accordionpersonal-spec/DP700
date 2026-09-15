# Plan a Medallion Architecture in Fabric
*DP-700 · Unit: Plan a medallion architecture in Fabric (need-driven reference)*

Three decisions shape the whole implementation:
1. **Where do the layers live?**
2. **What moves data through each layer?**
3. **How do consumers reach gold?**

---

## Decision 1 — Where do the layers live?

**The need:** different teams and rules need different *walls* between
layers. More walls = more safety, but also more things to manage.

| Option | The need it serves |
|---|---|
| **One lakehouse, three schemas** (bronze/silver/gold) | Small team, early project — keep it simple. Schemas group tables by layer, make them easy to find, and still allow different permissions per layer. Schemas are **on by default** in a new Fabric lakehouse. |
| **Separate lakehouses per layer** | Clearer separation — permissions set at the lakehouse level, harder to accidentally cross layers. |
| **Separate workspaces per layer** | Strongest isolation — for **regulatory / compliance** situations (think bank, healthcare) where "engineers can't even see prod gold" is a legal requirement, not a preference. |

Ladder to remember: **schemas → lakehouses → workspaces**
(each step = more isolation, more overhead).

---

## Decision 2 — Moving data through the layers

### Into bronze: shortcut vs. load

**The need:** a lot of source data already sits in cloud storage.
The old reflex — build a pipeline to *copy* it in — means duplicate
storage costs, sync lag, and one more pipeline to babysit and debug.

- **Data already in cloud storage** (OneLake, ADLS Gen2, Amazon S3,
  Google Cloud Storage) → **OneLake shortcut**: a *pointer*, not a copy.
  Like a desktop shortcut — the file lives where it lives, you see it in
  your lakehouse. Always in sync, zero ingestion code, zero duplication.
- **Everything else** (databases, APIs, on-prem files) → load it with
  **pipelines, dataflows, or notebooks**.

Exam trigger: *"data is already in S3/ADLS, minimize data movement"*
→ shortcut.

### Through silver: three tools, one question — "how heavy is the job?"

| Tool | When it's the right size |
|---|---|
| **Dataflow Gen2** | Low-code, visual. Simple cleanups: filter rows, rename columns, change types. For "I don't want to write code for this." |
| **Notebook** (Python/SQL) | The power tool: huge datasets, complex joins, custom calculations, calling APIs — anything a dataflow can't express. |
| **Materialized lake view** | "I can describe the clean table in SQL — Fabric, build it and *keep it fresh* for me." |

### The star of this unit: materialized lake views

**The need, step by step:**
- A **regular view** is just a saved query — it *reruns* every time
  anyone queries it. Heavy transform + many users = paying the same
  compute over and over, slow experience.
- A **scheduled rebuild** (notebook/pipeline writing a table) fixes
  that, but now *you* own the scheduling — and naive rebuilds reprocess
  the whole table even when 1% of rows changed.
- A **materialized lake view** saves the query result as a **real
  table**, and when new data lands in bronze, Fabric updates **only the
  rows that changed**. No schedule, no trigger.

**Why that's even possible:** every Fabric table is stored in **Delta**
format, and Delta logs every change. Fabric reads that log and patches
silver instead of rebuilding it.

Syntax shape (define the *result*, Fabric handles the rest):

    CREATE MATERIALIZED LAKE VIEW silver.orders
    AS
    SELECT
        order_id,
        LOWER(TRIM(customer_email))  AS customer_email,
        CAST(placed_at AS DATE)      AS order_date,
        quantity * unit_price        AS revenue
    FROM bronze.orders
    WHERE order_id IS NOT NULL

**The tradeoff:** the logic must fit in SQL. Need Python, API calls,
fancy branching? → notebook.

### Into gold: shape follows audience

**The need:** different consumers ask differently *shaped* questions —
one gold table can't fit everyone.
- BI / reporting team → **star schema** (facts + dimensions).
- Data science team → **flat, wide table** for feature engineering.
- Finance → **pre-aggregated summaries**.

Same silver underneath, **multiple gold layers on top** — that's normal,
not a hack. Built with the same three tools, coordinated by pipelines.

---

## Decision 3 — How consumers reach gold

| Door | Who it's for |
|---|---|
| **SQL analytics endpoint** | SQL-fluent people/tools — direct T-SQL queries over the Delta tables. |
| **Power BI semantic model** | Business users — friendly names + predefined measures, so "revenue" is calculated **one way, everywhere**. |
| **Fabric Data Warehouse as gold** | Teams that live in SQL end-to-end can use a warehouse instead of a lakehouse for the gold layer. |

---

## Exam trigger phrases → answers

| Question says… | Think… |
|---|---|
| regulatory / compliance isolation | separate **workspaces** |
| simplest to manage, small team | one lakehouse + **schemas** |
| data already in S3/ADLS, avoid copying | **shortcut** |
| low-code / no-code transformation | **Dataflow Gen2** |
| Python, APIs, complex logic, very large data | **Notebook** |
| SQL transform that stays current automatically | **Materialized lake view** |
| reruns every query vs. saved + auto-updated | regular view vs. **MLV** |
| business-friendly measures for dashboards | **semantic model** |
| SQL-first team wants warehouse-style gold | **Data Warehouse** |

## Memory hooks
- Walls ladder: **schemas → lakehouses → workspaces** (more walls, more work)
- **Shortcut = pointer, not a copy**
- **MLV = a view that did its homework and keeps it updated**
- **Gold isn't one thing** — shape follows audience