# 📘 Processing CDC Feed from Delta Change Data Feed in Databricks

This guide explains how to **process Change Data Capture (CDC)** using **Delta Change Feed** in Databricks.

---

## ✅ What Does "Processing CDC Feed" Mean?

Once you **enable change data feed** on a Delta table, Databricks captures changes like:

- ➕ **INSERT**
- 🔄 **UPDATE**
- ❌ **DELETE**

**Processing the CDC feed** means:

1. Reading the changed rows (not full table)
2. Understanding what type of change each row is
3. Applying those changes to your target table using `MERGE`, `INSERT`, `DELETE`, etc.

---

## 🧱 Typical Pipeline Flow

```

Source Delta Table (CDC Enabled)
⬇️
Read Changed Rows (only)
⬇️
Detect Change Type (insert/update/delete)
⬇️
Apply to Target Table using MERGE

```

---

## 🛠️ Step-by-Step Guide: Processing CDC Feed

---

### ✅ Step 1: Read Changes from Source Table

```python
cdc_df = spark.read.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", 15) \
    .table("customer_source")
````

> You’ll get only changed rows **since version 15**.

---

### ✅ Step 2: Inspect CDC Data

Each row includes a special column called `_change_type`.

Sample data:

| customer\_id | name | city  | \_change\_type |
| ------------ | ---- | ----- | -------------- |
| 101          | Amit | Pune  | insert         |
| 102          | Riya | Delhi | update         |
| 103          | null | null  | delete         |

---

### ✅ Step 3: Prepare a Temporary View

Create a view so you can apply SQL logic easily.

```python
cdc_df.createOrReplaceTempView("cdc_data")
```

---

### ✅ Step 4: Apply Changes Using `MERGE INTO`

Use `MERGE` to apply insert/update/delete in one step.

```sql
MERGE INTO customer_target AS tgt
USING cdc_data AS src
ON tgt.customer_id = src.customer_id

WHEN MATCHED AND src._change_type = 'update' THEN
  UPDATE SET *

WHEN MATCHED AND src._change_type = 'delete' THEN
  DELETE

WHEN NOT MATCHED AND src._change_type = 'insert' THEN
  INSERT *
```

> 🧠 This ensures the **target table stays in sync** with the source changes.

---

## 🔁 Bonus: Use Structured Streaming to Process CDC in Real-Time

```python
cdc_stream_df = spark.readStream.format("delta") \
    .option("readChangeData", "true") \
    .option("startingVersion", 20) \
    .table("customer_source")
```

You can use `foreachBatch` to apply logic like `MERGE` on streaming data.

---

## 📊 Understanding `_change_type`

| \_change\_type | Meaning              | How it is processed    |
| -------------- | -------------------- | ---------------------- |
| `insert`       | New row added        | Use `INSERT` in target |
| `update`       | Existing row changed | Use `UPDATE` in target |
| `delete`       | Row was removed      | Use `DELETE` in target |

> 🔸 Deleted rows will have `NULL` for other columns.

---

## 🧠 Summary: Processing CDC Feed

| Step               | Description                               |
| ------------------ | ----------------------------------------- |
| 1️⃣ Read Feed      | Use `.option("readChangeData", "true")`   |
| 2️⃣ Filter Changes | Detect change type using `_change_type`   |
| 3️⃣ Merge          | Use `MERGE INTO` to apply to target table |
| 4️⃣ Streaming      | Optionally use `readStream` for real-time |

---

## 📌 Best Practices

* Always provide a **`startingVersion`** to read CDC safely
* For streaming, handle **idempotency** (avoid duplicate updates)
* Use filters to process specific change types if needed

---

## 📎 Real Use Cases

| Use Case            | How CDC Feed Helps                    |
| ------------------- | ------------------------------------- |
| 🧪 ETL Pipelines    | Applies only changes, not full reload |
| 📈 Dashboards       | Keeps metrics updated live            |
| 🔁 Replicating Data | Keeps sync between systems or layers  |
| 🧾 Audit/Compliance | Tracks exactly what changed and when  |

---

## 🧰 Optional Enhancements

* Store `_commit_version` in the target table for lineage
* Use a **checkpoint table** to track last processed version
* Add retry mechanism for streaming CDC pipelines

---

## ✅ Conclusion

Processing CDC Feed is about:

* Efficiently tracking and reading changed rows
* Applying changes to your downstream data tables
* Keeping data fresh, clean, and in sync

It’s a **must-have pattern** for modern data lakes using Delta Lake and Databricks.
