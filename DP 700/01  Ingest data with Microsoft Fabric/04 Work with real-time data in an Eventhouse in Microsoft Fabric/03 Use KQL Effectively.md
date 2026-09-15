
# Use KQL Effectively (Query Optimization)

## 1. Why Optimization Matters

KQL query performance comes down to one thing: **how much data the query has to touch**, not how clever the syntax looks. A well-written query scans thousands of rows instead of millions, touches 3 columns instead of 50, and keeps performing the same way when your table grows 10x.

> **Core principle:** the less data a step has to process, the faster it runs — and every operator downstream of it inherits that savings.

> **Coming from Spark:** this will feel familiar — it's the same instinct behind Catalyst's predicate pushdown and column pruning (see [05 Work with data using Spark SQL](../03%20Use%20Apache%20Spark%20in%20Microsoft%20Fabric/05%20Work%20with%20data%20using%20Spark%20SQL.md)). The difference is *who* does the reordering: Catalyst will silently reorder your filters and joins for you behind the scenes, whereas KQL largely executes your pipeline in the order you wrote it. That means the discipline of ordering operators well is on **you**, not the optimizer.

---

## 2. Filter Data Early and Effectively

Filtering first shrinks everything that comes after it. Eventhouses hold time-series data, so a **time filter** is almost always your most powerful and cheapest filter — KQL uses a time index to jump straight to the relevant range instead of scanning the whole table.

```kql
TaxiTrips
| where pickup_datetime > ago(30min)  // Filter first - uses time index
| project trip_id, vendor_id, pickup_datetime, fare_amount
| summarize avg_fare = avg(fare_amount) by vendor_id
```

### The funnel rule: order filters by how much they eliminate

Stack multiple `where` clauses from **most-eliminating to least-eliminating** — start broad, then narrow:

```kql
TaxiTrips
| where pickup_datetime > ago(1d)    // Time filter first - eliminates most data
| where vendor_id == "VTS"           // Specific vendor - eliminates some data
| where fare_amount > 0              // Value filter - eliminates least data
| summarize trip_count = count()
```

---

## 3. Reduce Columns Early

`project`-ing down to only the columns you actually need cuts resource usage — the effect is bigger the wider the source table is.

```kql
TaxiTrips
| project trip_id, pickup_datetime, fare_amount  // Select columns early
| where pickup_datetime > ago(1d)                // Then filter
| summarize avg_fare = avg(fare_amount)
```

---

## 4. Optimize Aggregations and Joins

Aggregations and joins are the expensive operators — they have to hold and combine large amounts of data, so how you structure them matters more than with a simple filter.

### Aggregations: cap results while exploring

```kql
TaxiTrips
| where pickup_datetime > ago(1d)
| summarize trip_count = count() by trip_id, vendor_id
| limit 1000  // Limit results for exploration
```

### Joins: put the smaller table first

KQL processes the **first** table in a `join` and matches it against the second — so starting with the smaller table means fewer rows to build and match against, making the join cheaper.

```kql
// Good: small vendor table first
VendorInfo
| join kind=inner TaxiTrips on vendor_id

// Avoid: large taxi table first
TaxiTrips
| join kind=inner VendorInfo on vendor_id
```

> **Coming from Spark:** this is the manual version of what Spark's broadcast join does automatically for small tables — same underlying idea (don't make the engine push the big side around), except in KQL you express it by physically ordering the tables in the pipeline rather than relying on the optimizer to pick a strategy for you.

---

## 5. Putting It Together — The General Pipeline Order

When in doubt, structure a query in this order, top to bottom:

1. **Time filter** (cheapest, eliminates the most)
2. **Other filters**, most-selective first
3. **`project`** down to only the columns you need
4. **`join`**, smaller table first
5. **`summarize`**
6. **`limit`/`take`** if you're exploring rather than shipping a final result

---

## 6. Quick Reference

| Technique | Why it matters | Rule of thumb |
| --- | --- | --- |
| Time filter first | Uses the time index instead of a full scan | Always filter on time before anything else |
| Order filters by selectivity | Each filter shrinks what the next one has to process | Broadest filter first, narrowest last |
| `project` early | Fewer columns = less data moved through the rest of the pipeline | Project right after (or before) your first filter |
| `limit` during exploration | Avoids materializing huge result sets while you're still iterating | Add `limit`/`take` to ad-hoc aggregation queries |
| Smaller table first in `join` | Fewer rows to build/match against | Put the lookup/dimension-style table first |

---
