
# Materialized Views and Stored Functions

## 1. The Problem: Aggregating Massive Streaming Tables

Eventhouse tables routinely hold millions or billions of rows from IoT sensors, application logs, and other streaming sources. Re-running an aggregation query across all of that history every single time someone opens a dashboard is slow and wastes compute.

**Materialized views** solve this by storing the *precomputed* aggregation result and updating it incrementally as new data arrives, instead of recalculating from scratch on every query.

---

## 2. How Materialized Views Stay Current

A materialized view is made of two pieces that get combined automatically at query time:

* **The materialized part** — precomputed aggregation results from data already processed.
* **The delta** — new data that's arrived since the last background update.

When you query the view, KQL merges both parts on the fly, so you always get current results — regardless of when the background materialization job last ran. A background process periodically folds the delta into the materialized part to keep things tidy. The net effect: **speed of a precomputed aggregate + freshness of live data**, without you doing anything extra.

> **Coming from Spark:** don't confuse this with `.cache()`-ing a DataFrame. A Spark cache is a frozen snapshot as of the moment you cached it — new rows written afterward won't show up until you refresh it yourself. A KQL materialized view auto-merges the delta at query time, so it's always current with no manual refresh step.

---

## 3. Creating a Materialized View

A materialized view wraps a `summarize` statement — that's the aggregation it keeps precomputed and current:

```kql
.create materialized-view TripsByVendor on table TaxiTrips
{
    TaxiTrips
    | summarize trips = count(), avg_fare = avg(fare_amount), total_revenue = sum(fare_amount)
    by vendor_id, pickup_date = format_datetime(pickup_datetime, "yyyy-MM-dd")
}
```

---

## 4. Querying a Materialized View

Once created, query it exactly like any other table — the usual operators (`where`, `project`, `sort`) all apply:

```kql
TripsByVendor
| where pickup_date >= ago(7d)
| project pickup_date, vendor_id, trips, avg_fare, total_revenue
| sort by pickup_date desc, total_revenue desc
```

---

## 5. Stored Functions: Reusable Query Logic

A **stored function** encapsulates a query — optionally with parameters — so you can call it by name instead of retyping the same filter/transform logic every time.

They're especially useful in Eventhouses because multiple people are usually writing queries against the same streaming data: defining common logic once as a function keeps everyone's filtering and calculations consistent, instead of each person quietly reimplementing (and possibly diverging on) the same logic.

> **Not the same thing as a Python UDF:** if you've read [10 PySpark Internals](../03%20Use%20Apache%20Spark%20in%20Microsoft%20Fabric/10%20PySpark%20Internals%20-%20Why%20It's%20Not%20Just%20Python.md), you'll remember Python UDFs are slow because Spark has to ship data out to a Python process row-by-row. A KQL stored function is nothing like that — it's closer to a saved, parameterized view. It gets inlined into the query pipeline at execution time, not invoked per row, so there's no serialization penalty to worry about.

### Create a function

```kql
.create-or-alter function trips_by_min_passenger_count(num_passengers:long)
{
    TaxiTrips
    | where passenger_count >= num_passengers
    | project trip_id, pickup_datetime
}
```

### Call the function

Call it just like a table, passing arguments in parentheses. This example finds 10 trips with at least 3 passengers:

```kql
trips_by_min_passenger_count(3)
| take 10
```

---

## 6. Quick Reference

| Feature | Materialized View | Stored Function |
| --- | --- | --- |
| Purpose | Precompute and cache aggregation results | Reuse/parameterize query logic |
| Storage | Physically stores materialized + delta data | Stores no data — just the query definition |
| Performance benefit | Avoids rescanning full history on every query | Avoids duplicated/inconsistent logic; not a storage optimization itself |
| Used like | A regular table | A table — with optional parameters in parentheses |

---
