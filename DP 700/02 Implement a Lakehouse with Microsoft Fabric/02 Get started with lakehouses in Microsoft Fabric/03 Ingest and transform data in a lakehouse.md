
# Ingest and Transform Data in a Lakehouse

---

## 1. Lakehouse Architecture & Workspace Features

When you create a lakehouse within a Fabric-enabled workspace, it provides two primary storage areas alongside a dedicated T-SQL endpoint[cite: 1]:

* **`Tables/` Folder:** Stores structured **Delta Lake** tables[cite: 1]. Enforces schemas, provides ACID transaction guarantees, enables T-SQL querying, and integrates directly with Power BI[cite: 1].
* **`Files/` Folder:** Stores raw or semi-structured files in native formats (e.g., CSV, JSON, Parquet)[cite: 1]. It does not enforce schemas, serving as a flexible landing zone for initial data staging and exploration[cite: 1].

### Schemas & Namespaces
* Schemas are enabled by default, automatically generating a default `dbo` schema[cite: 1].
* Schemas allow organizing tables into domain-specific logical groups (e.g., `sales`, `hr`, `marketing`)[cite: 1].
* Supports **schema-level security permissions** and cross-workspace querying using four-part object namespaces (`workspace.lakehouse.schema.table`)[cite: 1].

### Dual Operational Modes

| Operational Mode | Purpose & Capabilities | Modification Rights |
| :--- | :--- | :--- |
| **Lakehouse Explorer** | Manage physical files, folders, and Delta tables[cite: 1]. Allows uploading files, generating tables, and adding reference lakehouses side-by-side[cite: 1]. | **Read / Write**[cite: 1] |
| **SQL Analytics Endpoint** | Execute T-SQL queries against Delta tables[cite: 1]. Build views, inline functions, and apply Row/Column Level Security[cite: 1]. | **Read-Only** (Data cannot be modified directly via T-SQL)[cite: 1] |

---

## 2. Ingestion Methods

Fabric provides multiple code and no-code ingestion methods to support diverse ETL requirements[cite: 1]:

1. **Direct File Upload:** Upload local files or directories directly via Lakehouse Explorer[cite: 1].
2. **Load to Table (No-Code GUI):** Select a Parquet or CSV file/folder in Lakehouse Explorer and convert it directly into a Delta table with append or overwrite capabilities[cite: 1].
3. **Dataflows Gen2:** Ingest and transform data visually using the low-code Power Query interface[cite: 1].
4. **PySpark / Spark Notebooks:** Programmatically extract, process, and load datasets at scale[cite: 1].
5. **Data Factory Pipelines:** Move large volumes of data from external cloud/on-premise sources using the **Copy Data** activity[cite: 1].

---

## 3. Data Integration via OneLake Shortcuts

**Shortcuts** create symbolic references to external data sources without copying underlying storage files[cite: 1].

* **Zero Data Duplication:** Reduces data copying across systems and storage boundaries[cite: 1].
* **Unified Security:** OneLake manages target permissions using pass-through identity authentication (requires read access on the target location)[cite: 1].
* **Supported Targets:** Other Fabric lakehouses, external Azure Data Lake Storage Gen2 (ADLS Gen2), and Amazon S3[cite: 1].
* **Schema Shortcuts:** Map an entire schema directory of Delta tables in ADLS Gen2 or another lakehouse so they present as local tables within the local schema namespace[cite: 1].

---

## 4. Data Transformation Tools

Once raw data lands in the `Files/` zone, you can transform and move it to structured `Tables/` using several core toolsets[cite: 1]:

| Transformation Tool | Target User Persona | Key Capabilities & Features |
| :--- | :--- | :--- |
| **Spark Notebooks** | Data Engineers (PySpark, SQL, Scala)[cite: 1] | Code-driven transformation engine[cite: 1]. Built-in Copilot support converts natural language into PySpark code and explains complex code logic[cite: 1]. |
| **Dataflows Gen2** | BI Analysts & Citizen Developers[cite: 1] | Uses visual Power Query interface familiar to Power BI and Excel users[cite: 1]. |
| **Data Factory Pipelines** | Data Engineers & Integration Architects[cite: 1] | Graphical orchestrator to execute multi-step activities in parallel or sequential workflows[cite: 1]. |

```