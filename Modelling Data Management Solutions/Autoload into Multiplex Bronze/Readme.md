# ⚙️ Autoload into Multiplex Bronze

## 📘 What is Autoloader?

> Autoloader is a tool in Databricks that automatically picks up new files from cloud storage like S3, ADLS, or Azure Blob, and loads only the new data into your pipeline or table — without reprocessing old files.

You don't have to manually check folders or write complex logic to detect new files — Autoloader does it for you.

Autoloader is a feature in **Databricks** that:

- 🚀 Automatically loads **new files** from cloud storage (like AWS S3 or Azure ADLS)
- 🧠 Understands file structure and infers schema
- 🔁 Continuously watches folders (streaming)
- ⚙️ Handles schema changes over time

👉 You don’t need to write a new script every time new data lands.

---

## 🧱 What is Multiplex Bronze?

Multiplex Bronze means:

- 📦 One **Bronze Delta Table**
- 🧩 Stores **multiple types** of raw data (e.g., orders, customers, products)
- 🏷️ Each row has a column like `record_type` to say what kind of data it is
- 🪵 Keeps data **raw and untouched** for traceability

---

## 🤝 What is "Autoload into Multiplex Bronze"?

It means:
> Use **Autoloader** to detect and ingest many types of data from folders like `/orders/`, `/customers/`, etc. into **one single Bronze table**.

---

## 📂 Example Folder Structure

Imagine your cloud storage (like S3) looks like this:

```

s3://raw-zone/
├── orders/
│    └── orders\_2025-07-05.json
├── customers/
│    └── customers\_2025-07-05.json
└── products/
└── products\_2025-07-05.json

```

---

## 🎯 What You Want to Do:

✅ Automatically load **all new files**  
✅ Store everything in **one Delta table**  
✅ Identify the **type** of each record (order, customer, etc.)

---

## 🛠️ Step-by-Step Explanation

### ✅ Step 1: Watch root folder using Autoloader

Tell Autoloader to monitor `s3://raw-zone/` and all subfolders

```python
spark.readStream.format("cloudFiles").load("s3://raw-zone/")
````

---

### ✅ Step 2: Extract record type from folder name

Use the folder name (like `/orders/`) to get the `record_type`

```python
from pyspark.sql.functions import input_file_name, regexp_extract

df_with_type = df.withColumn("record_type", regexp_extract(
    input_file_name(), r"/([^/]+)/[^/]+$", 1))
```

---

### ✅ Step 3: Add metadata columns

Add `ingestion_time` column to track when it was loaded

```python
from pyspark.sql.functions import current_timestamp

df_with_meta = df_with_type.withColumn("ingestion_time", current_timestamp())
```

---

### ✅ Step 4: Combine data into one field

Put all columns into a `data` struct so that different schemas can stay flexible

```python
from pyspark.sql.functions import struct

final_df = df_with_meta.select(
    "record_type",
    struct([col for col in df.columns]).alias("data"),
    "ingestion_time"
)
```

---

### ✅ Step 5: Write to a Multiplex Bronze Delta table

```python
(final_df.writeStream
  .format("delta")
  .option("checkpointLocation", "/chk/bronze_multiplex/")
  .outputMode("append")
  .table("bronze.multiplex_raw"))
```

---

## 🧾 Final Table Output: bronze.multiplex\_raw

| record\_type | data (all original fields)  | ingestion\_time     |
| ------------ | --------------------------- | ------------------- |
| "order"      | {"id": 101, "amount": 300}  | 2025-07-05 10:00 AM |
| "customer"   | {"id": 1, "name": "Amit"}   | 2025-07-05 10:01 AM |
| "product"    | {"id": 44, "price": 999.99} | 2025-07-05 10:02 AM |

---

## ✅ Benefits of This Approach

| ✅ Benefit               | Explanation                                       |
| ----------------------- | ------------------------------------------------- |
| Single job for all data | No need to write separate code for each file type |
| Auto ingestion          | Picks up new files automatically                  |
| Easy filtering          | Use `record_type` to separate records later       |
| Schema evolution        | Can handle structure changes over time            |
| Cost-efficient          | All raw data in one table saves time and space    |

---

## 🛑 Common Pitfalls to Avoid

| Problem                   | How to Fix                                  |
| ------------------------- | ------------------------------------------- |
| Wrong `record_type` value | Check folder structure and regex            |
| Autoloader skips file     | It only loads **new files**, not old ones   |
| Too many small files      | Use `OPTIMIZE` and `Z-ORDER` in Silver/Gold |
| Data types don’t match    | Wrap with `struct()` to avoid schema errors |

---

## 📐 Diagram: Architecture Flow

```
Cloud Storage (S3/ADLS)
    ├── /orders/*.json
    ├── /customers/*.json
    └── /products/*.json
         |
         v
Databricks Autoloader (cloudFiles)
         |
         v
+--------------------------+
| bronze.multiplex_raw     |
| - record_type            |
| - data (struct)          |
| - ingestion_time         |
+--------------------------+
```
