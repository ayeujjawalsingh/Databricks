# 🕒 Time Travel in Delta Lake (Databricks)

Delta Lake’s **Time Travel** feature allows you to query and restore older versions of your Delta table. This is useful for debugging, auditing, data recovery, and historical analysis.

---

## ✅ What is Time Travel?

Time Travel in Delta Lake enables users to:

- Query previous versions of a Delta table
- Restore data from a specific point in time
- Audit or analyze historical changes

It is powered by **Delta’s transaction log (`_delta_log`)**, which stores a versioned record of all table operations (INSERT, UPDATE, DELETE).

---

## 🎯 Why Use Time Travel?

| Use Case         | Purpose                                               |
|------------------|-------------------------------------------------------|
| Debugging        | Investigate data issues from the past                 |
| Auditing         | See who made what changes and when                    |
| Data Recovery    | Undo accidental changes or deletions                  |
| Historical Views | Generate reports from previous versions of the table  |

---

## 🔢 How Versions Work

Each time a Delta table is written to (insert, update, delete), Delta creates a new **version** in the `_delta_log` directory.

You can access these versions by:
- **Version number**
- **Timestamp**

---

## 🛠️ Querying Previous Versions

### By Version Number:
```sql
SELECT * FROM delta.`/path/to/table` VERSION AS OF 3;
```

### By Timestamp:

```sql
SELECT * FROM delta.`/path/to/table` TIMESTAMP AS OF '2024-06-01T10:00:00Z';
```

---

## 📜 View Table History

Use this to view all operations, timestamps, users, and version numbers.

```sql
DESCRIBE HISTORY delta.`/path/to/table`;
```

> This works for both path-based and catalog tables.

---

## ♻️ Restore Older Version

Restore a table to a previous state using:

```sql
RESTORE TABLE my_table TO VERSION AS OF 3;
-- OR
RESTORE TABLE my_table TO TIMESTAMP AS OF '2024-06-01T10:00:00Z';
```

> ✅ Note: `RESTORE` is supported only in **Unity Catalog**-enabled Delta Tables.

---

## 🧹 Time Travel and VACUUM

Delta Lake retains old data files to enable time travel **until VACUUM is run**.

### Default Behavior:

* **VACUUM** removes data files **older than 7 days** by default.
* Without VACUUM, all historical data and time travel remains available.

### Set Custom Retention:

```sql
VACUUM delta.`/path/to/table` RETAIN 720 HOURS; -- 30 days
```

> To reduce retention below 7 days, override the safety check:

```sql
-- Dry run
VACUUM delta.`/path/to/table` RETAIN 1 HOURS DRY RUN;

-- If safe, disable check and run
SET spark.databricks.delta.retentionDurationCheck.enabled = false;
VACUUM delta.`/path/to/table` RETAIN 1 HOURS;
```

---

## 🔐 Best Practices

| Environment   | Strategy                               |
| ------------- | -------------------------------------- |
| Dev/Testing   | Use time travel actively for debugging |
| Production    | Schedule VACUUM with 7+ day retention  |
| Critical data | Avoid vacuuming too frequently         |
| Cost control  | Balance between retention and storage  |

---

## 📌 Summary

| Feature            | Detail                               |
| ------------------ | ------------------------------------ |
| Enabled By Default | ✅ Yes                                |
| Version Query      | `VERSION AS OF` or `TIMESTAMP AS OF` |
| History Log        | Stored in `_delta_log/`              |
| Restore Support    | `RESTORE TABLE` (Unity Catalog only) |
| Cleanup Command    | `VACUUM`                             |
| Default Retention  | 7 days (168 hours)                   |
| Custom Retention   | Configurable using `RETAIN n HOURS`  |

