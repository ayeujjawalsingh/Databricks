# ✅ Data Quality Enforcement

## 📘 What is Data Quality Enforcement?

**Data Quality Enforcement** means:
> Making sure the data coming into your system is **correct**, **complete**, and **clean** — before you use it for reports, ML, or analytics.

It is like **checking and cleaning groceries** before cooking:
- 🧼 Remove bad items
- 🏷️ Fix labeling
- ✅ Ensure everything is fresh and in expected format

---

## 🏗️ Where Do We Apply Quality Enforcement?

Usually applied **after Bronze Layer**, during:
- 🔄 Bronze → Silver (raw → cleaned data)
- 📊 Silver → Gold (clean → business-ready)

---

## 🚦 Why Is It Important?

| Reason                  | Description |
|-------------------------|-------------|
| 🧹 Avoid garbage results | Dirty data = wrong analytics |
| 🛑 Prevent system failures | Nulls or wrong types can crash models |
| 📊 Build trust           | Business teams need reliable data |
| 🔁 Enable automation     | Clean data = less manual fixing |
| ✅ Regulatory compliance | Required in healthcare, finance, etc. |

---

## 🔍 Types of Data Quality Checks

| Check Type      | What It Means                                     | Example |
|------------------|--------------------------------------------------|---------|
| ✅ **Null checks**    | Make sure important columns are not null         | `customer_id IS NOT NULL` |
| 🔤 **Data type check**| Column is of correct type                        | `amount` should be number |
| 🔢 **Range check**    | Values are within expected range                 | `age BETWEEN 0 AND 100` |
| 📅 **Format check**   | Check if date is in valid format                 | `yyyy-MM-dd` |
| 🎯 **Unique check**   | Values like IDs or emails are not duplicated    | No 2 rows have same `email` |
| 🔗 **Foreign key check**| Reference values exist in lookup table        | `customer_id` must exist in `customers` table |

---

## 🛠️ How Do We Enforce Quality?

### ✨ Option 1: Filter Out Bad Data
Let bad data **go into a separate “quarantine” table** so it doesn’t pollute your clean tables.

```python
# Example: Separate good and bad records
good_data = df.filter("amount IS NOT NULL AND amount > 0")
bad_data = df.filter("amount IS NULL OR amount <= 0")

# Save clean data to Silver
good_data.write.format("delta").saveAsTable("silver.orders_clean")

# Save bad data to quarantine
bad_data.write.format("delta").saveAsTable("quarantine.orders_bad")
```

---

### ✨ Option 2: Throw Errors Using Expectation (Delta Live Tables)

```python
CREATE LIVE TABLE silver_customers_cleaned
AS SELECT * FROM live.bronze_customers
EXPECT customer_id IS NOT NULL
  ON VIOLATION DROP ROW
EXPECT age BETWEEN 0 AND 120
  ON VIOLATION DROP ROW
```

> ⚠️ **Delta Live Tables (DLT)** support built-in `EXPECT` clauses for validation

---

## 🧪 Tools & Techniques for Quality Checks

| Tool/Feature                  | Description                                  |
| ----------------------------- | -------------------------------------------- |
| ✅ **Delta Live Tables (DLT)** | Built-in data quality rules using `EXPECT`   |
| ✅ **Great Expectations**      | Open-source tool for automated tests on data |
| ✅ **Spark SQL filters**       | Use WHERE, CASE, IF to clean data            |
| ✅ **Quarantine table**        | Store bad data separately for later fixing   |
| ✅ **Schema enforcement**      | Ensure data matches expected structure       |
| ✅ **Alerts / Logging**        | Notify if bad data rate crosses threshold    |

---

## 🧩 Where to Store Invalid (Bad) Data?

It's a good practice to store it in a separate table:

```plaintext
/quarantine/
  └── orders_bad/
       ├── null_amount/
       └── negative_amount/
```

This helps in:

* Debugging
* Reprocessing later
* Sending alerts

---

## 🧾 Sample Quality Rules for Common Entities

### 📦 Orders Table

| Rule                     | Why?                 |
| ------------------------ | -------------------- |
| `order_id IS NOT NULL`   | Key field            |
| `amount > 0`             | No negative orders   |
| `order_date IS NOT NULL` | Must have order time |

### 👤 Customers Table

| Rule                      | Why?            |
| ------------------------- | --------------- |
| `customer_id IS NOT NULL` | Key field       |
| `email IS NOT NULL`       | Communication   |
| `age BETWEEN 0 AND 120`   | Valid human age |

---

## ✅ Summary – Easy Checklist

| Task                                   | Done? |
| -------------------------------------- | ----- |
| Check for nulls in important columns   | ✅     |
| Validate number ranges (amount, age)   | ✅     |
| Check formats for dates, emails        | ✅     |
| Store bad data separately (quarantine) | ✅     |
| Apply rules in Bronze → Silver         | ✅     |

---

## 📐 Real-Life Example Architecture

```plaintext
[ Bronze Table ]
     |
     v
[ Apply Quality Checks ]
     ├── Good Data  ──> Silver Table
     └── Bad Data   ──> Quarantine Table
```

---

## 🧠 Final Tip

Data Quality is not a one-time thing — it’s a **continuous process**.
Start simple, improve rules over time, and always monitor.
