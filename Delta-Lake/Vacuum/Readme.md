# 🧹 VACUUM in Databricks (Delta Lake)

The `VACUUM` command in Databricks is used to clean up obsolete files in Delta Lake tables and free up storage. This is essential for maintaining performance, especially after large delete or update operations.

---

## 📌 What is VACUUM?

Delta Lake stores the history of all changes (for **time travel** and **data versioning**). When records are deleted or updated, the old data files remain in storage until cleaned manually or automatically.

The `VACUUM` command:
- Physically **deletes unreferenced data files**.
- Helps reclaim **storage space**.
- Keeps the Delta table **optimized and performant**.

---

## 🧪 Syntax

```sql
VACUUM table_name [RETAIN <N> HOURS];
```

OR for external tables not registered in metastore:

```sql
VACUUM delta.`<path_to_table>` [RETAIN <N> HOURS];
```

---

## 📁 VACUUM on Table Types

| Table Type          | Example Syntax                                          |
| ------------------- | ------------------------------------------------------- |
| Managed Table       | `VACUUM managed_table RETAIN 1 HOURS;`                  |
| External Table      | `VACUUM delta.'/mnt/datalake/my_table' RETAIN 1 HOURS;` |
| Registered External | `VACUUM external_table RETAIN 1 HOURS;`                 |

---

## ⏳ Retention Period (Default: 7 Days)

By default, Delta Lake retains deleted files for **7 days** to support **time travel**.

You can specify a custom retention period using the `RETAIN` clause (minimum: `1 HOUR`):

```sql
VACUUM my_table RETAIN 168 HOURS; -- 7 days
VACUUM my_table RETAIN 1 HOURS;   -- 1 hour
```

---

## 🔐 Retention Check Warning

To protect against accidental data loss, Delta prevents vacuuming files younger than 7 days unless you **explicitly disable the safety check**:

```python
spark.conf.set("spark.databricks.delta.retentionDurationCheck.enabled", "false")
```

> ⚠️ **Only disable this if you're certain time travel is not needed.**

---

## 🛠 Full VACUUM Example (Python + SQL)

```python
# Disable retention duration safety check (optional)
spark.conf.set("spark.databricks.delta.retentionDurationCheck.enabled", "false")

# Vacuum managed table
spark.sql("VACUUM managed_table RETAIN 1 HOURS")

# Vacuum registered external table
spark.sql("VACUUM external_table RETAIN 1 HOURS")

# Vacuum unregistered external table by path
external_path = "/mnt/data/external_table_path"
spark.sql(f"VACUUM delta.`{external_path}` RETAIN 1 HOURS")
```

---

## 💡 When to Use VACUUM?

| Use Case                           | VACUUM Recommended? |
| ---------------------------------- | ------------------- |
| After large DELETE or UPDATE       | ✅ Yes               |
| To reduce cloud storage costs      | ✅ Yes               |
| Regular maintenance jobs           | ✅ Yes               |
| When using time travel extensively | ⚠️ Use with caution |

---

## 🧠 Related Concepts

| Feature         | Description                                                                 |
| --------------- | --------------------------------------------------------------------------- |
| **Time Travel** | Access previous versions using `VERSION AS OF` or `TIMESTAMP AS OF`         |
| **Delta Log**   | Keeps metadata and file references to support transactional reads           |
| **Retention**   | Period during which deleted data is retained for rollback                   |
| **Restore**     | You can `RESTORE` to previous versions — if old files haven’t been vacuumed |

---

## 🚫 Common Pitfalls

* Running `VACUUM` with low retention while expecting to use time travel or rollback.
* Incorrect path for external tables not ending in Delta table root.
* Forgetting to disable the retention check when specifying less than 7 days.

---

## 📌 Best Practices

* Run `VACUUM` regularly via scheduled jobs.
* Use `RETAIN` period wisely (minimum `1 HOUR`, default `168 HOURS`).
* Always test before vacuuming production tables.
* Don't disable retention checks casually in production environments.
* Monitor storage usage to identify when vacuuming is necessary.
