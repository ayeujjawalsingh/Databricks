# 📘 Delta Lake DML Operations in Databricks

Delta Lake supports SQL-like **Data Manipulation Language (DML)** operations such as `INSERT`, `UPDATE`, `DELETE`, and `MERGE`. These operations enable you to manage and modify data in Delta tables with full **ACID guarantees** on big data.

---

## 📥 INSERT in Delta Lake

### ✅ Purpose
Insert new records into a Delta table.

### 🧾 Syntax
```sql
INSERT INTO table_name (column1, column2)
VALUES (value1, value2);
```

### 📌 Example

```sql
INSERT INTO employees (id, name, department)
VALUES (101, 'Ujjawal Singh', 'Software Developer');
```

### 🧠 Notes

* Inserts one or more new rows.
* Can be used for both **Delta managed and external tables**.

---

## 🔄 UPDATE in Delta Lake

### ✅ Purpose

Update existing records in a Delta table based on a condition.

### 🧾 Syntax

```sql
UPDATE table_name
SET column1 = value1, column2 = value2
WHERE condition;
```

### 📌 Example

```sql
UPDATE employees
SET salary = salary * 1.10
WHERE department = 'Sales';
```

### 🧠 Notes

* Only records that match the condition will be updated.
* Changes are **tracked in Delta log** for time travel and auditing.

---

## ❌ DELETE in Delta Lake

### ✅ Purpose

Delete existing records from a Delta table based on a condition.

### 🧾 Syntax

```sql
DELETE FROM table_name
WHERE condition;
```

### 📌 Example

```sql
DELETE FROM orders
WHERE status = 'Cancelled';
```

### 🧠 Notes

* Supports conditional deletion.
* Old data files are marked as removed, not immediately deleted (unless `VACUUM` is run).

---

## 🔁 MERGE in Delta Lake (Upsert and Conditional Logic)

### ✅ Purpose

Perform an **Upsert** or conditional action like update, delete, or insert — all in a single query.

### 🧾 General Syntax

```sql
MERGE INTO target_table AS target
USING source_table AS source
ON target.id = source.id
WHEN MATCHED THEN
  UPDATE SET target.column = source.column
WHEN NOT MATCHED THEN
  INSERT (columns) VALUES (source.columns);
```

---

### 🔀 Merge Example 1: Update If Match, Insert If Not

```sql
MERGE INTO customers AS target
USING updates AS source
ON target.customer_id = source.customer_id
WHEN MATCHED THEN
  UPDATE SET target.email = source.email
WHEN NOT MATCHED THEN
  INSERT (customer_id, name, email)
  VALUES (source.customer_id, source.name, source.email);
```

🧠 **Use Case**: Sync customer information (update if exists, insert if new).

---

### 🔄 Merge Example 2: Delete If Match, Insert If Not

```sql
MERGE INTO blacklist_users AS target
USING incoming_users AS source
ON target.user_id = source.user_id
WHEN MATCHED THEN DELETE
WHEN NOT MATCHED THEN
  INSERT (user_id, username)
  VALUES (source.user_id, source.username);
```

🧠 **Use Case**: Remove existing blacklisted users if matched, insert new blacklist entries otherwise.

---

## 🧹 VACUUM After DML

After running heavy `UPDATE`, `DELETE`, or `MERGE` operations, it's a good practice to clean up old data files using `VACUUM`.

```sql
VACUUM table_name RETAIN 168 HOURS;
```

🧠 Retains files for 7 days (default). Adjust retention as needed.

---

## ⚙️ How DML Works Internally

* Delta Lake **never overwrites Parquet files directly**.
* Instead, it writes **new files** with updated data and tracks the change in the `_delta_log`.
* This enables:

  * **Time travel**
  * **Atomic commits**
  * **Scalability and fault tolerance**

---

## 📊 Summary Table

| Operation | Purpose                         | Syntax Support | Notes                                          |
| --------- | ------------------------------- | -------------- | ---------------------------------------------- |
| INSERT    | Add new records                 | ✅              | Supports static inserts                        |
| UPDATE    | Modify existing records         | ✅              | Works with WHERE clause                        |
| DELETE    | Remove unwanted records         | ✅              | Conditional deletions                          |
| MERGE     | UPSERT / Conditional operations | ✅              | Most flexible, supports INSERT, UPDATE, DELETE |

---

> ℹ️ **Tip:** Use `MERGE` for CDC (Change Data Capture) or syncing data from external sources like Kafka, S3, or Redshift.
