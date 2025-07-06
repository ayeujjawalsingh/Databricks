# 🏗️ Delta Live Tables – Define Pipeline, Tables & Views

This guide explains how to **define a Delta Live Table pipeline**, and how to create **tables** and **views** using SQL or Python in Databricks.

---

## 📦 What is a Delta Live Table (DLT) Pipeline?

A **DLT pipeline** is a collection of steps to:
- Read raw data
- Clean, filter, transform the data
- Save it into **Delta tables** (or views)

> 💡 You define the steps in a notebook.  
> Databricks handles the **execution**, **scheduling**, **monitoring**, and **storage**.

---

## 🧰 Step 1: How to Define a DLT Pipeline

1. Go to **Workspace > Delta Live Tables > Create Pipeline**
2. Fill in:
   - **Pipeline Name** – e.g., `sales_pipeline`
   - **Notebook Path** – SQL or Python notebook with table/view definitions
   - **Target Schema** – Where the tables will be stored
   - **Storage Location** – Location for data and logs
   - **Pipeline Mode** – Development (for testing) or Production (for real use)

> 📌 Once saved, you can **Start** the pipeline and monitor it using the UI.

---

## 📊 Step 2: Define Tables using DLT

DLT tables are special Delta Tables managed by the pipeline.

### ✅ In SQL:

```sql
CREATE LIVE TABLE raw_customers
AS
SELECT * FROM cloud_files("/mnt/data/customers", "json");
```

```sql
CREATE LIVE TABLE clean_customers
AS
SELECT * FROM live.raw_customers
WHERE email IS NOT NULL;
```

* `CREATE LIVE TABLE`: tells DLT to create and manage this table
* `cloud_files`: uses Auto Loader to read files
* `live.`: refers to another live table in the same pipeline

---

### ✅ In Python (with Decorators):

```python
import dlt
from pyspark.sql.functions import col

@dlt.table
def raw_customers():
    return spark.read.format("cloudFiles") \
        .option("cloudFiles.format", "json") \
        .load("/mnt/data/customers")

@dlt.table
def clean_customers():
    return dlt.read("raw_customers").filter(col("email").isNotNull())
```

* `@dlt.table`: decorator to define a table
* `dlt.read()`: to read another table from the same pipeline

---

## 🧾 Step 3: Define Views using DLT

Views are **temporary** (not stored on disk), useful for intermediate logic or joining data.

### ✅ In SQL:

```sql
CREATE LIVE VIEW customer_summary
AS
SELECT city, COUNT(*) AS customer_count
FROM live.clean_customers
GROUP BY city;
```

### ✅ In Python:

```python
@dlt.view
def customer_summary():
    df = dlt.read("clean_customers")
    return df.groupBy("city").count().withColumnRenamed("count", "customer_count")
```

* Views are **not saved to Delta Lake**, just used during pipeline run
* Great for logical grouping or reusable logic

---

## 📌 Summary: Table vs View in DLT

| Type         | Stored on Disk? | Use Case                              |
| ------------ | --------------- | ------------------------------------- |
| `LIVE TABLE` | ✅ Yes           | Final outputs, stored tables          |
| `LIVE VIEW`  | ❌ No            | Intermediate steps or temporary logic |

---

## 📘 Complete Example Flow

```sql
CREATE LIVE TABLE raw_orders
AS SELECT * FROM cloud_files("/mnt/orders", "json");

CREATE LIVE TABLE clean_orders
AS SELECT * FROM live.raw_orders WHERE status IS NOT NULL;

CREATE LIVE VIEW order_summary
AS SELECT status, COUNT(*) AS total FROM live.clean_orders GROUP BY status;
```

* `raw_orders`: reads raw data from S3/ADLS
* `clean_orders`: filters invalid rows
* `order_summary`: temporary summary for reporting

---

## ✅ Best Practices

| ✅ Tip                     | 🔍 Why It’s Useful                            |
| ------------------------- | --------------------------------------------- |
| Use tables for outputs    | They’re saved in Delta and queryable later    |
| Use views for logic reuse | Keeps code clean, avoid duplication           |
| Keep naming clear         | e.g., `bronze_`, `silver_`, `gold_` prefixes  |
| Add data quality rules    | Use `CONSTRAINT` to validate data if needed   |
| Use `cloud_files()`       | For automatic file ingestion with Auto Loader |

---

## ✅ Summary

* 🔧 Define DLT pipelines using SQL or Python notebooks
* 🧱 Use `LIVE TABLE` for final outputs
* 🧾 Use `LIVE VIEW` for intermediate logic
* 🛠️ All steps are handled and monitored by the DLT engine

> Delta Live Tables helps you build **simple, reliable, production-grade pipelines** with **very little code** and **great visibility**.
