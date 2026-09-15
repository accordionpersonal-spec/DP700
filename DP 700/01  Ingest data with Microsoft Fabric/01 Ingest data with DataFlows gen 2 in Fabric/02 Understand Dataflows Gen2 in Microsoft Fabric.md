## TL;DR

Dataflows Gen2 provide a reusable, low-code ETL/ELT layer in Fabric using Power Query Online to clean and transform data before landing it in destinations like Lakehouse or Warehouse. They enable self-service data prep, reduce source system load by staging extraction once, and can be run independently or orchestrated via Data Pipelines.

---

## Core Concepts

* **ETL vs. ELT Orchestration Patterns:**
* **ETL Pattern:** Source $\rightarrow$ Dataflow Gen2 (Transform) $\rightarrow$ Land to Destination (Lakehouse/Warehouse Delta tables).
* **ELT Pattern:** Source $\rightarrow$ Pipeline Copy Activity (Raw Land) $\rightarrow$ Dataflow Gen2 (Cleanse/Transform) $\rightarrow$ Curated Gold Layer/Semantic Model consumption.


* **Reusable Data Logic Layer:** Acts as an intermediate visual transformation layer that abstracts source connection complexity and prevents duplicate connections/queries against fragile underlying source systems.
* **Horizontal Partitioning (Reusable Modules):** A central/global dataflow cleanses core entities (e.g., standard Date dimension, Customer master); downstream analysts pull from this standard dataflow to build domain-specific semantic models.
* **Service Discoverability:** Dataflows Gen2 published in Fabric workspaces can be directly discovered and connected to via Power BI Desktop, serving as direct inputs for report modeling.

---

## Exam Focus

* **Dataflow Gen2 vs. Data Warehouse:** Dataflows Gen2 are *transformation engines and staging vehicles*, **not** storage or serving layers; they do not replace a Lakehouse or Data Warehouse.
* **Security Constraints (Key Exam Trap):** Dataflows Gen2 do **NOT** support Row-Level Security (RLS) or Object-Level Security (OLS). RLS must be implemented at the destination layer (SQL Analytics Endpoint, Warehouse, or Semantic Model).
* **Workspace Licensing Requirements:** Dataflows Gen2 require an active **Fabric Capacity workspace** (F SKU or P SKU); they cannot run under standard Power BI Shared Pro/PPU workspaces.
* **Source Load Optimization:** Select Dataflows Gen2 when you need to extract data once from slow/legacy source systems and reuse the transformed output across multiple downstream analysts.

---

## Gotchas

* **Security Delegation:** Relying on Dataflows Gen2 for data access control is a security risk—since RLS is unsupported, users with read access to the Dataflow output can see all transformed rows unless security is enforced downstream.
* **Dataflows Gen2 as Source vs. Destination:** A Dataflow Gen2 can act as an ETL transformation pipeline *that lands data into a Lakehouse*, OR it can act as an in-memory transformation layer *consumed directly by Power BI*. Don't confuse its dual role as an ingestion activity vs. a Power BI data source.
* **Direct Pipeline Copy vs. Dataflow ELT:** Copy Activity offers parallelized high-speed movement without row-by-row inspection; using Dataflow Gen2 purely to move data without needing transformations incurs unnecessary Power Query engine overhead.

---

## Quick Recall

* Uses **Power Query Online** interface for low-code/no-code data transformations.
* **Row-Level Security (RLS) is NOT supported** inside Dataflows Gen2—enforce RLS downstream in Lakehouse/Warehouse/Semantic Model.
* Cannot replace a Data Warehouse; it is an **ETL processing engine**, not a data storage artifact.
* Requires a **Fabric Capacity workspace** to execute.
* Reduces source load by extracting and cleansing data once for multi-analyst reuse.
* Can land transformed data directly to destinations or be consumed directly by **Power BI Desktop**.
* Can be triggered manually, on a **refresh schedule**, or orchestrated via **Fabric Data Pipelines**.