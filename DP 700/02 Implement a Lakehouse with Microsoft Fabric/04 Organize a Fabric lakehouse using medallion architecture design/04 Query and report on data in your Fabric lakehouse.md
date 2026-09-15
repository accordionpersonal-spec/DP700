# Serving the Gold Layer — Two Doors In
*DP-700 · Unit: Query and report on gold layer data (need-driven reference)*

## The need
A pipeline nobody can query is a very expensive hobby. Gold exists to
be consumed — but consumers come in two species with different needs:
- **SQL people** (analysts, engineers) → want to write queries
- **Business users** → want dashboards with friendly names, zero SQL

Fabric gives each species its own door. Same gold tables underneath.

---

## Door 1 — SQL analytics endpoint (for SQL people)

**The old pain:** lake data used to mean "copy it into a SQL database
first" or "learn Spark." Either data movement or a new skill tax.

**What it is:** every lakehouse automatically gets a SQL endpoint —
anyone with SQL skills writes **T-SQL** straight over the gold Delta
tables. Save views, create functions, apply SQL security. **Zero data
movement, no separate query layer to build.**

**The exam fact: it's READ-ONLY.**
- SELECT, views, functions, security → yes
- INSERT / UPDATE / DELETE → no
- Writes to lakehouse tables happen through the Spark side
  (notebooks, pipelines, dataflows). The endpoint is a *reading
  surface* over the same files — one writer engine, many readers,
  no write conflicts.
- Need full T-SQL read **and write**? That's what a **Fabric Data
  Warehouse** is for (ties back to "warehouse as gold layer" from
  the planning unit).

---

## Door 2 — Power BI semantic model (for business users)

**The pain it prevents:** if every report author connects straight to
tables, each one re-defines "revenue," re-builds relationships,
re-names columns. The chaos gold killed at the data layer sneaks back
in at the *report* layer — nine dashboards, nine formulas.

**What it is:** from the lakehouse → **New semantic model** → pick the
gold tables to include. In the model you define relationships,
measures, and business-friendly names **once**. All reports connect
to the **model**, never to the raw Delta tables directly.

Think of it as gold's public menu: measures pre-defined, terminology
agreed, one blessed "revenue" formula that every report inherits.

---

## Direct Lake — how the model reads the data

**The old dilemma (pre-Fabric Power BI had two modes, both a tradeoff):**
- **Import mode:** copies data into Power BI. Fast dashboards — but
  it's a *copy*: scheduled refreshes, stale numbers between them,
  and 6 AM refresh failures to babysit.
- **DirectQuery:** no copy, always live — but every visual fires a
  query at the source. Slow dashboards, hammered source.

**Direct Lake = the third way.** The semantic model reads the **Delta
files in OneLake directly** — no imported copy, no refresh step.
Reports always show current data, at import-like speed.

**Why this is even possible:** everything in Fabric is stored as
Delta (Parquet) — a columnar format the Power BI engine can load
natively. Same theme as MLVs and shortcuts: *Delta everywhere is the
enabler.* New data lands in gold → the report sees it. No refresh
pipeline, nothing to babysit.

---

## Exam triggers

| Question says… | Think… |
|---|---|
| analysts want T-SQL over lakehouse, no data movement | SQL analytics endpoint |
| INSERT/UPDATE via T-SQL on a lakehouse | ❌ endpoint is read-only → Warehouse |
| define measures/terminology once for all reports | semantic model |
| reports show current data **without scheduled refresh** | Direct Lake |
| import vs live tradeoff eliminated | Direct Lake (reads Delta in OneLake directly) |

## Memory hooks
- **SQL endpoint = a window into gold** — look, don't touch
- **Semantic model = gold's menu** — one blessed "revenue"
- **Direct Lake = no copy, no refresh, no babysitting** — Delta made it possible