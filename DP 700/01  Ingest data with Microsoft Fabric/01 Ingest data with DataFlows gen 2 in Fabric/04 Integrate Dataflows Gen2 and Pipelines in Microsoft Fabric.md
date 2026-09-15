## TL;DR

Dataflows Gen2 integrated into Fabric Data Pipelines allow visual ETL transformations to be orchestrated alongside downstream actions like PySpark Notebooks, Stored Procedures, and Copy activities. This combination shifts Dataflows Gen2 from standalone manual/scheduled data preps into fully automated, event- or schedule-driven enterprise pipelines.

---

## Core Concepts

* **Dataflow Activity:** A native pipeline activity used to invoke and execute a Dataflow Gen2 within an orchestration graph.
* **Control Flow & Dependency Chaining:** Enables conditional sequencing (On Success, On Failure, On Completion) between Dataflows Gen2 and other pipeline activities.
* **Hybrid Orchestration:** Allows performing low-code cleansing in Dataflows Gen2, followed immediately by code-first processing (PySpark Notebooks or SQL Stored Procedures) on the landed data.
* **Pipeline Triggers & Automation:** Provides scheduled or event-driven execution for Dataflows Gen2, removing the need for manual refreshes.

---

## Exam Focus

* **Dataflow Scheduling vs. Pipeline Scheduling:**
* Scheduling a Dataflow directly only triggers that specific dataflow.
* Scheduling a **Pipeline** containing a Dataflow allows running pre/post-processing tasks (e.g., Get Metadata, Stored Procedures, alerting) along with the dataflow execution.


* **Common Pipeline Activity Types:** Memorize key activities commonly combined with Dataflows Gen2: **Copy data**, **Notebook**, **Get metadata**, and **Execute script or stored procedure**.
* **Failure Handling & Notifications:** DP-700 frequently tests error handling patterns—linking the **On Failure** path of a Dataflow activity to a Teams/Email/Web notification activity.
* **Parameter Passing:** Dynamic parameters can be passed from the Data Pipeline into the Dataflow Gen2 query execution at runtime.

---

## Gotchas

* **Duplicate Execution Risks:** If a standalone schedule is configured on the Dataflow Gen2 *and* it is also scheduled via a Data Pipeline, the Dataflow will execute twice. Always disable the standalone schedule when orchestrating via pipelines.
* **Synchronous Execution:** A Dataflow activity in a pipeline runs synchronously; downstream activities will not start until the Dataflow Gen2 completes all transformations and finishes landing data to its destination.
* **Copy Activity vs. Dataflow Activity:** Use **Copy Activity** for maximum throughput when landing raw data without transformation; use **Dataflow Activity** when UI-driven M transformations are needed before landing.

---

## Quick Recall

* Dataflows Gen2 run as native **Dataflow activities** inside Fabric Data Pipelines.
* Pipelines automate execution via **scheduled or trigger-based** mechanisms.
* Frequently chained with **Copy data**, **Notebook**, **Get metadata**, and **Script/Stored Procedure** activities.
* Enables post-ingestion tasks like executing stored procedures on landed data.
* Use pipeline dependency paths (**On Success / On Failure / On Completion**) for robust error handling.
* Always disable standalone Dataflow refresh schedules when orchestrating through Data Pipelines.