# Medallion Architecture — In Simple Words
*DP-700 · Unit: Describe medallion architecture (plain-English reference)*

## The setup (one example we follow throughout)
An online store gets data from three places:
- the website's **order database**
- a **CRM** that exports customer details
- two **ad platforms** sending spend reports

Different formats, different speeds, different column names for the
same thing.

## The core problem
The systems *creating* the data are built to **run** the business,
not to **answer questions** about it:
- The website DB is tuned so saving one order takes milliseconds —
  each order is split across many small tables. Great for the app,
  awful for "sales by city last month."
- These systems **overwrite themselves**: a customer updates their
  phone number → the old value is gone. Analytics often needs history.

→ So you copy data out and clean it **in stages**, keeping the
  original safe the whole time.

## Bronze — keep the original, touch nothing
- Everything lands **exactly as it arrived**: the website's JSON, the
  CRM's CSV, the ad platforms' odd exports — untouched, just stamped
  with an arrival time.
- **Why store mess? Insurance.** Discover next month that your cleaning
  logic had a bug and two weeks of numbers are wrong? Fix the bug and
  re-run from your own bronze copy. Without bronze, you're begging the
  ad platform for old data they may have already deleted.
- **Audience: data engineers only.** Nobody else shops in the
  receiving dock.

## Silver — clean once, so everyone stops cleaning separately
What happens here:
- drop duplicate orders (the website sometimes sends the same event twice)
- standardize values ("IN" vs "India" → pick one)
- unify date formats, handle nulls
- merge CRM customers with orders into one integrated customer table

**The payoff:** analysts query *this* instead of each writing their own
cleanup code. Without silver, analyst A excludes test orders, analyst B
forgets to — and two different "total revenue" numbers land in the
same meeting.

**Audience: analysts and data scientists.**

## Gold — reshape around the questions
- Silver is clean but still **shaped like the sources** — wide technical
  tables, columns like `cust_id_src2`.
- Gold rearranges data around what the business asks: one central table
  of numbers (sales amount, quantity) surrounded by small descriptive
  tables (customer, product, date, region). That's a **star schema**.
- Result: "sales by product by month" = a tiny two-join query,
  dashboards load fast, and "revenue" is defined **once** in the model —
  not nine different ways in nine notebooks.
- **Audience: business users and BI tools.**

## Why the audience split matters
If everyone shares one set of tables:
- an engineer renames a column → an analyst's dashboard dies overnight
- an analyst "fixes" values in place → a model quietly trains on
  changed data

Layers are **lanes**: each group reads from a stable layer that
someone else owns and maintains.

## "Adapt to your needs" (exam nuance)
Three layers is the **default, not a law**:
- Files arrive zipped or need a quarantine check? → add a **landing
  zone before bronze**.
- One team wants its own aggregates without touching the shared model?
  → add a **domain layer after gold**.

The rule: any layer must have a **clear purpose and a clear audience** —
never "because the diagram had three."

## One-line memory hooks
- **Bronze** = receiving dock (raw, untouched, insurance)
- **Silver** = prep station (cleaned, merged, one version)
- **Gold** = plated dish (modeled for business questions)