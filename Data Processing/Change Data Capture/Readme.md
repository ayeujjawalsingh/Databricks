# 📘 Change Data Capture (CDC) using Delta Lake CDF in Databricks

This guide explains how to track, read, and apply **only changed data (INSERT/UPDATE/DELETE)** using **Delta Lake Change Data Feed (CDF)**. It covers:

- ✅ What is CDC and why it matters
- ✅ How Delta CDF works
- ✅ How to process the CDC feed in batch or streaming
- ✅ Real examples and use cases

---

## 🔍 What is Change Data Capture (CDC)?

**Change Data Capture (CDC)** is a way to track and apply only the **changes** (inserts, updates, deletes) made to a table — without reprocessing the entire data.

---

### 📦 Why is CDC useful?

| Feature        | Benefit                                                       |
|----------------|---------------------------------------------------------------|
| ⏱️ Faster       | Reads only the rows that have changed                        |
| 💰 Cost-effective | Saves compute resources and storage                         |
| 🔁 Incremental | Keeps downstream tables and reports updated in real-time      |
| ✅ Scalable     | Works well with large datasets and layered architectures      |

---

## 🧠 What is Delta Lake Change Data Feed (CDF)?

Delta Lake **CDF** is a feature that enables **CDC natively** on Delta tables. When enabled, it captures all changes made to a table (insert/update/delete) and allows you to query only the **changed rows**.

---

## 🧱 Architecture Flow

```

Source Delta Table (CDF Enabled)
⬇️
Read Only Changed Rows (by Version/Time)
⬇️
Detect Change Type (\_change\_type)
⬇️
Apply to Target Table (using MERGE or Logic)

```

---

## 🛠️ Step-by-Step: Enable & Use CDC with Delta CDF

---

### ✅ Step 1: Enable Change Data Feed on the Table

```sql
ALTER TABLE <your_table>
SET TBLPROPERTIES (
  delta.enableChangeDataFeed = true
);
````

> 🔹 Do this before you write any changes — only future changes will be tracked.

---

### ✅ Step 2: Write Some Changes (Insert/Update/Delete)

```sql
-- Insert
INSERT INTO your_table VALUES (101, 'Amit', 'Pune');

-- Update
UPDATE your_table SET city = 'Mumbai' WHERE customer_id = 101;

-- Delete
DELETE FROM your_table WHERE customer_id = 101;
```

---

### ✅ Step 3: Read the Changed Data

#### 🧾 Batch Mode

```python
cdc_df = spark.read.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", 5) \
    .table("your_table")
```

> Use `startingVersion` or `startingTimestamp` to define how far back you want to track changes.

---

### ✅ Step 4: Process CDC Feed Using `MERGE INTO`

```python
cdc_df.createOrReplaceTempView("cdc_data")

spark.sql("""
MERGE INTO target_table AS tgt
USING cdc_data AS src
ON tgt.customer_id = src.customer_id

WHEN MATCHED AND src._change_type = 'update' THEN
  UPDATE SET *

WHEN MATCHED AND src._change_type = 'delete' THEN
  DELETE

WHEN NOT MATCHED AND src._change_type = 'insert' THEN
  INSERT *
""")
```

This keeps your **target table updated** with minimal cost and effort.

---

### ✅ Step 5 (Optional): Stream CDC Feed in Real-Time

```python
stream_df = spark.readStream.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", 20) \
    .table("your_table")

# You can apply the same MERGE logic in foreachBatch()
```

---

## 🧾 Sample Output from CDF

| customer\_id | name | city   | \_change\_type | \_commit\_version |
| ------------ | ---- | ------ | -------------- | ----------------- |
| 101          | Amit | Pune   | insert         | 5                 |
| 101          | Amit | Mumbai | update         | 6                 |
| 101          | NULL | NULL   | delete         | 7                 |

---

## 📊 Metadata Columns Added by CDF

| Column              | Description                             |
| ------------------- | --------------------------------------- |
| `_change_type`      | `insert`, `update`, or `delete`         |
| `_commit_version`   | Delta version where the change occurred |
| `_commit_timestamp` | When the change was committed           |

---

## 📌 Use Cases of CDC + Delta CDF

| Use Case                                           | Purpose                                  |
| -------------------------------------------------- | ---------------------------------------- |
| ⏩ Incremental ETL Pipelines                        | Load only changed rows                   |
| 📊 Real-Time Dashboards                            | Keep visualizations up-to-date           |
| 🔁 Sync to External Warehouses                     | Replicate changes to Redshift, Snowflake |
| 🧾 Auditing / Compliance                           | Know exactly what changed and when       |
| 🪄 Medallion Architecture (Bronze → Silver → Gold) | Efficient change propagation             |

---

## ⚠️ Things to Keep in Mind

* You must enable `delta.enableChangeDataFeed = true` before it works.
* You must define `startingVersion` or `startingTimestamp` to query.
* Deleted rows will have `NULL` values (except for primary key).
* CDF is available **only on Delta tables**, not Parquet or external formats.

---

## ✅ Summary

| Topic           | Summary                                   |
| --------------- | ----------------------------------------- |
| What is CDC?    | Capturing only changed data rows          |
| What is CDF?    | Delta Lake feature to track those changes |
| How to Use?     | Enable → Modify → Read → Apply            |
| Output Includes | `_change_type`, `_commit_version`, etc.   |
| Supported Modes | Batch and Streaming                       |
