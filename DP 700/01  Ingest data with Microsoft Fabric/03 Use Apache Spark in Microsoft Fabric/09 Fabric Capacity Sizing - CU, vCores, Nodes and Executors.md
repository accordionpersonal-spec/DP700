
# Fabric Capacity Sizing: CU, vCores, Nodes & Executors

## 1. Checking Your Spark Configuration: Settings vs. Live Runtime

There are two different places to look, depending on what you actually want to know.

### A. Workspace-level pool settings (config, not live)

Go to **Workspace → Workspace settings → Data Engineering/Science → Spark settings → Pool tab**. This shows whether the workspace default is the **Starter Pool** (fixed Medium nodes, autoscale + dynamic executor sliders only) or a **Custom Pool** you defined yourself.

### B. Live/runtime config (what's actually active in your session)

Run this directly in a notebook cell — much faster than digging through the UI:

```python
# Basic session info
print(spark.version)
print(spark.sparkContext.getConf().getAll())

# Specific useful settings
spark.conf.get("spark.executor.instances", "not set (dynamic alloc likely on)")
spark.conf.get("spark.executor.cores")
spark.conf.get("spark.executor.memory")
spark.conf.get("spark.dynamicAllocation.enabled")
spark.conf.get("spark.dynamicAllocation.maxExecutors")
```

> **Tip:** After you run a cell, the **Spark session details / Monitor** link at the top of the notebook opens the Spark UI. The **Environment** tab lists the full config; the **Executors** tab shows the live node/executor count.

---

## 2. Reading the Pool Settings UI

### Starter Pool (workspace default)

* **Default pool dropdown:** Every notebook/job in the workspace uses this pool unless overridden. A workspace can have multiple pools, but only one is ever "default."
* **Node family — Memory optimized:** Starter pools are locked to this family (skewed RAM-to-vCore ratio — good for caching, joins, and shuffles, i.e. typical Delta/Parquet workloads).
* **Node size — Medium (fixed):** Starter pools can't be resized; you're locked to Medium. This is exactly where `spark.driver.cores` / `spark.executor.cores` values come from at runtime.
* **Number of nodes (e.g. 1–2):** This is the **autoscale range** in raw node count — Fabric runs with as few as 1 node and scales up to the max if the workload demands it.
* **Job bursting note:** Tells you your capacity supports burst (up to 3x vCores), but a Starter Pool only lets a single job use whatever max node count the admin configured — it does not automatically unlock your full capacity's burst headroom.
* **"Customize compute configurations for items":** When **On**, individual notebooks/jobs can override the workspace default pool with their own compute settings. When **Off**, every item in the workspace is forced onto the default pool with no exceptions.
* **"Optimize for your use case" wizard:** A guided form that back-fills these same settings based on questions like "read-heavy or write-heavy?" — functionally identical to setting the fields manually.

### Custom Pool (Create New Pool dialog)

* **Node family / Node size:** Unlike Starter, you get real choices here — Small / Medium / Large / XLarge / XXLarge, and (for custom pools) **Compute optimized** as an alternative to Memory optimized.
* **Autoscale (min → max nodes):** Directly controls how many worker nodes can spin up. Raising the max is how you actually claim more of your capacity's burst headroom — Starter Pool caps you at a fixed 1–2.
* **Dynamically allocate executors (min → max):** A separate layer from node autoscale — it decides how many executor **processes** actually spin up across your available nodes, based on real-time task demand.

---

## 3. Node Size Reference Table

| Node size | vCores | Memory |
| --- | --- | --- |
| Small | 4 | 32 GB |
| Medium | 8 | 64 GB |
| Large | 16 | 128 GB |
| XLarge | 32 | 256 GB |
| XXLarge | 64 | 512 GB |

**Node family** determines the vCore-to-RAM ratio for a given size:

* **Memory optimized** — more RAM per core; good for caching, joins, and shuffles (typical Delta/Parquet workloads). This is the only family Starter Pools offer.
* **Compute optimized** — more vCores per GB of RAM; better for CPU-heavy, less memory-hungry work. Only available on custom pools.

---

## 4. Nodes vs. Executors: The Critical Distinction

* A **node** is a virtual machine — the physical unit that autoscale adds or removes.
* An **executor** is a process that runs *inside* a node — the unit that dynamic allocation adds or removes.

Think of it as: **node autoscale** = how many machines ("workshops") you're allowed to spin up, and **executor allocation** = how many worker-teams (processes) actually get assigned within those machines.

You can have 10 nodes physically provisioned but still be running with just 1 active executor if the dynamic allocation slider stays at 1 — wasting the extra nodes. The two sliders must both be raised to get true multi-node distributed processing.

---

## 5. Capacity Unit (CU) → vCore → Burst Formulas

Fabric capacity is purchased in **Capacity Units (CU)**, e.g. an F64 SKU. Here's the exact chain that turns a CU number into an actual node/executor budget:

1. **CU → base vCores:** `Total vCores = CU × 2` — each capacity unit maps to two Spark vCores. (F64 → 64 × 2 = 128 vCores base.)
2. **Base → burst vCores:** `Burst vCores = Base vCores × 3` — Fabric capacities support bursting, allowing consumption of up to 3× your purchased Spark vCores. (128 × 3 = 384.)
3. **Combined shortcut:** `Max burst vCores = CU × 2 × 3 = CU × 6`. Check: F64 → 64 × 6 = 384 ✓.
4. **vCores → nodes:** `Total vCores used = Node count × vCores per node`. E.g. Medium nodes (8 vCores each) × 48 max nodes = 384 — which is why 48 is the max node count allowed on Medium at full F64 burst.
5. **vCores per node → driver/executor cores:** `vCores per node = spark.driver.cores` (for the driver's node) and `= spark.executor.cores` (for each executor node).
6. **Current vCore usage:** `Your vCore usage = driver.cores + (executor.cores × executor.instances)`.

One-line version of the whole chain:

```text
CU → (×2) → Base vCores → (×3 burst) → Max vCores → (÷ vCores/node) → Max node count
```

---

## 6. Worked Example

Given a live config dump showing `spark.driver.cores = 8`, `spark.executor.cores = 8`, `spark.executor.instances = 1`:

* **Current vCore usage:** `8 + (8 × 1) = 16 vCores` consumed by the session.
* **On an F64 capacity:** `64 CU → 384 max vCores` → at 8 vCores/node (current node size) → a theoretical ceiling of **48 nodes**.
* **But note:** if the workspace is still on its Starter Pool (Medium × 2 nodes max), the *actual* ceiling for that pool is `8 × 2 = 16 vCores` — the extra burst headroom exists on the capacity but isn't reachable until a custom pool with a higher max node count is configured.

See also: [02 Spark Pool Architecture](02%20Spark%20Pool%20Architecture.md) for the driver/worker roles behind these numbers, and [03 Spark Pools in Microsoft Fabric](03%20Spark%20Pools%20in%20Microsoft%20Fabric.md) for the Starter vs. Custom pool comparison.

---
