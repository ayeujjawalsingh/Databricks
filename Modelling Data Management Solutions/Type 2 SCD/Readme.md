# 🧾 SCD Type 2 – Slowly Changing Dimensions (Full History Tracking)

## 📘 What is SCD Type 2?

SCD Type 2 is a way to **track full history of data changes** in a dimension table.

> Every time something changes (e.g. customer moved city), we:
- **Keep the old record**
- **Insert a new row** for the new value
- **Mark old row as inactive**

🎯 Goal: Never lose the past — keep track of “what was true at what time”.

---

## 🧪 Simple Real-Life Example

### Imagine this customer record:

| customer_id | name | city   |
|-------------|------|--------|
| 1           | Amit | Mumbai |

Later, Amit moves to Pune. In SCD Type 2, we do **not update** the city.

✅ Instead, we insert a new record and keep both:

| surrogate_key | customer_id | name | city   | start_date | end_date   | current_flag |
|---------------|-------------|------|--------|------------|------------|---------------|
| 101           | 1           | Amit | Mumbai | 2020-01-01 | 2023-07-01 | false         |
| 102           | 1           | Amit | Pune   | 2023-07-01 | null       | true          |

---

## 🧩 Key Columns in SCD Type 2

| Column         | Purpose |
|----------------|---------|
| `surrogate_key` | Unique row ID (primary key) |
| `customer_id`   | Business key (natural key) |
| `start_date`    | When this version started |
| `end_date`      | When this version ended (null if current) |
| `current_flag`  | Marks whether row is active (`true`) |

---

## 🧠 Why Use SCD Type 2?

| Reason                        | Benefit |
|-------------------------------|---------|
| ✅ Full history of changes     | Great for auditing and compliance |
| ✅ Time-based reporting        | Ask "What was the value on July 2021?" |
| ✅ Historical trends           | Analyze how customer or product changed over time |
| ✅ Data accuracy               | Avoids overwriting past data |

---

## 🏗️ How It Works (Flow Diagram)

```plaintext
[ New data arrives ]
        |
        v
[ Compare with existing ]
        |
   ┌────┴────┐
   | Changed? | ─── No ──> Ignore (Already Up-to-date)
   |   Yes    |
   └────┬────┘
        v
[ Update old row: current_flag = false, end_date = today ]
[ Insert new row: new values, current_flag = true, end_date = null ]
```

---

## 🛠️ How to Implement SCD Type 2 (Delta + Spark SQL)

### Step-by-Step:

1. Detect which records **changed** (compare new vs existing).
2. Update old record → set `current_flag = false`, set `end_date`
3. Insert new record → with new values, `current_flag = true`, `end_date = null`

### 🔧 Sample Code in Databricks

```sql
-- Step 1: Create staging table (new data)
CREATE OR REPLACE TEMP VIEW staging_customers AS
SELECT 1 AS customer_id, 'Amit' AS name, 'Pune' AS city;

-- Step 2: Merge into dimension table (Type 2 logic)
MERGE INTO dim_customers AS target
USING staging_customers AS source
ON target.customer_id = source.customer_id
  AND target.current_flag = true
  AND (target.city <> source.city OR target.name <> source.name)

WHEN MATCHED THEN
  UPDATE SET current_flag = false, end_date = current_date()

WHEN NOT MATCHED THEN
  INSERT (customer_id, name, city, start_date, end_date, current_flag)
  VALUES (source.customer_id, source.name, source.city, current_date(), null, true);
```

---

## ✅ Best Practices

| Practice                                    | Why Important                      |
| ------------------------------------------- | ---------------------------------- |
| Use `surrogate_key`                         | Keeps each version unique          |
| Index on `customer_id`, `current_flag`      | Faster lookups                     |
| Add `ingest_date` or `created_by`           | For auditing pipeline changes      |
| Use `MERGE` in Delta Lake                   | Handles updates + inserts together |
| Partition by `current_flag` or `start_date` | Optimizes performance              |

---

## 🔎 Use Cases

| Domain     | What Changes Slowly?      | Use SCD2? |
| ---------- | ------------------------- | --------- |
| Retail     | Customer address, loyalty | ✅         |
| HR         | Employee role, department | ✅         |
| Finance    | Product interest rate     | ✅         |
| Healthcare | Doctor specialty, clinic  | ✅         |

---

## ⚠️ Common Mistakes

| Mistake                       | Fix It By                            |
| ----------------------------- | ------------------------------------ |
| Overwriting old data (Type 1) | Use insert + update instead          |
| No `start_date` or `end_date` | Add time-tracking fields             |
| Duplicates in final table     | Use `surrogate_key` as primary key   |
| Not updating `current_flag`   | Always set `false` when ending a row |

---

## ✅ Summary – SCD Type 2 At a Glance

| Concept              | Meaning                                   |
| -------------------- | ----------------------------------------- |
| Insert on change     | Keep old, insert new                      |
| Update old row       | Mark as inactive, set end date            |
| Start + end date     | Know when each version was active         |
| True historical view | See all past changes for a record         |
| Preferred for BI     | Especially in slowly-changing master data |

