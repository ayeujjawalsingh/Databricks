# 📘 Complete Guide to Change Data Capture (CDC) using Delta Lake CDF in Databricks

This document provides a complete end-to-end understanding of **Change Data Capture (CDC)** in Databricks using **Delta Lake’s Change Data Feed (CDF)**.

You’ll learn:

- ✅ What is CDC, and why it matters
- ✅ How Delta CDF works internally
- ✅ How to process changes (insert, update, delete)
- ✅ How to propagate DELETEs downstream
- ✅ Batch and Streaming examples
- ✅ Best practices and common gotchas

---

## 🔍 What is Change Data Capture (CDC)?

**CDC (Change Data Capture)** is a technique to track changes made to data — such as:

- ➕ New rows added (INSERT)
- 📝 Existing rows modified (UPDATE)
- ❌ Rows removed (DELETE)

Instead of reprocessing the **entire table**, CDC allows you to process **only the rows that changed** — making data pipelines **faster, cheaper, and scalable**.

---

## 📦 Why Use CDC?

| ✅ Benefit                  | 📌 Description                                               |
|----------------------------|--------------------------------------------------------------|
| Incremental processing     | Only changed rows are processed                             |
| Reduced compute cost       | Avoids full table scans                                     |
| Real-time sync             | Keeps downstream systems up to date quickly                 |
| Clean layering             | Efficient Bronze → Silver → Gold data movement              |
| Ideal for streaming        | Supports low-latency pipelines                              |

---

## 🧠 What is Delta Change Data Feed (CDF)?

Delta Lake’s **Change Data Feed (CDF)** is a **built-in CDC feature** that lets you **query only the changed rows** (insert/update/delete) from a Delta table.

It leverages Delta’s internal **transaction log (Delta log)** and exposes changes as a queryable DataFrame with metadata.

---

## 🔁 What Does CDF Capture?

| Operation | Description            | Captured in CDF? | Notes                         |
|-----------|------------------------|------------------|-------------------------------|
| INSERT    | New row added          | ✅ Yes           | Captured as new data          |
| UPDATE    | Row values modified    | ✅ Yes           | Before & after combined       |
| DELETE    | Row removed            | ✅ Yes           | Columns null, key remains     |

---

## 🛠️ Step-by-Step: Enable and Use Delta CDF for CDC

---

### ✅ Step 1: Enable Change Feed on the Source Table

```sql
ALTER TABLE customer_source
SET TBLPROPERTIES (
  delta.enableChangeDataFeed = true
);
```

> 🔸 You must enable CDF **before changes occur**. It only tracks future changes.

---

### ✅ Step 2: Perform Some Changes (Insert / Update / Delete)

```sql
-- Insert
INSERT INTO customer_source VALUES (101, 'Amit', 'Pune');

-- Update
UPDATE customer_source SET city = 'Mumbai' WHERE customer_id = 101;

-- Delete
DELETE FROM customer_source WHERE customer_id = 101;
```

---

### ✅ Step 3: Read the Changed Data using CDF

#### 📄 Batch Mode

```python
cdf_df = spark.read.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", 5) \
    .table("customer_source")
```

You can also use:

```python
.option("startingTimestamp", "2025-07-01T00:00:00.000Z")
```

---

### ✅ Step 4: Understand the CDC Output

| customer\_id | name | city   | \_change\_type | \_commit\_version |
| ------------ | ---- | ------ | -------------- | ----------------- |
| 101          | Amit | Pune   | insert         | 5                 |
| 101          | Amit | Mumbai | update         | 6                 |
| 101          | NULL | NULL   | delete         | 7                 |

> 🔸 Deleted rows will have `NULL` values except for key columns.

---

### ✅ Step 5: Process CDC Feed using `MERGE INTO`

```python
cdf_df.createOrReplaceTempView("cdc_data")

spark.sql("""
MERGE INTO customer_target AS tgt
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

✅ This handles **insert, update, and delete** in a single query.

---

### ✅ Step 6: (Optional) Use CDF with Streaming

```python
stream_df = spark.readStream.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", 20) \
    .table("customer_source")

stream_df.writeStream.foreachBatch(process_batch_fn).start()
```

Inside `process_batch_fn(df, batchId)`, apply the same `MERGE` logic.

---

## 🔁 CDF and Propagating DELETEs

---

### ✅ How are DELETEs Represented in CDF?

When a row is deleted from the source:

* `_change_type = 'delete'`
* Non-key columns = `NULL`
* Key column (e.g., `customer_id`) is preserved

📄 Example:

| customer\_id | name | city | \_change\_type |
| ------------ | ---- | ---- | -------------- |
| 101          | NULL | NULL | delete         |

---

### ✅ Why It’s Important?

In real-time systems, it’s critical to **remove deleted rows** from:

* Silver or Gold tables
* Reporting layers
* External sync systems (Redshift, Elasticsearch, etc.)

---

### ✅ How to Apply DELETEs to Target

Handled inside `MERGE INTO` using:

```sql
WHEN MATCHED AND src._change_type = 'delete' THEN
  DELETE
```

---

### ✅ If You Want to Handle DELETEs Separately:

```python
delete_df = cdf_df.filter("_change_type = 'delete'")
```

Apply direct delete like:

```python
delete_ids = delete_df.select("customer_id").rdd.flatMap(lambda x: x).collect()

spark.sql(f"""
DELETE FROM customer_target
WHERE customer_id IN ({','.join([str(x) for x in delete_ids])})
""")
```

---

## 📊 Metadata Columns Added by Delta CDF

| Column              | Description                             |
| ------------------- | --------------------------------------- |
| `_change_type`      | `insert`, `update`, or `delete`         |
| `_commit_version`   | Delta log version where change occurred |
| `_commit_timestamp` | When the change was committed           |

---

## 📌 Use Cases of CDC + Delta CDF

| Use Case                         | Purpose                                       |
| -------------------------------- | --------------------------------------------- |
| ⏩ Incremental ETL Pipelines      | Process only new and changed records          |
| 📊 Real-Time Dashboards          | Keep BI data up-to-date                       |
| 🔁 External Sync                 | Push only deltas to Snowflake, Redshift, etc. |
| 🧾 Audit / Compliance            | Track what changed, when, and by whom         |
| 🪄 Bronze → Silver → Gold Layers | Propagate changes efficiently across layers   |

---

## ⚠️ Best Practices

* ✅ Always define `startingVersion` or `startingTimestamp`
* ✅ Use strong match conditions in `MERGE`
* ✅ Track `last_processed_version` in checkpoints or audit logs
* ✅ Handle NULLs carefully in DELETE records
* ✅ Use streaming with `foreachBatch()` for real-time sync

---

## ❓ FAQ

**Q: Does CDF track history before enabling it?**
➡️ No, only changes made **after enabling** CDF are tracked.

**Q: Can I use CDF with streaming?**
➡️ Yes, both batch and streaming are supported.

**Q: What happens if I forget `startingVersion`?**
➡️ You will get an error or unpredictable results. Always specify it.

---

## ✅ Summary

| Topic                 | Description                                   |
| --------------------- | --------------------------------------------- |
| What is CDC?          | Capturing inserts, updates, deletes           |
| What is CDF?          | Delta Lake feature to query only changed data |
| Supports DELETEs?     | ✅ Yes, with `_change_type = 'delete'`         |
| How to process?       | Use `MERGE`, `foreachBatch`, or SQL           |
| Works with streaming? | ✅ Yes                                         |
| Output includes?      | Metadata columns with commit info             |
