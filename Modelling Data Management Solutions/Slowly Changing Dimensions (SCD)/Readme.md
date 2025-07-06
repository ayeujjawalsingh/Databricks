# 🐢 Slowly Changing Dimensions (SCD)

## 📘 What are Dimensions?

In data warehousing:
- ✅ **Dimensions** are descriptive data, like:
  - Customer details
  - Product info
  - Employee records
- ✅ These describe facts (like sales, orders, etc.)

---

## 🧠 What is a Slowly Changing Dimension (SCD)?

> A **Slowly Changing Dimension** is a dimension where the values change **slowly over time** — not every day, but occasionally.

🔁 You want to **track those changes** to:
- See **historical values**
- Support **reporting over time**
- Keep **latest version** for active use

---

## 🧪 Example: Customer Table

| customer_id | name     | city      | signup_date |
|-------------|----------|-----------|--------------|
| 1           | Amit     | Mumbai    | 2020-01-01   |
| 2           | Rahul    | Delhi     | 2021-03-15   |

If customer `Amit` moves from Mumbai to Pune:
- This is a **slow change**
- You may want to **track old + new city**

---

## 🎯 Why Use SCD?

| Reason                     | Benefit |
|----------------------------|---------|
| 📈 Track history            | Know what changed and when |
| 🔍 Audit trail              | See who changed what and when |
| 🕵️ Historical analysis      | Answer "What did we know at that time?" |
| 🧾 Business reporting       | Create time-based trends and reports |

---

## 🔢 Types of Slowly Changing Dimensions

| Type | What It Does                              | Use Case |
|------|-------------------------------------------|----------|
| 🔁 **Type 1** | Overwrite the old value                | When you don’t care about history |
| 🧾 **Type 2** | Keep history as new rows               | Best when history matters |
| 🔢 **Type 3** | Add a new column for previous value    | Only one historical value needed |

---

## ✅ Type 1 – Overwrite (No History)

**Just update the row.**
```sql
UPDATE dim_customer
SET city = 'Pune'
WHERE customer_id = 1;
```

🛑 You lose the old value (Mumbai).

🟢 Good when you only care about the latest data.

---

## ✅ Type 2 – Keep History (Track Changes)

**Insert a new row with new data and mark old row as inactive.**

| customer\_id | name | city   | start\_date | end\_date  | current\_flag |
| ------------ | ---- | ------ | ----------- | ---------- | ------------- |
| 1            | Amit | Mumbai | 2020-01-01  | 2023-07-01 | false         |
| 1            | Amit | Pune   | 2023-07-01  | null       | true          |

🔁 You now have **two rows**:

* One for **old value** (Mumbai)
* One for **new value** (Pune)

🟢 Great for historical reporting.

---

## ✅ Type 3 – Previous Value Column

Add a new column like `previous_city`.

| customer\_id | name | city | previous\_city |
| ------------ | ---- | ---- | -------------- |
| 1            | Amit | Pune | Mumbai         |

🟢 Tracks **one change only** — not full history.

---

## 🛠️ How to Implement SCD in Spark/Databricks

### Using MERGE (Type 2 Example)

```python
from pyspark.sql.functions import current_date, lit

# Load new customer data (maybe from Bronze/Silver layer)
new_data = spark.read.table("staging.customers")

# Add dates and flags
new_data = new_data.withColumn("start_date", current_date()) \
                   .withColumn("end_date", lit(None)) \
                   .withColumn("current_flag", lit(True))

# Perform MERGE into Dimension table
spark.sql("""
MERGE INTO dim_customers AS target
USING staging_customers AS source
ON target.customer_id = source.customer_id
  AND target.current_flag = true
  AND (target.city <> source.city OR target.name <> source.name)

WHEN MATCHED THEN
  UPDATE SET current_flag = false, end_date = current_date()

WHEN NOT MATCHED THEN
  INSERT (customer_id, name, city, start_date, end_date, current_flag)
  VALUES (source.customer_id, source.name, source.city, current_date(), null, true)
""")
```

---

## 🧩 Use Cases of SCD in Real Projects

| Data Type           | Type of SCD | Why?                                  |
| ------------------- | ----------- | ------------------------------------- |
| Customers           | Type 2      | Need history for audit                |
| Employees           | Type 2      | Promotions, location changes          |
| Products            | Type 1      | Only latest price or category matters |
| Marketing Campaigns | Type 3      | Just last vs. current campaign        |

---

## 🧾 Summary Table

| SCD Type | Keeps History? | Adds Rows? | Use When…                     |
| -------- | -------------- | ---------- | ----------------------------- |
| Type 1   | ❌ No           | ❌ No       | You only need current values  |
| Type 2   | ✅ Yes          | ✅ Yes      | You need full history         |
| Type 3   | 🔸 Partial     | ❌ No       | You need one level of history |

---

## ✅ Best Practices for SCD

| Tip                                        | Why It Matters                        |
| ------------------------------------------ | ------------------------------------- |
| Use surrogate key (row\_id)                | Prevent duplicate rows                |
| Add `start_date`, `end_date`, `is_current` | Helps in filtering active records     |
| Use `MERGE` in Delta                       | Makes updates + inserts easy          |
| Monitor change volume                      | Too many changes = performance issues |
| Partition on `current_flag` or date        | Faster queries on active data         |

---

## 🧠 Final Tip

If history **matters**, always go with **SCD Type 2**
If you only need the **latest snapshot**, use **Type 1**
