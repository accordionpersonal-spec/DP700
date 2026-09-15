
# Work with Data Using Spark SQL


## 1. Overview of Spark SQL & The Spark Catalog

**Spark SQL** is a core module in Apache Spark that allows data analysts and developers to query structured data using standard ANSI SQL expressions alongside DataFrames.

### The Spark Catalog
The **Spark Catalog** acts as a central metastore for relational database objects—such as tables, views, and databases. It allows Spark to seamlessly link code written in PySpark, Scala, or Java with SQL queries.

---

## 2. Temporary Views vs. Catalog Tables

When working with DataFrames in Spark SQL, you can register data as either temporary views or persistent tables.

### Temporary Views
A **Temporary View** acts as an in-memory SQL interface over a DataFrame. 
* **Scope:** Temporary views exist only within the current Spark session and are automatically deleted when the session ends.
* **Storage:** No underlying files or metadata are saved to persistent storage.

```python
# Create or replace a temporary view from an existing DataFrame
df.createOrReplaceTempView("products_view")

```

---

### Managed vs. External Tables

| Feature | Managed Tables | External Tables |
| --- | --- | --- |
| **Data Storage Location** | Stored inside the default catalog directory (`Tables/` directory in a Lakehouse). | Stored in a custom or external file path (e.g., `Files/` storage area). |
| **Creation Method** | `df.write.saveAsTable("table_name")` or `spark.catalog.createTable()` | `spark.catalog.createExternalTable()` |
| **Deletion Behavior (`DROP TABLE`)** | **Deletes both** table metadata in the catalog and raw data files on disk. | **Deletes metadata only**; original underlying data files remain intact. |

#### Example: Saving a DataFrame as a Delta Table

```python
# Save a DataFrame as a managed Delta table in the Spark catalog
df.write.format("delta").saveAsTable("products")

```

> **Note on Delta Lake:** The default and preferred storage format in modern engines like Microsoft Fabric and Databricks is **Delta Lake** (`format("delta")`). Delta tables provide relational database capabilities, including ACID transactions, data versioning (time travel), and streaming support.

> **Tip:** Just like Parquet files, Delta tables can be physically partitioned (e.g., `.partitionBy("Category")`) to drastically improve SQL query filtering performance through partition pruning.

---

## 3. Querying Data using the Spark SQL API

You can write native SQL queries inside PySpark code using `spark.sql()`. This function executes the query against catalog tables/views and returns the result as a standard **Spark DataFrame**.

```python
# Execute an ANSI SQL query against the 'products' table using PySpark
bikes_df = spark.sql("""
    SELECT 
        ProductID, 
        ProductName, 
        ListPrice 
    FROM products 
    WHERE Category IN ('Mountain Bikes', 'Road Bikes')
""")

# Display top results in notebook environment
display(bikes_df)

```

---

## 4. Executing Pure SQL Cells (`%%sql` Magic)

In interactive notebook environments (e.g., Microsoft Fabric, Synapse, Databricks), you can write pure SQL without wrapping it in Python code by using the `%%sql` magic directive at the top of the cell.

```sql
%%sql
-- Query catalog tables directly in a dedicated SQL cell
SELECT 
    Category, 
    COUNT(ProductID) AS ProductCount
FROM products
GROUP BY Category
ORDER BY Category;

```

```

> **Rule:** `%%sql` must be the very first line of the cell — nothing else, not even a comment, above it.

---

## 5. What Really Happens When You Run `%%sql`

`%%sql` is **not** a separate, lighter-weight SQL engine — it's just a notebook front-end shorthand that tells the kernel "treat everything below this line as raw SQL." That SQL is still routed through the exact same Spark SQL engine as `spark.sql(...)`: it queries the same Spark Catalog, sees the same tables/views, and executes on the same cluster.

The only real difference is output handling — `%%sql` auto-executes and renders the result as a table in the cell, without giving you a DataFrame variable to keep chaining in Python.

```python
%%sql
SELECT * FROM products
```

is functionally equivalent to:

```python
spark.sql("SELECT * FROM products").display()
```

### The Full Execution Pipeline

Whether the query arrives as a `%%sql` cell, a `spark.sql("...")` call, or a chained `.filter().select()` DataFrame expression, it funnels through the same Catalyst → Tungsten pipeline:

1. **Parsing** — Spark parses the SQL text (or DataFrame call chain) into an unresolved logical plan.
2. **Analysis** — The Catalyst optimizer resolves table names, column names, and types against the Spark Catalog.
3. **Logical plan optimization** — Catalyst applies rule-based optimizations (predicate pushdown, column pruning, constant folding, etc.).
4. **Physical planning** — Catalyst chooses physical execution strategies (which join algorithm, whether to broadcast a table, etc.) and generates a physical plan.
5. **Code generation** — Spark generates JVM bytecode for the plan (whole-stage code generation) for efficient execution.
6. **Job → Stages → Tasks** — The physical plan is broken into a DAG of stages, split into tasks.
7. **Cluster execution** — The driver node sends tasks to executors on worker nodes, each processing a partition of data in parallel.
8. **Result collection** — Results are shuffled/aggregated back and returned to the notebook for display.

> **Takeaway:** SQL, the DataFrame API, and even Scala/Java code all funnel into one unified Catalyst → Tungsten execution engine — performance is identical no matter which "face" you use to write the query.

---