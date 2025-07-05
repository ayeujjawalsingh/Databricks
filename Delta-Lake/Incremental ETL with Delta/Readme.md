# 📘 Incremental ETL Loads with Versioning in Delta Lake

This document explains how to perform **Incremental ETL (Extract, Transform, Load)** using **Delta Lake** with built-in support for **versioning and time travel**. This approach helps you build efficient, fault-tolerant data pipelines.

---

## 📌 What is Incremental ETL?

### 🔁 Full Load vs Incremental Load

| Type           | Description |
|----------------|-------------|
| **Full Load**  | Loads all data every time (slow and expensive). |
| **Incremental Load** | Loads only **new or changed** data (fast and efficient). |

---

## ✅ Why Use Incremental Loads?

- ⏱️ **Faster processing** — no need to reprocess old data.
- 💸 **Cost-effective** — less storage and compute usage.
- 📈 **Scalable** — works well with growing data volumes.
- 🛡️ **Reliable** — minimizes risk of duplication or data loss.

---

## 🧠 How Delta Lake Helps

Delta Lake is a storage layer that brings **ACID transactions**, **schema enforcement**, and **versioning** to data lakes.

Key Features for Incremental ETL:
- `MERGE INTO` support for UPSERTs (insert + update)
- Built-in **versioning** for every change
- **Time travel** to view or restore old versions
- Support for cleaning old files via `VACUUM`

---

## 🛠️ Step-by-Step Guide: Incremental Load with Delta Lake

### 1️⃣ Read Only the New Data

Use a timestamp or an incremental key to filter new records:

```python
new_data = spark.read.format("parquet").load("/raw/sales_data") \
    .filter("last_updated_at > '2025-07-04'")
```

✅ This reads only data after the last processed date.

---

### 2️⃣ Merge New Data into the Delta Table

Use the Delta Lake `MERGE` operation to perform **UPSERT** (update if exists, insert if not):

```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "/delta/sales")

delta_table.alias("target") \
  .merge(
    new_data.alias("source"),
    "target.order_id = source.order_id"
  ) \
  .whenMatchedUpdateAll() \
  .whenNotMatchedInsertAll() \
  .execute()
```

✅ This handles both updates and new inserts in a single command.

---

### 3️⃣ Understand Versioning in Delta Lake

Delta Lake automatically creates a **new version** of the table with every write operation (like insert, update, delete, or merge).

#### View History

```sql
DESCRIBE HISTORY delta.`/delta/sales`
```

✅ Shows all past versions and operations performed.

---

### 4️⃣ Time Travel — Go Back in Time!

You can read your table as it was at a previous point using version number or timestamp.

#### By Version:

```python
df = spark.read.format("delta") \
    .option("versionAsOf", 5) \
    .load("/delta/sales")
```

#### By Timestamp:

```python
df = spark.read.format("delta") \
    .option("timestampAsOf", "2025-07-04T00:00:00") \
    .load("/delta/sales")
```

✅ Useful for audits, debugging, or restoring bad data.

---

### 5️⃣ Clean Up Old Versions — VACUUM

Delta Lake retains old files for 7 days by default. To free up storage, run:

```sql
VACUUM delta.`/delta/sales` RETAIN 168 HOURS
```

🧹 This will delete obsolete files not required by any active version.

---

## 📦 Example Architecture

```
[Kafka / Batch File Ingestion]
              ↓
        [Spark Job]
              ↓
[Filter Only New Records using updated_at]
              ↓
[MERGE INTO Delta Table (Upsert)]
              ↓
[BI Tools / ML Models read latest data]
```

---

## ✅ Best Practices

* Always include a column like `updated_at` or `last_modified` in your source data.
* Use `MERGE` for combining new and existing data.
* Regularly monitor Delta table history using `DESCRIBE HISTORY`.
* Schedule `VACUUM` to clean old data and reduce storage cost.
* Use `versionAsOf` or `timestampAsOf` for rollback or historical analysis.

---

## 🧠 Key Concepts Summary

| Concept              | Description                                                 |
| -------------------- | ----------------------------------------------------------- |
| **Incremental Load** | Load only new or changed data using filter (like timestamp) |
| **MERGE INTO**       | Upsert data (update if exists, insert if not)               |
| **Versioning**       | Every change creates a new version of the table             |
| **Time Travel**      | Read the table as it was in the past                        |
| **VACUUM**           | Clean up old data files to save space                       |

---

## 📚 Related Tools

* **Apache Spark** — Engine to run ETL jobs
* **Databricks** — Unified platform for Delta Lake pipelines
* **AWS S3 / Azure ADLS** — Cloud storage for Delta tables
* **Power BI / Tableau** — For reading latest or versioned data
