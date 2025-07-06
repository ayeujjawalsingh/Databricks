# 📘 Delta Change Feed (CDC) in Databricks

Delta Change Feed (DCF) allows you to **capture row-level changes** (INSERT, UPDATE, DELETE) in Delta tables. It helps in building **incremental pipelines** where you read only the changed data, instead of scanning the full table.

---

## 📌 Why Use Delta Change Feed?

| Benefit                          | Description                                              |
|----------------------------------|----------------------------------------------------------|
| ✅ Incremental Reads             | Read only rows that changed (not the entire dataset)     |
| ✅ Track Change Type             | Know if the row was inserted, updated, or deleted        |
| ✅ Efficient for ETL             | Reduces cost and processing time                         |
| ✅ Supports Streaming            | Can be used with structured streaming for real-time use  |
| ✅ Works with Delta Tables       | Built-in support in Delta Lake with transaction logs     |

---

## 🧠 How Delta Change Feed Works

Delta Lake stores all changes to a table in **delta logs** (versioned commits). When DCF is enabled:

- Every change (insert/update/delete) is recorded
- Metadata like `_change_type`, `_commit_version`, `_commit_timestamp` is tracked
- You can query this changed data using Spark (batch or streaming)

---

## 🛠️ Step-by-Step Implementation

### ✅ Step 1: Enable Change Data Feed

```sql
ALTER TABLE <your_table_name>
SET TBLPROPERTIES (
  delta.enableChangeDataFeed = true
);
```

> 🔸 Must be done **before** you want to track changes.

---

### ✅ Step 2: Perform Some Changes

```sql
-- Insert
INSERT INTO <your_table_name> VALUES (101, 'Amit', 'Pune');

-- Update
UPDATE <your_table_name> SET city = 'Mumbai' WHERE customer_id = 101;

-- Delete
DELETE FROM <your_table_name> WHERE customer_id = 101;
```

These changes will be captured in Delta logs.

---

### ✅ Step 3: Read the Changed Data

#### 🧾 Batch Read Example

```python
df_changes = spark.read.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", 10) \
    .table("your_table_name")
```

#### 🧾 Sample Output

| customer\_id | name | city   | \_change\_type |
| ------------ | ---- | ------ | -------------- |
| 101          | Amit | Pune   | insert         |
| 101          | Amit | Mumbai | update         |
| 101          | null | null   | delete         |

---

### ✅ Step 4: Apply Changes Using `MERGE INTO`

```python
df_changes.createOrReplaceTempView("delta_cdc")

spark.sql("""
MERGE INTO target_table AS tgt
USING delta_cdc AS src
ON tgt.customer_id = src.customer_id

WHEN MATCHED AND src._change_type = 'update' THEN UPDATE SET *
WHEN MATCHED AND src._change_type = 'delete' THEN DELETE
WHEN NOT MATCHED AND src._change_type = 'insert' THEN INSERT *
""")
```

This keeps your target table up-to-date with only changed data.

---

### ✅ Step 5: (Optional) Use Delta Change Feed in Streaming

```python
stream_df = spark.readStream.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", 45) \
    .table("your_table_name")
```

This lets you process the changes in real time, ideal for streaming pipelines.

---

## 📊 Metadata Columns Added by DCF

| Column Name         | Description                                  |
| ------------------- | -------------------------------------------- |
| `_change_type`      | Type of change: `insert`, `update`, `delete` |
| `_commit_version`   | Delta log version where change occurred      |
| `_commit_timestamp` | Timestamp when the change was committed      |

---

## 💡 Use Cases of Delta Change Feed

| Use Case                            | Purpose                                                |
| ----------------------------------- | ------------------------------------------------------ |
| 🔄 Incremental ETL Pipelines        | Load only changed data                                 |
| 📊 Real-Time Dashboards             | Always show fresh, accurate data                       |
| 🔁 Data Replication / Sync          | Copy changes to other systems like Redshift, Snowflake |
| 📂 Audit & Version Tracking         | Know what changed, when, and why                       |
| 🚀 Bronze → Silver → Gold Pipelines | Efficiently propagate changes across layers            |

---

## ⚠️ Things to Keep in Mind

* Change feed is only available **after you enable it**.
* You must provide a **starting version** when querying changes.
* Deletes return rows with `null` values and `_change_type = 'delete'`.
* It’s only supported on **Delta tables**, not standard Parquet or CSV.

---

## 🧠 Summary

| Feature              | Description                                 |
| -------------------- | ------------------------------------------- |
| Purpose              | Track and process changes in Delta tables   |
| Supported Operations | `INSERT`, `UPDATE`, `DELETE`                |
| Usage Type           | Batch and Streaming                         |
| Trigger              | Enable with table property                  |
| Integration          | Works with `MERGE INTO` and Spark Streaming |
