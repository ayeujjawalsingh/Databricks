# 🧾 Delta Transaction Logs in Delta Lake

Delta Lake brings **ACID transactions** to big data workloads using an **immutable transaction log**. This log tracks every change made to the Delta table, making your data **reliable, consistent, and versioned**.

---

## 📌 What Is a Delta Transaction Log?

The **Delta Transaction Log** (also called **Delta Log**) is the **central source of truth** for any Delta table.

- It tracks all changes (data + metadata) to a table in a versioned, append-only log.
- Stored inside the `_delta_log/` directory at the root of the Delta table.
- Powers features like **time travel, ACID compliance, schema evolution, and audit history**.

---

## 📂 Delta Log File Structure

```

/your-delta-table/
├── part-00000.snappy.parquet      # actual data file
├── ...
└── \_delta\_log/
├── 00000000000000000000.json      # commit v0
├── 00000000000000000001.json      # commit v1
├── ...
├── 00000000000000000100.checkpoint.parquet
└── \_last\_checkpoint

```

### 🔸 JSON Files
- Each `.json` file represents a **transaction commit**.
- JSON files contain actions such as:
  - `add`: new data file added
  - `remove`: data file deleted
  - `metaData`: schema or table properties
  - `protocol`: read/write version info
  - `txn`: info for idempotent transactions

### 🔸 Checkpoint Files
- Stored as **Parquet**.
- Created every **10 commits by default**.
- Consolidates previous JSON logs into a compact format for **faster table reads**.
- Example: `00000000000000000100.checkpoint.parquet`

---

## 🛠️ Key Delta Log Actions

| Action        | Description |
|---------------|-------------|
| `add`         | A new data file has been added to the table |
| `remove`      | A file is removed (during delete/update/overwrite) |
| `metaData`    | Stores schema, partition info, table properties |
| `protocol`    | Specifies required Delta reader/writer version |
| `txn`         | Supports exactly-once writes for streaming jobs |

---

## 💡 Why Is the Delta Log Important?

- ✅ **ACID Compliance**: Ensures atomicity and consistency across write operations.
- ✅ **Time Travel**: Enables reading data as of a specific version or timestamp.
- ✅ **Audit Trail**: Complete historical tracking of table operations.
- ✅ **Schema Evolution**: Metadata updates recorded automatically.
- ✅ **Concurrency**: Allows concurrent reads and writes using optimistic concurrency control.

---

## 🔍 How to Explore the Transaction Log

### 🔸 Using Spark

```python
# Load the log JSON files
spark.read.json("/mnt/delta/sales/_delta_log/*.json").show(truncate=False)
````

### 🔸 Read a Past Version (Time Travel)

```python
# Version-based
df = spark.read.format("delta").option("versionAsOf", 3).load("/mnt/delta/sales")

# Timestamp-based
df = spark.read.format("delta").option("timestampAsOf", "2023-07-01T12:00:00").load("/mnt/delta/sales")
```

---

## ⏪ Example: What Happens During a Write?

```python
df.write.format("delta").mode("append").save("/mnt/delta/sales")
```

This triggers:

1. Writing a new Parquet data file.
2. Creating a commit JSON in `_delta_log/`:

   ```json
   {
     "add": {
       "path": "part-00000-abc.snappy.parquet",
       "size": 12345,
       "modificationTime": 1625246768000,
       "dataChange": true
     }
   }
   ```

---

## 🧹 Log Retention and Cleanup

* Delta retains log history for **30 days by default**.
* You can clean up old logs and data files using `VACUUM`.

### Run VACUUM

```sql
-- Retain only 7 days of history
VACUUM delta.`/mnt/delta/sales` RETAIN 168 HOURS;
```

### Important:

* Use `VACUUM` carefully.
* Time travel won’t work for versions that have been vacuumed.

---

## 📊 Table Version Timeline Example

| Version | Action             | Details                               |
| ------- | ------------------ | ------------------------------------- |
| v0      | Table Created      | Contains schema and metadata          |
| v1      | Data Appended      | New Parquet file added (`add` action) |
| v2      | Row Deleted        | File removed and new file added       |
| v3      | Schema Changed     | New metadata with updated schema      |
| v10     | Checkpoint Written | Combines commits 0–9 into checkpoint  |

---

## 📘 Sample Table Layout

```
/mnt/delta/events/
  ├── part-00001.parquet
  └── _delta_log/
        ├── 00000000000000000000.json
        ├── 00000000000000000001.json
        ├── ...
        ├── 00000000000000000010.checkpoint.parquet
        └── _last_checkpoint
```

---

## ✅ Best Practices

* Use `VACUUM` periodically to delete obsolete log/data files.
* Use `DESCRIBE HISTORY` for audit and lineage tracking.
* Configure **log retention period** based on business needs.
* Avoid `VACUUM` if you still need older versions for time travel.

---

## 📚 Related Delta Commands

```sql
-- View table history
DESCRIBE HISTORY delta.`/mnt/delta/sales`;

-- Time travel using SQL
SELECT * FROM delta.`/mnt/delta/sales` VERSION AS OF 5;

-- Read Delta log JSON (advanced)
spark.read.json("/mnt/delta/sales/_delta_log/*.json").show()
```
