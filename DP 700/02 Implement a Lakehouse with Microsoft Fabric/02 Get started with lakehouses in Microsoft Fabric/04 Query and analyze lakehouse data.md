# Query and Analyze Lakehouse Data

---

## 1. Overview of Lakehouse Analysis Tools

After data is ingested and transformed inside a Microsoft Fabric lakehouse, it can be queried and analyzed using multiple complementary approaches[cite: 1]. You can choose the optimal interface based on workflow preferences and technical requirements—ranging from T-SQL for standard analytical queries to Spark for advanced data science or Power BI for interactive reporting[cite: 1].

---

## 2. Query Data Using the SQL Analytics Endpoint

The **SQL Analytics Endpoint** is a read-only engine automatically created with every lakehouse[cite: 1]. It enables data querying via ANSI T-SQL without modifying underlying Delta storage files[cite: 1].

### Primary Use Cases
* **Ad-hoc Data Investigation:** Quickly explore datasets to answer immediate business questions[cite: 1].
* **BI & External Tool Connectivity:** Connect tools like Power BI, Microsoft Excel, or Azure Data Studio directly to lakehouse tables[cite: 1].
* **Data Validation:** Verify the accuracy and shape of data post-ingestion or transformation[cite: 1].
* **Reusable Business Views:** Create database views to encapsulate complex join logic, apply business rules, or present curated datasets for reporting[cite: 1].

### Security & AI Capabilities
* **Granular Security:** Native support for **Row-Level Security (RLS)** and **Column-Level Security (CLS)** to restrict record and field access per user role[cite: 1].
* **Copilot for SQL:** Generates T-SQL code from natural language prompts, accelerating query authoring and learning[cite: 1].

---

## 3. Query Data Using Spark Notebooks

Notebooks provide a flexible, code-centric environment for exploratory data analysis, statistical modeling, and machine learning preparation[cite: 1].

### Spark SQL vs. PySpark

| Interface | Access Method | Key Characteristics & Strengths |
| :--- | :--- | :--- |
| **Spark SQL** | Standard SQL syntax executed in notebook cells or via `spark.sql()`[cite: 1]. | Ideal for traditional SQL queries against catalog tables (e.g., `SELECT * FROM schema.table`)[cite: 1]. |
| **PySpark** | Python API using the DataFrame object (e.g., `df.select()`, `df.filter()`)[cite: 1]. | High flexibility for complex procedural transformations and seamless integration with Python ML/statistical libraries[cite: 1]. |

### Notebook Use Cases
* **Exploratory Analysis:** Detect data patterns, anomalies, and statistical relationships[cite: 1].
* **Complex Transformations:** Express business logic that is difficult to implement in standard SQL[cite: 1].
* **Cross-Workspace Queries:** Query across multiple lakehouses using the 4-part namespace (`workspace.lakehouse.schema.table`)[cite: 1].
* **Copilot Assistance:** Natural language code generation and inline explanation for PySpark or Spark SQL[cite: 1].

---

## 4. Analyze and Visualize with Power BI

Power BI serves as the enterprise consumption and business intelligence layer in Fabric[cite: 1].

### Integration Patterns
1. **Direct SQL Endpoint Connection:** Analysts can query the read-only SQL endpoint directly to perform ad-hoc exploration in Power BI or Excel before building reports[cite: 1].
2. **Semantic Models:** Define explicit table relationships, calculated measures, and centralized business logic for report authors[cite: 1].

### Direct Lake Mode & AI Integration
* **Direct Lake Mode:** By default, Power BI reads Delta Lake Parquet files directly from OneLake without importing or duplicating data[cite: 1]. This delivers sub-second query performance while reflecting real-time lakehouse updates[cite: 1].
* **AI Grounding:** Well-structured semantic models (with defined relationships and business measures) serve as the foundation for **Copilot in Power BI**, enabling automated report generation and natural language Q&A[cite: 1].