
---

## 1. Spark Architecture: DataFrames vs. RDDs

### Resilient Distributed Datasets (RDDs)

At its core, Apache Spark stores and processes data across multiple computers using **RDDs (Resilient Distributed Datasets)**.

* **Resilient:** Automatically recovers from hardware or node failures.
* **Distributed:** Data is split into smaller chunks (partitions) across multiple machines.
* **Dataset:** The underlying collection of data objects.

**The Problem with RDDs:** Working directly with RDDs requires low-level, procedural code. Spark doesn't know the internal structure (schema) of the data inside an RDD, making automatic query optimization difficult.

### Spark DataFrames

A **DataFrame** is a higher-level abstraction built on top of RDDs, provided by the `pyspark.sql` module.

* It organizes data into **named columns**, exactly like a table in a relational database or a Pandas DataFrame in Python.
* **Why it matters:** Because DataFrames have a defined **schema** (column names and data types), Spark's internal engine (**Catalyst Optimizer**) can rewrite, reorder, and optimize your operations for maximum execution speed across a cluster.

> **Scala/Java Note:** Scala and Java also support **Datasets** (strongly typed DataFrames, e.g., `Dataset<Product>`). PySpark relies on DataFrames because Python is dynamically typed.

---

## 2. Notebook Magic Commands (`%%pyspark` & `%%spark`)

In interactive notebook environments (like Azure Synapse, Microsoft Fabric, or Databricks), a single notebook can execute code in multiple programming languages.

* **Magics** are special directives placed at the very top line of a code cell (prefixed by `%%`) to instruct the execution engine which language kernel to use for that cell.

```python
# Tells the notebook engine to execute this cell using PySpark (Python)
%%pyspark
df = spark.read.load('Files/data/products.csv', format='csv', header=True)
display(df.limit(10))

```

```scala
// Tells the notebook engine to execute this cell using Scala
%%spark
val df = spark.read.format("csv").option("header", "true").load("Files/data/products.csv")
display(df.limit(10))

```

---

## 3. Loading Data: Inferring vs. Defining Schemas

When loading structured files (such as CSVs), Spark must determine the data type for every column (e.g., String, Integer, Float).

### Method A: Inferring the Schema automatically

Spark reads the entire file or a portion of it to guess data types based on the values.

```python
df = spark.read.load(
    'Files/data/products.csv',
    format='csv',
    header=True  # Uses the first row for column names
)

```

* **Pros:** Convenient for fast exploration.
* **Cons:** Slower on large datasets (Spark must scan raw data to guess types) and can misinterpret data types (e.g., treating zip codes like `07001` as integers, dropping the leading zero).

### Method B: Specifying an Explicit Schema

You manually define column names and exact data types using `StructType` and `StructField`.

```python
from pyspark.sql.types import StructType, StructField, IntegerType, StringType, FloatType

# Explicitly declare schema structure
productSchema = StructType([
    StructField("ProductID", IntegerType(), True),
    StructField("ProductName", StringType(), True),
    StructField("Category", StringType(), True),
    StructField("ListPrice", FloatType(), True)
])

# Load file using the defined schema
df = spark.read.load(
    'Files/data/product-data.csv',
    format='csv',
    schema=productSchema,
    header=False  # Data file has no header row
)

```

### Key Schema Benefits

1. **Performance Boost:** Spark skips reading the entire file just to guess types.
2. **Data Integrity:** Ensures data fits strictly expected shapes and data types.

---

## 4. DataFrame Operations

Spark DataFrames are **immutable**. When you apply operations like `select()`, `where()`, or `groupBy()`, Spark does not modify the original DataFrame—it creates a **new DataFrame object**.

### Column Selection

Extract a subset of columns:

```python
# Standard syntax using select()
pricelist_df = df.select("ProductID", "ListPrice")

# Shorthand column-indexing syntax
pricelist_df = df["ProductID", "ListPrice"]

```

### Chaining Transformations (Filtering)

Multiple transformation methods can be chained sequentially:

```python
# Select specific columns and apply logical filters
bikes_df = df.select("ProductName", "Category", "ListPrice") \
             .where((df["Category"] == "Mountain Bikes") | (df["Category"] == "Road Bikes"))

display(bikes_df)

```

* Bitwise operators like `|` (OR) and `&` (AND) are used for logical checks.
* Wrap individual condition evaluations inside parenthetical groupings `(...)` to ensure standard precedence rules.

### Grouping and Aggregation

Summarize data across categories:

```python
# Group by Category column and count occurrences
counts_df = df.select("ProductID", "Category") \
              .groupBy("Category") \
              .count()

display(counts_df)

```

---

## 5. Storage Formats: Writing to Apache Parquet

Once data is transformed, write the result back into storage (such as a Lakehouse or cloud storage bucket).

```python
bikes_df.write.mode("overwrite").parquet('Files/product_data/bikes.parquet')

```

### Why Parquet Over CSV?

| Metric | CSV File Format | Apache Parquet |
| --- | --- | --- |
| **Structure** | Row-oriented | Column-oriented |
| **Storage Efficiency** | Uncompressed text (large size) | Highly compressed binary (small size) |
| **Performance** | Must scan full row values | Scans only requested columns |
| **Schema Support** | No built-in schema | Embedded schema and data types |

---

## 6. Partitioning Data for Query Performance

### Concept of Partitioning

Partitioning physically splits data files across subdirectories based on distinct values in one or more columns.

```python
# Save DataFrame partitioned by Category column values
bikes_df.write.partitionBy("Category").mode("overwrite").parquet("Files/bike_data")

```

### Resulting Lakehouse Folder Structure:

```text
Files/bike_data/
├── Category=Mountain Bikes/
│   ├── part-0000.parquet
│   └── part-0001.parquet
└── Category=Road Bikes/
    ├── part-0000.parquet
    └── part-0001.parquet

```

### Partition Pruning Optimization

When querying partitioned data, Spark reads **only** the relevant subdirectories and ignores the rest (called **Partition Pruning**).

```python
# Load only the Road Bikes subfolder
road_bikes_df = spark.read.parquet('Files/bike_data/Category=Road Bikes')
display(road_bikes_df.limit(5))

```

> **Important Detail:** When reading directly from a specific partition folder, the partitioned column (`Category`) is omitted from the resulting DataFrame structure, as every record inside that path naturally shares that value.

---