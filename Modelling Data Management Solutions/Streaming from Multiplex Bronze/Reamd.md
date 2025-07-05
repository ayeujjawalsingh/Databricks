# 🔄 Streaming from Multiplex Bronze

## 📘 What is Multiplex Bronze?

**Multiplex Bronze** is an advanced design pattern in the **Bronze Layer** where:
- Multiple different source data streams are **ingested into a single Delta table**
- The table holds **raw data from many domains** (e.g., orders, customers, products) together
- A **source identifier** column (e.g., `record_type`) helps distinguish each type

---

## 🎯 Why Use Multiplex Bronze?

| ✅ Benefit              | 📄 Explanation |
|------------------------|----------------|
| Centralized ingestion  | Handle many sources using one pipeline and one table |
| Lower storage cost     | Shared storage, partitioned by type or time |
| Easier schema evolution| Single process to manage schema drift |
| Unified streaming      | One stream for processing all bronze data |
| High throughput        | Scalable for large data pipelines |

---

## 🧱 Table Structure: Multiplex Bronze Delta Table

| Column Name     | Description                        |
|-----------------|------------------------------------|
| `record_type`   | Type of record (e.g., order, user) |
| `data`          | Raw payload (often a struct or JSON) |
| `ingestion_time`| When the record was loaded         |
| `source`        | Source system (e.g., Kafka topic, API name) |
| `metadata`      | Optional – filename, batch ID, etc.|

---

## 🧪 Sample Multiplex Bronze Delta Table (Logical View)

| record_type | data (JSON / struct)              | ingestion_time       | source      |
|-------------|-----------------------------------|-----------------------|-------------|
| "order"     | {"id":123, "amount":200}          | 2025-07-05T11:00:00Z  | "kafka_orders" |
| "customer"  | {"id":456, "name":"Ujjawal"}      | 2025-07-05T11:00:05Z  | "api_customers" |
| "product"   | {"id":789, "price":99.99}         | 2025-07-05T11:00:10Z  | "s3/products"   |

---

## 🔄 Streaming from Multiplex Bronze – How It Works?

### 🧠 Concept

You read the **single multiplex bronze table as a stream**, and **branch** (split) it based on the `record_type` column.

Then:
- Send "orders" to the Silver orders table
- Send "customers" to Silver customers table
- Send "products" to Silver products table

---

## 🧪 Example Code: Streaming from Multiplex Bronze

```python
# Step 1: Read stream from multiplex bronze table
bronze_stream = (spark.readStream
    .format("delta")
    .table("bronze.multiplex_raw"))

# Step 2: Filter different record types
orders_df = bronze_stream.filter("record_type = 'order'") \
    .selectExpr("data.id as order_id", "data.amount", "ingestion_time")

customers_df = bronze_stream.filter("record_type = 'customer'") \
    .selectExpr("data.id as customer_id", "data.name", "ingestion_time")

# Step 3: Write each stream to its own Silver table
orders_df.writeStream \
    .format("delta") \
    .option("checkpointLocation", "/chk/silver/orders/") \
    .table("silver.orders")

customers_df.writeStream \
    .format("delta") \
    .option("checkpointLocation", "/chk/silver/customers/") \
    .table("silver.customers")
```

---

## 🗂️ Directory Layout (Delta Tables)

```plaintext
/delta/
  ├── bronze/
  │    └── multiplex_raw/         <- All raw records in one place
  ├── silver/
  │    ├── orders/                <- Cleaned "order" records
  │    └── customers/             <- Cleaned "customer" records
```

---

## 🧠 Key Use Cases

| Use Case                                             | Example                                   |
| ---------------------------------------------------- | ----------------------------------------- |
| IoT devices sending mixed signals                    | Thermostat + motion + light in one stream |
| Kafka topic with mixed event types                   | Orders, refunds, payments from same topic |
| API gateway writing multiple endpoints to single log | User login, sign-up, purchases            |

---

## ✅ Best Practices

* Always include a `record_type` or `event_type` field to distinguish streams
* Use schema evolution carefully (nested schemas may vary per type)
* Avoid heavy transformation in Bronze – do it in Silver
* Partition by `record_type`, `ingestion_date` to optimize storage
* Monitor stream performance (too many small writes may cause lag)

---

## ⚠️ Challenges & How to Handle

| Challenge                   | Solution                                          |
| --------------------------- | ------------------------------------------------- |
| Different schema per type   | Use `data` as JSON or struct; flatten in Silver   |
| High volume with many types | Use separate jobs for critical record types       |
| Checkpointing failure       | Use unique checkpoint path per record type        |
| Query performance           | Use Z-ORDER or partition pruning on `record_type` |

---

## 📐 Architecture: Streaming from Multiplex Bronze

```plaintext
          [Multiple Sources]
         /        |        \
        v         v         v
   ┌────────┐ ┌────────┐ ┌────────┐
   | Kafka  | | APIs   | | S3     |
   └────────┘ └────────┘ └────────┘
         \       |        /
          v      v       v
     ┌────────────────────────────┐
     | Delta Table: bronze.multiplex_raw |
     └────────────────────────────┘
            |
     [ Read as Stream ]
            |
 ┌────────────┬────────────┬────────────┐
 | Filter "order" | "customer" | "product" |
 └────────────┴────────────┴────────────┘
        |              |             |
     Silver.orders  Silver.customers Silver.products
```

---

## 🧾 Summary

| 🔷 Key Points – Streaming from Multiplex Bronze   |
| ------------------------------------------------- |
| Ingest all types of raw data into one table       |
| Use `record_type` to identify source type         |
| Stream and branch records into Silver tables      |
| Makes ingestion architecture simpler and scalable |
| Best for high-throughput, mixed data environments |

