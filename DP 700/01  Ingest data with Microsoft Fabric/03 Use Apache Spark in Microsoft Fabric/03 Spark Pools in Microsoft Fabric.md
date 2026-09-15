**Part 3: Spark Pools in Microsoft Fabric (Starter vs. Custom Pools)**

---

### Core Concept

In Microsoft Fabric, Spark compute management is abstracted into two main pool types: **Starter Pools** (preconfigured, instant-start default) and **Custom Spark Pools** (user-defined for specific resource needs).

---

### Starter Pools vs. Custom Pools

| Feature | Starter Pool (Default) | Custom Spark Pool |
| --- | --- | --- |
| **Startup Time** | **5 to 10 seconds** (Uses pre-warmed background clusters) | **2 to 3 minutes** (Standard provision) or **~5s** (If configured as Custom Live Pool) |
| **Setup Effort** | Zero configuration required; created automatically per workspace. | Manual setup by Workspace Admins. |
| **Node Size** | Fixed to **Medium** node families. | Customizable (Small, Medium, Large, Extra Large, Single Node). |
| **Best Used For** | Ad-hoc analytics, quick exploratory notebooks, and fast testing. | Heavy ETL pipelines, memory-intensive jobs, strict SLA tasks, or cost control. |

---

### Key Pool Configuration Settings

When tuning or creating custom pools, you control four critical parameters:

* **Node Family:** Virtual machine sizing optimized primarily for **Memory-Optimized** tasks.
* **Autoscale:** Automatically adds or removes worker nodes based on real-time execution load (between a specified minimum and maximum threshold).
* **Dynamic Allocation:** Automatically scales the number of *executor processes* running on worker nodes up or down as data volume changes during execution.
* **Capacity Admin Controls:** Capacity-level administrators can restrict workspace admins from creating custom pools to enforce organization-wide governance and cost guardrails.

---

> **Going deeper:** For the exact Capacity Unit (CU) → vCore → node/executor math behind these settings — plus how to inspect a workspace's Pool tab and a live session's config side by side — see [09 Fabric Capacity Sizing - CU, vCores, Nodes and Executors](09%20Fabric%20Capacity%20Sizing%20-%20CU%2C%20vCores%2C%20Nodes%20and%20Executors.md).