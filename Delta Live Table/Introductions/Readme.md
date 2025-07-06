# 📘 Delta Live Tables (DLT)

Delta Live Tables (DLT) is a framework by **Databricks** that makes it easy to create **ETL pipelines** using **SQL or Python**. It handles table creation, scheduling, monitoring, and error handling automatically.

---

## 📌 What is Delta Live Tables?

> Delta Live Tables lets you write **data pipelines** without managing complex Spark jobs or retries.  
You define **what to do**, and DLT manages **how and when** to do it.

---

## ✅ Why Use DLT?

| Feature | What It Means |
|--------|----------------|
| 🚀 Easy to use | Use simple SQL or Python functions |
| 🔁 Automatic scheduling | DLT runs pipelines at intervals or on-demand |
| 🔍 Monitors itself | Provides logs, errors, lineage graph |
| 🧠 Understands dependencies | Runs steps in correct order |
| 💾 Stores data as Delta Tables | Data stored in reliable Delta format |
| 🧹 Cleans bad data | Add simple rules to drop or log invalid rows |
| ⚡ Batch & Streaming | Works with both real-time and batch data |

---

## 📂 Basic Components

### 1. `CREATE LIVE TABLE`
Used to define a table in the pipeline.

```sql
CREATE LIVE TABLE raw_orders
AS SELECT * FROM cloud_files("/mnt/orders", "json");
```

### 2. `live.<table_name>`

Used to reference another live table.

```sql
CREATE LIVE TABLE clean_orders
AS SELECT * FROM live.raw_orders WHERE order_status IS NOT NULL;
```

### 3. In Python

```python
import dlt

@dlt.table
def raw_orders():
    return spark.read.format("cloudFiles") \
      .option("cloudFiles.format", "json") \
      .load("/mnt/orders")

@dlt.table
def clean_orders():
    return dlt.read("raw_orders").where("order_status IS NOT NULL")
```

---

## 🔍 DLT UI Walkthrough

| Section                    | Purpose                                         |
| -------------------------- | ----------------------------------------------- |
| 🔗 **Graph View**          | Shows data flow between tables                  |
| 📝 **Run Logs**            | Shows run history, status, logs, and time       |
| 🛡️ **Data Quality Rules** | Add constraints to drop bad rows                |
| 🧬 **Schema Evolution**    | Automatically tracks and handles schema changes |
| 📊 **Monitoring Metrics**  | Shows rows processed, bytes read, duration      |

---

## ⚙️ Modes of DLT Pipeline

| Mode                   | Description                              |
| ---------------------- | ---------------------------------------- |
| ✅ **Development Mode** | For testing pipelines quickly            |
| 🔒 **Production Mode** | For stable, regular production workloads |

---

## 🗃️ Table Types

| Table Type             | Description                       |
| ---------------------- | --------------------------------- |
| `LIVE TABLE`           | Regular managed table             |
| `STREAMING LIVE TABLE` | Table with streaming data support |

---

## 🧹 Example: Data Cleaning with Rules

```sql
CREATE LIVE TABLE valid_orders
  CONSTRAINT valid_id EXPECT (order_id IS NOT NULL) ON VIOLATION DROP ROW
AS SELECT * FROM live.clean_orders;
```

* If `order_id` is empty, row is **dropped** and logged.

---

## 📦 Where Data is Stored

* DLT saves all tables as **Delta tables**
* Can be saved as **managed** (by Databricks) or **external** (your path)
* Can integrate with **Unity Catalog** for data governance

---

## 📊 Example Data Flow Diagram

```
            Source Files (S3, ADLS, etc.)
                        │
          ┌────────────▼────────────┐
          │ raw_orders (DLT Table)  │ ← reads JSON
          └────────────┬────────────┘
                        │
          ┌────────────▼────────────┐
          │ clean_orders (DLT)      │ ← filters bad rows
          └────────────┬────────────┘
                        │
          ┌────────────▼────────────┐
          │ gold_orders (DLT)       │ ← does aggregation
          └─────────────────────────┘
```

---

## ✅ Summary

* Delta Live Tables let you create pipelines **faster and safer**
* You write **what to do**, DLT handles **how and when**
* Works with both **batch and streaming**
* Helps you monitor, retry, and clean data easily

---

## 🔗 Useful Terms

| Term                    | Meaning                               |
| ----------------------- | ------------------------------------- |
| `cloud_files`           | Autoloader function for reading files |
| `live.<table>`          | Reference to another DLT table        |
| `@dlt.table`            | Python decorator for defining table   |
| `CONSTRAINT ... EXPECT` | Data quality rule                     |
