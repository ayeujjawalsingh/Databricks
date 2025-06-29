# 🧠 Delta Table Types (Creation, Partitioning, Schema Evolution, Optimization)

Delta Lake is an open-source storage layer that brings ACID transactions, scalable metadata, and unifies batch + streaming workloads to Apache Spark and Databricks.

This document covers the core concepts related to:
- Creating Delta Tables (SQL and PySpark)
- Managed vs External Tables
- Reading/Writing to Delta Tables
- Partitioning
- Schema Evolution
- 32 Column Optimization Concept

---

## 📁 1. Delta Table Types – Managed vs External

### ✅ Managed Table
- Databricks manages both the **data and metadata**.
- Data is stored in a **managed location (DBFS)**.
- When you drop the table, **both data and metadata are deleted**.

### ✅ External Table
- You provide a custom **LOCATION** (e.g. S3 or DBFS path).
- Databricks only manages the **metadata**.
- Dropping the table **does NOT delete** the actual data.

| Feature              | Managed Table                        | External Table                        |
|----------------------|--------------------------------------|---------------------------------------|
| Data Location        | DBFS / Managed by Databricks         | User-defined path                     |
| On DROP TABLE        | Deletes both data + metadata         | Deletes only metadata                 |
| Use Case             | Temporary or internal data           | Persistent or shared with other tools |

---

## 🛠️ 2. Creating Delta Tables

### ✅ Using SQL

#### 👉 Managed Table
```sql
CREATE TABLE employee_managed (
  id INT,
  name STRING,
  salary DOUBLE
)
USING DELTA;
```

#### 👉 External Table

```sql
CREATE TABLE employee_external (
  id INT,
  name STRING,
  salary DOUBLE
)
USING DELTA
LOCATION '/mnt/data/employee_external';
```

---

### ✅ Using Python (PySpark)

#### 👉 Managed Table

```python
df = spark.createDataFrame([
    (1, "Alice", 50000),
    (2, "Bob", 60000)
], ["id", "name", "salary"])

df.write.format("delta").saveAsTable("employee_managed")
```

#### 👉 External Table

```python
df.write.format("delta").save("/mnt/data/employee_external")

spark.sql("""
    CREATE TABLE employee_external
    USING DELTA
    LOCATION '/mnt/data/employee_external'
""")
```

---

## 📖 3. Reading from Delta Tables

### ✅ SQL

```sql
SELECT * FROM employee_managed;
```

### ✅ PySpark

```python
# From managed table
df = spark.read.table("employee_managed")

# From external path
df2 = spark.read.format("delta").load("/mnt/data/employee_external")
```

---

## ✍️ 4. Writing to Delta Tables

### ✅ PySpark Modes

```python
# Overwrite existing data
df.write.format("delta").mode("overwrite").save("/mnt/data/employee_external")

# Append new records
df.write.format("delta").mode("append").save("/mnt/data/employee_external")
```

---

## 📦 5. Partitioning in Delta Lake

### ✅ What is Partitioning?

* Organizes data into **directory structure** by column values.
* Improves **query speed** by pruning folders.

### ✅ Example

Data stored as:

```
/sales_data/country=India/
/sales_data/country=USA/
```

### ✅ SQL Example

```sql
CREATE TABLE sales_data (
  id INT,
  amount DOUBLE,
  country STRING
)
USING DELTA
PARTITIONED BY (country)
LOCATION '/mnt/sales_data';
```

### ✅ PySpark Example

```python
df.write.partitionBy("country").format("delta").save("/mnt/sales_data")
```

### ✅ Querying Partition

```sql
SELECT * FROM sales_data WHERE country = 'India';
```

Only `/country=India/` folder is scanned.

---

## 🔁 6. Schema Evolution in Delta Lake

### ✅ What is Schema Evolution?

Allows new columns to be added without recreating the table.

### ✅ PySpark Example

```python
df_new = spark.createDataFrame([
    (1, "Alice", 30),
    (2, "Bob", 25)
], ["id", "name", "age"])  # 'age' is new

df_new.write \
  .format("delta") \
  .mode("append") \
  .option("mergeSchema", "true") \
  .save("/mnt/employee_data")
```

Without `mergeSchema = true`, the above write would fail.

### ✅ With `saveAsTable`

```python
df.write.option("mergeSchema", "true") \
  .mode("append") \
  .saveAsTable("employee_data")
```

---

## 📊 7. Delta Lake 32-Column Optimization Concept

### ✅ What is It?

Delta Lake collects **min/max column statistics** only for the **first 32 columns** (by default). These stats help in **data skipping**.

### ✅ Why It Matters?

If your table has more than 32 columns:

* Only first 32 columns can be used for **query optimization** (data skipping).
* Columns after 32 are **not considered** for min/max statistics.

### ✅ Data Skipping Example

```sql
SELECT * FROM sales WHERE order_date = '2024-01-01';
```

If `order_date` is within first 32 columns, Delta Lake skips irrelevant files using stats. If it's beyond 32, it reads all files.

---

### ✅ Best Practices

* **Reorder schema** to keep frequently filtered columns in first 32.
* Use `DESCRIBE DETAIL delta.` to inspect schema and stats.

### ✅ Can We Change This Limit?

❌ No. As of now, this is a **hardcoded internal optimization**.

| Rule                     | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| Stats applied to columns | Only for first 32 columns                                    |
| Impact                   | Faster filters if column is in first 32                      |
| Workaround               | Put high-value columns (e.g. `date`, `region`, `type`) early |

---

## ✅ Summary Table

| Feature              | SQL Example                    | PySpark Example                           |
| -------------------- | ------------------------------ | ----------------------------------------- |
| Managed Table        | `CREATE TABLE ... USING DELTA` | `saveAsTable("name")`                     |
| External Table       | `CREATE TABLE ... LOCATION`    | `write.save("path") + CREATE TABLE`       |
| Read Table           | `SELECT * FROM table`          | `read.table("name")` or `read.load(path)` |
| Write Table          | INSERT INTO                    | `mode("append" / "overwrite")`            |
| Partition Table      | `PARTITIONED BY (column)`      | `partitionBy("column")`                   |
| Schema Evolution     | Not supported directly in SQL  | `option("mergeSchema", "true")`           |
| 32 Column Limitation | No direct control in SQL       | Schema order affects data skipping        |
