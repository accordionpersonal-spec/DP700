**Part 1: Introduction to Real-Time Data in an Eventhouse**

---

### Core Concept

An **Eventhouse** is Fabric's home for data that never stops arriving. Under the hood, it's a container for one or more **KQL databases** (powered by the Kusto engine — the same tech behind Azure Data Explorer), purpose-built to ingest, store, and query high-velocity, continuously-streaming data with very low latency.

---

### Why Not Just Use the Lakehouse/Warehouse You Already Know?

This is the real question to answer before anything else:

* **Lakehouse / Warehouse** = built for data **at rest**. Files or batches land periodically, get curated into structured tables, and get queried for reporting/BI — "how did we do this month" matters more than "what happened 2 seconds ago."
* **Eventhouse / KQL database** = built for data **in motion**. Telemetry, application logs, clickstreams, IoT sensor readings, financial ticks — anything arriving continuously, where you need near-instant answers on both the freshest second *and* months of history, without waiting for a batch or Spark job to run.

**Rule of thumb:** if the source pattern is "an event just happened and is being pushed to me right now," you're in Eventhouse territory. If it's "a file lands once an hour/day," you're in Lakehouse/Warehouse territory. A KQL database can be queried *while it's still being written to* — a Delta table isn't built for that ingestion rate.

---

### Why Real-Time Data Behaves Differently

Real-time data is made of **immutable events** — things that already happened and can never change (a 3:15 PM sensor reading is that reading, forever). That leads to two consequences:

* **Append-only:** new events keep getting added; existing ones are almost never updated or deleted — the opposite of a relational database, where you update rows and join tables through relationships.
* **Automatic time-partitioning:** because every event is permanently pinned to the moment it happened, Kusto files it away by arrival time automatically. A query for "the last 5 minutes" only has to look in one place instead of scanning everything ever ingested — think of it as a conveyor belt that self-organizes as it moves.

---

### What You Can Do Once Data Lands

* **Query** it with **KQL** or **T-SQL**, inside a KQL queryset.
* **Visualize** it with **Real-Time Dashboards**.
* **Automate** reactions to it with **Fabric Activator**.

---

### Key Takeaways

* **Eventhouse** = container for one or more KQL databases, purpose-built for continuously arriving, real-time data — not a replacement for the Lakehouse/Warehouse, a different tool for a different arrival pattern.
* **Ingestion paths:** Eventstream (streaming) or direct ingestion.
* **Consumption:** KQL/T-SQL queries, Real-Time Dashboards, Fabric Activator.
* Automatic time-based partitioning works *because* the data is immutable, append-only, and time-series in nature — unlike mutable, relationship-driven relational data.

---
