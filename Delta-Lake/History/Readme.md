# 🧾 Delta Lake Table History (Databricks)

Delta Lake keeps track of **all changes** made to a table over time. This feature is called **"Table History"**, and it enables powerful functionality like **auditing**, **debugging**, and **time travel** (querying old versions of the data).

---

## 📌 What Is Table History?

Delta Lake stores every action (like `INSERT`, `UPDATE`, `DELETE`, or `MERGE`) in a **transaction log**. This log helps you:
- Track **who changed the data**
- Know **what operation** was done
- See **when it happened**
- Query **previous versions** of the data

The transaction log lives in the `_delta_log/` directory inside the table location.

---

## 📜 How to View Table History

You can use SQL or PySpark to view history.

### ✅ SQL Syntax

```sql
DESCRIBE HISTORY delta.`/path/to/table`
```

Or for catalog tables:

```sql
DESCRIBE HISTORY database_name.table_name;
```

### 🧑‍💻 PySpark Example

```python
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "/path/to/delta-table")
history_df = delta_table.history()  # View full history
history_df.show()
```

---

## 🧠 Sample Output

| version | timestamp           | operation | userId | operationParameters        |
| ------- | ------------------- | --------- | ------ | -------------------------- |
| 3       | 2025-06-28 15:12:00 | DELETE    | user1  | predicate -> \["id = 101"] |
| 2       | 2025-06-28 14:30:00 | MERGE     | user1  | rowsMerged -> 500          |
| 1       | 2025-06-28 10:00:00 | INSERT    | user1  | numFiles -> 3              |

---

## ⏳ Time Travel (Query Old Versions)

Delta lets you **query the table as it was in the past**, either by version or timestamp.

### 🔁 Query by Version

```sql
SELECT * FROM delta.`/path/to/table` VERSION AS OF 2;
```

### ⏰ Query by Timestamp

```sql
SELECT * FROM delta.`/path/to/table` TIMESTAMP AS OF '2025-06-28T10:00:00';
```

> ✅ Useful for debugging, data audits, or reprocessing old data.

---

## 📦 How Delta Maintains History Internally

Every Delta table has a `_delta_log/` folder that contains:

* **JSON log files** for each operation
* **Checkpoint files** for performance
* Metadata like schema, version, timestamps, stats

Each write operation (like `UPDATE`, `DELETE`) creates a new **version** of the table.

---

## 🧹 Cleaning Up Old Versions (VACUUM)

Delta stores older data files to support time travel. But to save space, you can clean them using `VACUUM`.

### 🧼 Example

```sql
VACUUM delta.`/path/to/table` RETAIN 168 HOURS; -- Retain 7 days
```

> ⚠️ By default, Delta keeps files for **7 days**. You can change the retention duration.

---

## 🔐 Use Cases of Table History

* 🔍 **Audit**: See who changed what and when
* 🧪 **Debug**: Reproduce bugs by going back in time
* 📦 **Rollback**: Restore a previous version of the table
* 📊 **Reprocessing**: Use historical snapshots for reporting

---

## ✅ Summary

| Feature       | Description                                          |
| ------------- | ---------------------------------------------------- |
| History       | Tracks all operations on the table                   |
| Versioning    | Creates new version after every write                |
| Time Travel   | Query data from a specific version or timestamp      |
| `_delta_log/` | Internal folder that stores all metadata and history |
| Retention     | Keeps data for 7 days by default (can be changed)    |
| VACUUM        | Used to delete old files and free up space           |

---

## 📚 Additional Tips

* Use `DESCRIBE HISTORY` to see the change log
* Use `VERSION AS OF` and `TIMESTAMP AS OF` to time travel
* Never run `VACUUM RETAIN 0 HOURS` in production unless you're **absolutely sure** — it will delete all old data files

---

> 🧠 Delta History is one of the most powerful features of Delta Lake — it gives you built-in **data versioning**, **lineage**, and **rollback** with no extra tools needed.
