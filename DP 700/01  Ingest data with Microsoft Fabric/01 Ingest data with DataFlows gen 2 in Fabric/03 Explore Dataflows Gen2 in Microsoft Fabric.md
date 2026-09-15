## TL;DR

Dataflows Gen2 provide a low-code/no-code ETL interface powered by Power Query Online to visually transform data. They support a wide range of source connectors and allow landing transformed data into Fabric internal stores (Lakehouse, Warehouse, SQL DB) or external Azure services.

---

## Core Concepts

* **Power Query Online Interface:** Comprises five main UI areas: Ribbon, Queries pane, Diagram view, Data Preview pane, and Query Settings pane.
* **Queries & Staging Logic:** Individual data sources are managed as "queries". Disabling load on a query allows creating intermediate staging queries that feed referenced or merged tables without writing unnecessary output to the target destination. → *how do I add a 2nd source? see [01.4 Adding Multiple Sources](01.4%20Adding%20Multiple%20Sources.md)*
* **Query Reordering & Step Execution:** Transformations are logged sequentially in the **Applied Steps** list. Steps can be renamed, reordered, deleted, or edited via the gear icon or underlying M code in the Advanced Editor.
* **Supported Destinations:** Natively supports landing transformed queries into Fabric Lakehouses, Warehouses, and Fabric SQL databases, as well as external destinations including Azure SQL Database, Azure Data Explorer (KQL), and Azure Synapse Analytics.

---

## Exam Focus

* **Query Reference vs. Duplicate:**
* **Duplicate:** Creates an independent copy of the query definition; changes in the parent do not propagate.
* **Reference:** Creates a downstream query that branches off the parent query's state; updates in the parent flow through to downstream queries (essential for building star-schema dimensions/facts from a single flat extract).


* **Query Folding Verification:**
* DP-700 frequently tests transformation pushdown. Verify query folding by right-clicking an applied step in Query Settings to check **View Native Query**. If available, computation is pushed down to the source database rather than evaluated in the Power Query engine. → *lost? see [01.3 Query Folding](01.3%20Query%20Folding.md)*


* **Disable Load Feature:**
* Use **Disable Load** on intermediate queries so only final, fully-cleansed star-schema tables land in the data store.


* **Data Destination Configuration:**
* Configured per query in the **Query Settings pane**. Memorize the supported target destinations (Lakehouse, Warehouse, Fabric SQL DB, Azure SQL DB, Azure Synapse, Azure Data Explorer).



---

## Gotchas

* **Merge vs. Append:** **Merge** performs horizontal joins (adds columns via key matching), whereas **Append** performs vertical unions (combines rows across matching schemas).
* **Data Preview vs. Full Execution:** The Data Preview pane operates on a **sample subset of rows** for speed; step validation in preview does not process full data volume until the dataflow is published and executed.
* **Diagram View Lineage:** Visualizing query steps in Diagram View helps trace query dependencies, but editing underlying M code directly in Advanced Editor can break visual step rendering if custom M syntax is used.

---

## Quick Recall

* Built on **Power Query Online** with visual step tracking and full **M code** customization via Advanced Editor.
* Can be created in **Data Factory**, **Power BI Workspaces**, or directly inside a **Lakehouse**.
* **Reference** creates a linked downstream query; **Duplicate** creates an isolated clone.
* **Disable Load** keeps intermediate lookup or staging queries in memory without outputting them to storage.
* Right-click steps to check **View Native Query** for **Query Folding** status.
* Native output targets: **Lakehouse**, **Warehouse**, **Fabric SQL DB**, **Azure SQL DB**, **Azure Synapse**, and **Azure Data Explorer**.
* Use **Diagram View** to visually map transformation lineage and query relationships.