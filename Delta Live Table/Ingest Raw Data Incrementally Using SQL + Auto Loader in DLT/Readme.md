# ⚡ Ingest Raw Data Incrementally Using SQL + Auto Loader in Delta Live Tables

This guide explains how to **ingest raw data incrementally** using **SQL and Auto Loader** inside a **Delta Live Tables (DLT)** pipeline.

You’ll learn how to:
- Read new files automatically from a folder (e.g., S3 or ADLS)
- Use SQL to define ingestion logic
- Let DLT manage updates, retries, and metadata

---

## 🧠 What is Auto Loader?

**Auto Loader** is a feature in Databricks that:
- Detects **new files** automatically from a folder
- **Ingests only new data** (incrementally)
- Supports multiple formats: JSON, CSV, Parquet, etc.
- Is **optimized and scalable** for large datasets

> ✅ With Auto Loader, you don’t need to re-read the same files again and again.

---

## 📂 Typical Use Case

Let’s say:
- You get new `orders` data as **JSON files** every hour in an S3 bucket:  
  `/mnt/raw/orders/`
- You want to:
  - Read only new files
  - Store the raw data in a Delta table
  - Keep it updated automatically

---

## 🔧 Step-by-Step: Ingest with SQL + Auto Loader

### ✅ Step 1: Use `cloud_files()` to Read Raw Data

```sql
CREATE LIVE TABLE raw_orders
AS SELECT * FROM cloud_files(
  "/mnt/raw/orders",         -- Folder path
  "json"                     -- File format
);
```

### 🔍 What’s happening here?

* `cloud_files()` uses Auto Loader under the hood
* It tracks metadata and **automatically picks up only new files**
* This works even if thousands of files are in the folder

---

## 📥 How it Works Internally

* Auto Loader maintains a **checkpoint** (metadata file)
* It **remembers which files are already processed**
* When new files are added → they are ingested → checkpoint updated
* This makes the ingestion **incremental**

---

## 🧪 Add Data Quality Rule (Optional)

You can add a **constraint** to drop bad records:

```sql
CREATE LIVE TABLE raw_orders
  CONSTRAINT valid_order_id EXPECT (order_id IS NOT NULL) ON VIOLATION DROP ROW
AS SELECT * FROM cloud_files("/mnt/raw/orders", "json");
```

---

## 🧰 Optional Parameters (Advanced but Useful)

You can add options to customize Auto Loader:

```sql
CREATE LIVE TABLE raw_orders
AS SELECT * FROM cloud_files(
  "/mnt/raw/orders", 
  "json",
  map(
    "cloudFiles.inferColumnTypes", "true",
    "cloudFiles.schemaEvolutionMode", "rescue"
  )
);
```

| Option                | Description                           |
| --------------------- | ------------------------------------- |
| `inferColumnTypes`    | Auto-detect column types from data    |
| `schemaEvolutionMode` | Handles new columns using rescue mode |

---

## 🗃️ What Happens After Ingestion?

Once `raw_orders` is created:

* It becomes a **Delta table**
* You can use it in further steps like:

```sql
CREATE LIVE TABLE clean_orders
AS SELECT * FROM live.raw_orders WHERE status IS NOT NULL;
```

---

## 🧪 How to Test It?

1. Drop a few JSON files into `/mnt/raw/orders/`
2. Start the pipeline in **development mode**
3. Use this to query the table:

```sql
SELECT * FROM raw_orders;
```

4. Only **new files** will be picked up

---

## 💡 Best Practices

| ✅ Do This                    | 💬 Why                                      |
| ---------------------------- | ------------------------------------------- |
| Use `cloud_files()`          | To enable incremental, file-based ingestion |
| Use JSON or Parquet          | Most optimized for Auto Loader              |
| Separate folders per table   | Avoid mixing files from different schemas   |
| Use `CONSTRAINT` for quality | Catch bad or missing data early             |
| Use meaningful table names   | Like `raw_`, `bronze_` prefix for raw data  |

---

## 📌 Summary

| Step | What You Do                                             |
| ---- | ------------------------------------------------------- |
| 1️⃣  | Use `CREATE LIVE TABLE` in SQL                          |
| 2️⃣  | Use `cloud_files()` to point to raw data folder         |
| 3️⃣  | Add constraints to validate data (optional)             |
| 4️⃣  | Start the pipeline and watch Auto Loader pick new files |
| 5️⃣  | Use the table in your next ETL step                     |

---

## ✅ Final Example

```sql
CREATE LIVE TABLE raw_customers
AS SELECT * FROM cloud_files("/mnt/data/customers", "json");

CREATE LIVE TABLE clean_customers
AS SELECT * FROM live.raw_customers WHERE email IS NOT NULL;
```

> DLT + Auto Loader lets you build **real-time, scalable, and low-code ingestion pipelines** using just SQL.
