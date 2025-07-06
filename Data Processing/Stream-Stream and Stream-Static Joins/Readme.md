# 📘 Stream Joins in Structured Streaming (Databricks / Spark)

This guide explains two types of joins used in real-time data pipelines using **Spark Structured Streaming**:

- 🔄 Stream-Static Join  
- 🔁 Stream-Stream Join  

These are core to building **real-time ETL pipelines**, **event correlation systems**, and **enrichment workflows**.

---

## 🔄 1. Stream-Static Join

### ✅ What is It?

Join a **real-time streaming DataFrame** with a **fixed/static dataset** (like a lookup or reference table).

> 🔹 Think of enriching streaming data using extra details from a static table.

---

### 🧠 Simple Example

| Stream                    | Static Table                | Use Case                     |
|---------------------------|-----------------------------|------------------------------|
| Login Events (live)       | User Info (static)          | Add name, email to logins    |
| Transactions (stream)     | Country Metadata (static)   | Add country name to events   |
| Product Views (stream)    | Product Catalog (static)    | Add category, brand, price   |

---

### 🛠️ Code Example

```python
# Static dimension table
users_df = spark.read.format("delta").table("user_profiles")

# Live stream of login events
logins_df = spark.readStream.format("delta").table("login_events")

# Stream-Static Join
enriched_df = logins_df.join(users_df, on="user_id", how="left")
```

---

### ✅ Supported Join Types

| Join Type  | Supported       |
| ---------- | --------------- |
| inner      | ✅               |
| left       | ✅               |
| right/full | ❌ Not Supported |

---

### 💡 Key Points

* Static table is read **once** and **broadcasted**.
* Best for **enrichment** (lookup extra columns).
* Static data **won’t update unless you restart the stream**.

---

## 🔁 2. Stream-Stream Join

### ✅ What is It?

Join **two real-time streaming DataFrames** based on matching conditions **within a time window**.

> 🔹 Think of matching two live events that happened close to each other (within a few minutes).

---

### 🧠 Simple Example

| Stream A          | Stream B              | Use Case                    |
| ----------------- | --------------------- | --------------------------- |
| Orders (stream)   | Payments (stream)     | Match paid orders           |
| GPS Data (stream) | Traffic Logs (stream) | Identify vehicle violations |
| Logins (stream)   | Blacklist Updates     | Detect suspicious access    |

---

### ⏱️ Real-Time Needs Special Handling

To work reliably in real-time, you must use:

| Feature         | Why It’s Needed                                 |
| --------------- | ----------------------------------------------- |
| **Watermark**   | To define how late data is accepted             |
| **Event Time**  | To define when the event actually occurred      |
| **Join Window** | To define how long you want to wait for a match |

---

### 🛠️ Code Example

```python
# Orders stream
orders_df = spark.readStream.format("delta").table("orders_stream") \
    .withWatermark("order_time", "10 minutes")

# Payments stream
payments_df = spark.readStream.format("delta").table("payments_stream") \
    .withWatermark("payment_time", "10 minutes")

# Stream-Stream Join with time window
joined_df = orders_df.join(
    payments_df,
    expr("""
        orders_df.order_id = payments_df.order_id AND
        payments_df.payment_time BETWEEN orders_df.order_time AND orders_df.order_time + INTERVAL 15 minutes
    """)
)
```

---

### ✅ Supported Join Types

| Join Type    | Supported |
| ------------ | --------- |
| inner        | ✅         |
| left\_outer  | ✅         |
| right\_outer | ✅         |
| full\_outer  | ❌         |
| cross        | ❌         |

---

## 🧾 Summary: Stream-Static vs. Stream-Stream

| Feature                  | Stream-Static Join        | Stream-Stream Join       |
| ------------------------ | ------------------------- | ------------------------ |
| Inputs                   | 1 stream + 1 static table | 2 live streams           |
| Use case                 | Enrichment (lookup)       | Real-time correlation    |
| Watermark needed         | ❌ Not required            | ✅ Yes (for both streams) |
| Time window needed       | ❌ No                      | ✅ Yes                    |
| Complexity               | ✅ Simple                  | ⚠️ Medium/High           |
| Output mode              | Append/Update             | Append/Update/Complete   |
| Static side refreshable? | ❌ Only on restart         | ❌ Not applicable         |

---

## ✅ When to Use What?

| Scenario                                         | Join Type     |
| ------------------------------------------------ | ------------- |
| Add country name to transaction events           | Stream-Static |
| Match order with payment stream                  | Stream-Stream |
| Add product category to product view stream      | Stream-Static |
| Correlate login with suspicious IPs in real-time | Stream-Stream |

---

## 📌 Key Takeaways

* Use **Stream-Static** when joining with lookup/dimension data
* Use **Stream-Stream** when **both sides are real-time events**
* Stream-Stream needs **event time**, **watermark**, and **time window**
