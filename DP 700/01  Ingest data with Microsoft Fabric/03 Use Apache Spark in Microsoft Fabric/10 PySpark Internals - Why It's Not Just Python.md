
# PySpark Internals: Why It's Not "Just Python"

## 1. Spark's True Engine: Scala on the JVM

Apache Spark itself is written in **Scala** and runs on the **JVM (Java Virtual Machine)**. The actual execution engine — the Catalyst optimizer, Tungsten, the DAG scheduler, task execution — is all JVM code.

**PySpark** is a Python wrapper *around* that JVM engine — it doesn't reimplement Spark in Python.

---

## 2. What Your Python Code Actually Does

When you write:

```python
df = spark.read.parquet("path")
df.filter(df.Category == "Mountain Bikes")
```

you are **not** running Python logic that processes your data. You're writing Python code that *builds a plan* and sends instructions to the real Spark engine (JVM) to execute.

### The Py4J Bridge

PySpark uses **Py4J**, a bridge library that translates your Python calls into JVM calls. The flow looks like this:

```text
Your Python code (PySpark)
        |  (Py4J bridge)
        v
JVM Spark Driver
        |
        v
Catalyst optimizer -> physical plan
        |
        v
Tasks sent to Executors (JVM, on worker nodes)
        |
        v
Results come back to Python
```

---

## 3. Why This Matters in Practice

* **Standard DataFrame/SQL operations** — `.select()`, `.filter()`, `.groupBy()`, SQL queries — run **entirely inside the JVM**. There's no performance penalty for writing them in Python: you're just describing a plan, and execution happens in Scala/JVM land regardless of which language authored the code.
* **Python UDFs (User-Defined Functions)** are the one place the penalty shows up. To run a Python UDF, Spark has to:
  1. Serialize data **out of** the JVM.
  2. Spin up a Python process on each executor.
  3. Run your Python logic row-by-row.
  4. Serialize the results **back into** the JVM.

  That round trip is genuinely slow — which is why Python UDFs are discouraged whenever a built-in Spark SQL function can do the same job.

> **One-line takeaway:** PySpark doesn't run your logic in Python — it uses Python syntax to control a Scala/JVM engine. As long as you stick to DataFrame/SQL operations (not custom Python UDFs), you get full native Spark performance, not Python speed.

See also: [05 Work with data using Spark SQL](05%20Work%20with%20data%20using%20Spark%20SQL.md) — section 5 walks through how `%%sql` and `spark.sql()` both funnel into this same JVM-side Catalyst engine.

---
