## TL;DR

Dataflows Gen2 serve as Microsoft Fabric's low-code/no-code ETL mechanism powered by the Power Query M engine. They allow you to extract, transform, and load data from disparate sources into Fabric destinations (Lakehouse, Warehouse, KQL Database) and integrate seamlessly with Fabric Data Pipelines for orchestration.

---

## Core Concepts

* **Power Query M Engine:** Provides visual data wrangling and transformation capabilities, generating M code under the hood for repeatable ETL execution. → *lost? see [01.1 Power Query M Engine](01.1%20Power%20Query%20M%20Engine.md)*
* **Native Data Destinations:** Unlike legacy Dataflows (Gen1), Gen2 allows landing transformed data directly into Fabric artifacts (Lakehouse Delta tables, Warehouse tables, KQL DB) or external destinations like Azure SQL.
* **Staging Compute Engine:** Dataflows Gen2 automatically leverage Fabric backend staging (using Lakehouse Parquet/SQL compute) to offload heavy transformations, enabling faster parallel processing and data landing. → *lost? see [01.2 Staging Compute Engine](01.2%20Staging%20Compute%20Engine.md)*
* **Pipeline Activity Integration:** Operates as a native activity in Fabric Data Pipelines, enabling parameter passing, failure handling, and scheduled end-to-end orchestration.

---

## Exam Focus

* **Dataflows Gen2 vs. Copy Activity:**
* Use **Copy Activity (Pipeline)** for high-throughput, raw bulk ingestion without transformation (ELT pattern).
* Use **Dataflows Gen2** for low-code/no-code transformations, cleansing, and type casting prior to landing (ETL pattern).


* **Dataflows Gen2 vs. Notebooks:**
* Choose **Dataflows Gen2** for business analyst accessibility, small-to-medium dataset wrangling, or complex visual transformations without writing Spark code.
* Choose **Notebooks (PySpark)** for heavy big-data transformations, complex algorithmic processing, or custom Delta Lake manipulations.


* **Destination Update Modes:**
* Memorize the distinction between **Append** (adds new rows) and **Replace** (drops and recreates/overwrites table target schema and data).


* **Staging Mechanics:**
* Staging is enabled by default to boost transformation performance using Fabric internal Lakehouse compute before writing to the target destination.



---

## Gotchas

* **Dataflows Gen1 vs. Dataflows Gen2:** Gen1 stores output in Power BI workspaces/Dataverse as internal entities; Gen2 lands output directly into Fabric items (Lakehouse, Warehouse, etc.) and supports staging offload.
* **Query Folding Limits:** If staging is disabled, all M transformations must fold back to the source; non-folding operations will fail or fallback to local memory processing. → *lost? see [01.3 Query Folding](01.3%20Query%20Folding.md)*
* **Schema Drift on Replace:** Using **Replace** destination mode drops target tables, which can invalidate downstream Power BI semantic model relationship bindings or schema enforcement in Delta tables.

---

## Quick Recall

* Powered by the **Power Query M engine** for visual, low-code data wrangling.
* Directly lands cleansed data into **Lakehouse (Delta)**, **Warehouse**, **KQL DB**, and **Azure SQL**.
* **Staging engine** offloads transformations to Fabric backend compute for high performance.
* Callable as a native **Dataflow Activity** within Fabric Data Pipelines.
* Supports **Append** and **Replace** load strategies at the destination level.
* Use **Copy Activity** for raw ingestion speed; use **Dataflows Gen2** when UI-based cleansing is required before landing.
* Seamlessly acts as a reusable data source for Power BI semantic modeling.