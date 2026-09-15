### 1. TL;DR

Covers Microsoft Fabric Data Pipelines for orchestrating ETL/ELT processes using Copy Data, Dataflows (Gen2), Notebooks, and Stored Procedures. It focuses on activity-based orchestration, parameterization, dynamic control flows, and scheduling vs. on-demand executions.

---

### 2. Core Concepts

* **Orchestration Engine:** Built on Azure Data Factory (ADF) engine under the hood, adapted for native Fabric items (Lakehouse, Warehouse, KQL Database, Mirrored DBs).
* **Data Transformation Activities:** Heavy-lifting tasks. Use **Copy Data** for raw/fast ingestion into OneLake (Files or Delta Tables); **Dataflow Gen2** for no-code Power Query ETL; **Notebook** for PySpark/Scala scale-out jobs; **Stored Procedure** for SQL Warehouse transformations.
* **Control Flow Activities:** Execution logic handlers such as `For Each`, `If Condition`, `Until`, `Switch`, `Set Variable`, and `Execute Pipeline`.
* **Dependency Conditions:** Links between activities based on outcome: **Upon Success** (Green), **Upon Failure** (Red), **Upon Completion** (Blue), or **Upon Skip** (Gray).
* **Parameters vs. Variables:**
* *Parameters:* External inputs passed at trigger/schedule time (read-only during run).
* *Variables:* Internal pipeline values updated dynamically during run via `Set Variable` / `Append Variable`.



---

### 3. Exam Focus: DP-700 High-Yield

* **Copy Data Activity Limits & Behavior:**
* **Direct Ingestion:** Target can be direct Lakehouse Delta tables or raw Files.
* **Auto-create Table:** Copy activity can automatically create target Delta tables, mapping types to standard Delta/Parquet data types.
* **Binary Copy:** Always select Binary copy when copying files as-is (e.g., raw Parquet/CSV to Lakehouse `Files/` section without schema parsing).


* **Dataflow Gen2 vs. Copy Activity:**
* Use **Copy Data** for maximum throughput when moving data without complex transformations.
* Use **Dataflow Gen2** when row-level filtering, unpivoting, or low-code Power Query transforms are required *during* ingestion.


* **Parameter Syntax:** Expression Language uses `@pipeline().parameters.ParamName` or `@activity('ActivityName').output...`.
* **Lookup vs. Get Metadata:**
* *Lookup:* Reads a dataset/table and returns a dataset array (up to 5,000 rows / 4MB).
* *Get Metadata:* Returns metadata attributes (child items, existence, file size, last modified date).


* **High-Throughput / Capacity Units (CUs):** Pipeline activity executions consume Fabric Capacity CUs based on run duration and target compute engines (Spark vs. Dataflow Gen2 vs. Pipeline orchestration).

---

### 4. Gotchas & Distinctions

| Feature / Scenario | Copy Data Activity | Dataflow Gen2 | Notebook Activity |
| --- | --- | --- | --- |
| **Primary Use Case** | Fast raw ingestion / movement | Low-code Power Query transformation | Big Data PySpark/SQL transformations |
| **Compute Engine** | Pipeline Data Movement Engine | Mashup Engine + Staging Lakehouse | Spark Cluster |
| **Staging Requirement** | Optional (used for staging sources) | Automatically uses Lakehouse staging | None |
| **Transformations** | Schema mapping, column rename/typecast | 300+ Power Query transforms | Code-first, complex Delta Lake operations |

* **Dataflow Gen2 Staging Lakehouse:** Dataflows Gen2 natively staged data in an internal Lakehouse prior to writing to the destination. If an error occurs in a Dataflow Gen2 activity within a pipeline, check staging configurations.
* **Execute Pipeline vs. Web Activity:** Use `Execute Pipeline` for modular design (child pipelines) with synchronous/asynchronous waiting options. Use `Web` / `Web Activity` for calling external REST APIs or triggering external webhooks.
* **For Each Iteration Limits:** `For Each` runs sequentially by default if configured, or up to **50 concurrent iterations** in parallel.

---

### 5. Quick Recall

* **Copy Data = Max throughput, minimal transform** (Use for landing raw data into OneLake `Files/`).
* **Dataflow Gen2 = Low-code ETL** (Uses Power Query; automatically stages data in Lakehouse).
* **Notebook Activity = Code-first scale-out** (Best for Medallion Bronze $\rightarrow$ Silver $\rightarrow$ Gold processing).
* **Lookup Activity Limit = 5,000 rows / 4 MB** (If exceeded, pipeline execution fails).
* **Parameters are immutable per run; Variables are dynamic** (Set via `Set Variable`).
* **Activity Outcomes:** Success, Failure, Completion, Skipped drive control flow branching.
* **Get Metadata** returns child item lists or file attributes without loading file content.