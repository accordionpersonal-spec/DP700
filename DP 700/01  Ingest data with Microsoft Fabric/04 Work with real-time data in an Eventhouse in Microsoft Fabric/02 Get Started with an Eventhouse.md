
# Get Started with an Eventhouse

You've never created or queried a KQL database before, so this file starts from zero — what exists inside an Eventhouse, how data gets in, and how to write your very first KQL query by leaning on SQL you already know.

---

## 1. What You Get When You Create an Eventhouse

* Creating an Eventhouse automatically creates a **default KQL database with the same name** — you don't start with an empty container.
* One Eventhouse can hold **one or more KQL databases**. Use the default one, or create additional ones if you need to separate data by team, source, or purpose.
* Inside a KQL database you can create: **tables**, **stored procedures**, **materialized views**, **functions**, **data streams**, and **shortcuts** — the same building blocks you'd expect from a database, just tuned for streaming/time-series data.

---

## 2. Getting Data Into a KQL Database

There are three distinct ways data ends up inside your Eventhouse:

### A. Direct ingestion
Point the KQL database at a source and let it pull data in:
* Local files, Azure Storage, Amazon S3
* Azure Event Hubs, Fabric Eventstream, Real-Time hub
* OneLake, Data Factory copy, Dataflows
* Connectors for Apache Kafka, Confluent Cloud Kafka, Apache Flink, MQTT, Amazon Kinesis, Google Cloud Pub/Sub

### B. Database shortcuts
A **shortcut** lets you query an existing KQL database in *another* Eventhouse — or even an Azure Data Explorer database — as if it were stored locally, **without copying the data**. Same idea as a Lakehouse shortcut, just pointed at KQL data instead of files.

### C. OneLake availability
You can flip on **OneLake availability** for a specific KQL database or table, which exposes it to the rest of the Fabric ecosystem — Power BI, Warehouse, Lakehouse — for cross-workload queries, again without duplicating the data.

---

## 3. Querying: The KQL Queryset

When you create a KQL database, Fabric automatically attaches a **KQL queryset** — your workspace for running and managing queries against it.

* Supports both **KQL** and **T-SQL** — you're not forced to learn a new language on day one if you just want to run familiar SQL.
* Lets you save queries, organize them across multiple tabs, and share them with teammates.
* Query results can be rendered directly as charts/tables for quick visual exploration.

---

## 4. KQL Syntax Basics — Mapped to SQL You Already Know

### The pipeline mental model

KQL reads like a **funnel**, not a single SQL statement. You start with a whole table, then pipe (`|`) it through a chain of operators — each one transforms whatever came out of the step before it. Order matters, because every step only sees the result of the previous step (unlike SQL, where clause order is fixed by the language regardless of how you conceptually build the query).

> **Important — case sensitivity:** KQL is case-sensitive for *everything*: table names, column names, function names, operators, keywords, and string values. `TaxiTrips`, `taxitrips`, and `TAXITRIPS` are three different identifiers. This is a common gotcha coming from SQL, which is usually case-insensitive for identifiers.

### Operator-to-SQL cheat sheet

| KQL | Rough SQL equivalent | What it does |
| --- | --- | --- |
| `TableName` | `SELECT * FROM TableName` | Simplest possible query — returns all columns (row count limited by your query tool's defaults). |
| `\| where <condition>` | `WHERE <condition>` | Filters rows. |
| `\| project col1, col2` | `SELECT col1, col2` | Picks specific columns. |
| `\| take N` | `SELECT TOP N *` | Returns the first N rows — useful for sampling a large table without scanning all of it. |
| `\| summarize x = agg() by col` | `GROUP BY col` (with an aggregate) | Aggregates rows into a summary table. |

### Worked examples

**1. Return everything (bounded by tool defaults):**
```kql
TaxiTrips
```

**2. Sample 100 rows to explore structure, cheaply:**
```kql
TaxiTrips
| take 100
```

**3. Filter, pick columns, then limit — same query as the SQL you'd write, just piped:**
```kql
TaxiTrips
| where fare_amount > 20
| project trip_id, pickup_datetime, fare_amount
| take 10
```
Equivalent SQL: `SELECT TOP 10 trip_id, pickup_datetime, fare_amount FROM TaxiTrips WHERE fare_amount > 20`

**4. Aggregate — count trips per taxi:**
```kql
TaxiTrips
| summarize trip_count = count() by taxi_id
```
Equivalent SQL: `SELECT taxi_id, COUNT(*) AS trip_count FROM TaxiTrips GROUP BY taxi_id`

---

## 5. Copilot for Real-Time Intelligence

If your admin has enabled it, **Copilot** shows up as a pane next to the query editor inside a queryset. Ask your question in plain English and it generates the KQL for you — a genuinely useful crutch while you're still building fluency with the pipeline syntax and operator names above.

---
