# Medallion Architecture — Why It Exists

## The problem it solves

### Failure mode 1: one-shot ETL (transform in flight, keep nothing)
Read from source → clean/transform in memory → write final table.
Breaks down because **raw data was never preserved**:
- Finance flags "March 5 revenue looks wrong" → can't tell if the *source*
  sent garbage or *my logic* mangled it. No checkpoint to compare against.
- Found a bug (dedup edge case)? Fixing history needs a source re-pull —
  but the POS API only retains 30 days. History is gone forever.
- Every new use case = another extract hammering the operational system.

### Failure mode 2: the data swamp (dump raw, "figure it out later")
- Every analyst writes their own cleaning logic.
- Analyst A excludes test orders, Analyst B doesn't → two different
  "total revenue" numbers in the same meeting → trust in data dies.

## What each layer buys you

| Layer  | Role | Payoff |
|--------|------|--------|
| Bronze | Raw, as-arrived, append-only, load timestamps | Replay after bug fixes, audit trail, no source re-pulls |
| Silver | Dedup, type-cast, null handling, standardize — once | Everyone reuses one clean version; analysts stop disagreeing |
| Gold   | Star schemas, aggregates, blessed business definitions | Fast BI queries on small modeled tables; one version of truth |

**Debugging becomes binary search:** report wrong → check gold → silver →
bronze → source. You isolate *where* corruption entered.
