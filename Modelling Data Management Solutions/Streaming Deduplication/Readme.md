# 🔁 Streaming Deduplication

## 📘 What is Streaming Deduplication?

When you’re ingesting data using **streaming** (e.g., from Kafka, files, or APIs), **duplicate records** can come in due to:
- Retry logic in upstream systems
- Network failures
- Late arriving data

👉 **Streaming Deduplication** means:
> Removing these duplicate records **before storing them**, so your Silver/Gold tables don’t have repeated or incorrect data.

---

## ❗ Why Is Deduplication Important?

| Problem                        | Impact                          |
|--------------------------------|----------------------------------|
| ✅ Duplicate orders             | Double sales numbers             |
| ✅ Duplicate customer records   | Wrong customer counts            |
| ✅ Duplicate sensor readings    | Incorrect IoT analytics          |

🔧 You need **deduplication logic** to make your data **clean, accurate, and trustworthy**.

---

## 🧠 How to Identify Duplicates?

To detect duplicates, you need at least one of:

| Field             | Description |
|-------------------|-------------|
| **Unique ID**     | A key like `order_id`, `event_id`, etc. |
| **Event Timestamp**| When the event actually happened |
| **Ingestion Time** | When you received the data |

---

## 🛠️ Deduplication in Streaming (Structured Streaming)

### ✅ Approach: Use `dropDuplicates()`

Spark supports **deduplication in streaming** using:

```python
dropDuplicates(["unique_column"])
```

Or based on a **combination of columns**:

```python
dropDuplicates(["order_id", "event_timestamp"])
```

---

## 🧪 Example: Deduplicating Orders Stream

```python
stream_df = (spark.readStream
    .format("delta")
    .table("bronze.orders_raw"))

# Deduplicate using order_id
deduped_df = stream_df.dropDuplicates(["order_id"])

# Write to silver layer
deduped_df.writeStream \
    .format("delta") \
    .outputMode("append") \
    .option("checkpointLocation", "/chk/silver/orders/") \
    .table("silver.orders_cleaned")
```

---

## 🧩 Best Columns for Deduplication

| Column Type        | Use It When...                            |
| ------------------ | ----------------------------------------- |
| `event_id`         | You have a natural event key (e.g., UUID) |
| `order_id`         | In transactional systems like e-commerce  |
| `customer_id + ts` | For events per customer                   |
| `record_hash`      | Create a hash of important fields as ID   |

---

## ⚠️ Important Notes on Streaming Deduplication

| Caution                                            | Explanation                                                  |
| -------------------------------------------------- | ------------------------------------------------------------ |
| Works only in `append` mode                        | Cannot deduplicate in `complete` or `update` modes           |
| Not guaranteed across all batches                  | Only deduplicates **within a single micro-batch** by default |
| Needs a **watermark** for time-based deduplication | Prevents memory overload when deduplicating by time          |

---

## ⏱️ Deduplication with Watermark (for late events)

Sometimes events arrive **late**, and you want to deduplicate within a time window.

Use a **watermark** to define how late an event can be.

```python
from pyspark.sql.functions import expr

deduped_df = (stream_df
    .withWatermark("event_time", "10 minutes")
    .dropDuplicates(["order_id", "event_time"]))
```

📝 This means:

> Only keep the **first unique order\_id** per 10-minute window based on event time

---

## 💡 Tip: Create a Hash Column for Deduplication

If you don’t have a unique ID, you can **create a fingerprint** of the row:

```python
from pyspark.sql.functions import sha2, concat_ws

df = df.withColumn("record_hash", sha2(concat_ws("||", *df.columns), 256))
deduped_df = df.dropDuplicates(["record_hash"])
```

---

## ✅ Best Practices

| Best Practice                    | Why It Helps                 |
| -------------------------------- | ---------------------------- |
| Use unique keys if available     | Easy and fast                |
| Use combination of fields + time | Better for sensor / IoT data |
| Always use watermarks for time   | Controls memory usage        |
| Log dropped duplicates           | Helps in monitoring          |
| Use `record_hash` if no keys     | Flexible fallback option     |

---

## 📐 Architecture Flow – Streaming Deduplication

```
[ Raw Stream from Kafka or Autoloader ]
            |
            v
 ┌──────────────────────────────┐
 | dropDuplicates(["order_id"]) |
 └──────────────────────────────┘
            |
            v
[ Clean Data Stored in Silver Table ]
```

---

## 🧾 Summary – Streaming Deduplication in a Nutshell

| ✅ What to Do                | 🧠 Why It Matters         |
| --------------------------- | ------------------------- |
| Use `dropDuplicates()`      | Remove duplicates easily  |
| Use watermark with time     | Handle late data properly |
| Use `record_hash` if needed | When no ID is available   |
| Log or store dropped rows   | For audit and debugging   |
| Store clean data in Silver  | Keep analytics accurate   |
