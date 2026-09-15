## TL;DR

This hands-on lab demonstrates end-to-end data ingestion using Dataflows Gen2 to extract CSV data from a web endpoint, apply transformations (adding calculated M columns and casting types), and load it into a Lakehouse Delta table using the **Append** load pattern, followed by pipeline orchestration.

---

## Core Concepts

* **M-Based Custom Columns:** Transformations use Power Query M expressions (e.g., `Date.Month([OrderDate])`) directly inside the Dataflows Gen2 visual editor.
* **Automatic Target Ingestion:** Creating a Dataflow Gen2 directly from within a Lakehouse automatically sets that Lakehouse as the default target destination.
* **Connection Identities:** Accessing external URLs uses Anonymous authentication, while landing data into internal Lakehouse destinations requires authenticating via an organizational account identity.
* **Table Schema & Destination Settings:** Disabling "Use automatic settings" allows explicitly switching destination load modes (e.g., **Append** vs. **Replace**) and mapping explicit column data types.

---

## Gotchas

* **Automatic vs. Manual Destination Binding:** If you launch Dataflow Gen2 from inside the Lakehouse UI, the destination auto-attaches; launching it from the Workspace item list requires manually invoking **Add data destination**.
* **Destination Data Type Mismatches:** Although destination schemas can be edited on the destination settings screen, changing data types in the **Power Query Applied Steps** pane is the recommended best practice to prevent pipeline schema drift errors.
* **Legacy Power BI Connector Name:** Connecting Power BI Desktop directly to a published Fabric Dataflow Gen2 uses the connector named **Power BI dataflows (Legacy)** in Desktop.

---

## Quick Recall

* Ingests web CSVs via Anonymous auth URL connections.
* Uses **M formula language** (`Date.Month()`) for custom calculated columns.
* Applied transformations track sequentially in the **Applied Steps** pane and visually via **Diagram View**.
* Supports **Append** (preserving existing target table rows) and **Replace** destination modes.
* Target table names (e.g., `orders`) are explicitly configured in the Lakehouse destination settings.
* Orchestrated inside Data Pipelines via the native **Dataflow activity**.
* Power BI Desktop connects directly using the **Power BI dataflows (Legacy)** connector.