
# Visualize Data in a Spark Notebook


## 1. Overview of Data Visualization in Spark

Visualizing query results as charts is essential for intuitive data analysis. In Spark notebooks (such as Microsoft Fabric, Azure Synapse, or Databricks), you have two main approaches for visualizing data:
1. **Built-in UI Charts:** Quick, low-code interactive visualizations rendered directly below code cells.
2. **Code-based Graphics Libraries:** Advanced, highly customized plots generated using Python visualization libraries like Matplotlib or Seaborn.

---

## 2. Using Built-in Notebook Charts

When you execute a DataFrame query or a `%%sql` cell in a notebook environment, the results are displayed underneath the cell.

* **Default View:** Displays data as a standard tabular grid.
* **Chart View:** Toggle the view from **Table** to **Chart** directly within the cell output interface.
* **Customization:** Use built-in property menus to adjust chart types (Bar, Line, Pie), X/Y axes, and aggregation types without writing additional code.

> **When to use:** Ideal for rapid data exploration, sanity checks, and quick inline summaries during exploratory analysis.

---

## 3. Advanced Visualizations Using Code Libraries

For granular control over formatting, layout, colors, and styling, you can use Python's ecosystem of visualization packages.

### Converting Spark DataFrames to Pandas
Most Python plotting libraries (such as Matplotlib and Seaborn) operate on single-node data structures rather than distributed Spark DataFrames. 
* Use the `.toPandas()` method to convert a Spark DataFrame into a local **Pandas DataFrame** before plotting.

---

### Example: Creating a Bar Chart with Matplotlib

The following PySpark example aggregates data using Spark SQL, converts the output to Pandas, and uses **Matplotlib** to build a formatted bar chart:

```python
from matplotlib import pyplot as plt

# 1. Execute SQL query and convert the distributed Spark DataFrame to a local Pandas DataFrame
data = spark.sql("""
    SELECT 
        Category, 
        COUNT(ProductID) AS ProductCount
    FROM products
    GROUP BY Category
    ORDER BY Category
""").toPandas()

# 2. Clear any previous plot figure
plt.clf()

# 3. Define figure canvas size (width=12, height=8)
fig = plt.figure(figsize=(12, 8))

# 4. Create a bar plot
plt.bar(x=data['Category'], height=data['ProductCount'], color='orange')

# 5. Apply custom formatting, labels, gridlines, and title
plt.title('Product Counts by Category')
plt.xlabel('Category')
plt.ylabel('Products')
plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
plt.xticks(rotation=70)  # Rotate category labels for readability

# 6. Render the plot inside the notebook cell
plt.show()

```

---

## 4. Key Python Graphics Libraries

| Library | Primary Use Case | Key Features |
| --- | --- | --- |
| **Matplotlib** | Base visualization library | Low-level control over every visual element (axes, fonts, grids, figure size). |
| **Seaborn** | Statistical data visualization | Built on top of Matplotlib with high-level functions for complex statistical plots (heatmaps, boxplots, pairplots). |
| **Plotly / Bokeh** | Interactive web visualizations | Hover tooltips, zooming, panning, and dynamic dashboard integration. |

```

```