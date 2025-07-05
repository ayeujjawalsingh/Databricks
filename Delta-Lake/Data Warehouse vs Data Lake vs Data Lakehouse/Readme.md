# 📘 Data Warehouse vs Data Lake vs Data Lakehouse

This document explains the core differences between **Data Warehouse**, **Data Lake**, and **Data Lakehouse**, along with how **ETL** and **ELT** work in each. It also covers how tools like **Delta Lake** and **Databricks** connect into the Lakehouse architecture.

---

## 🔹 What is a Data Warehouse?

A **Data Warehouse** is a centralized repository that stores **cleaned and structured data** for **analytics, dashboards, and reporting**.

### ✅ Key Features:
- Stores only **processed/clean** data
- Fast SQL querying performance
- Strict **schema enforcement**
- High cost per GB (usually)

### 🔄 ETL in Data Warehouse:
Data is cleaned and transformed **before** being stored.

**ETL = Extract → Transform → Load**

| Step | Example |
|------|---------|
| Extract | Pull data from source systems (APIs, DBs) |
| Transform | Clean, filter, join, remove nulls |
| Load | Load final data into Data Warehouse (Snowflake, Redshift, BigQuery) |

### 📌 Use Cases:
- Business Intelligence
- Reporting
- Dashboards

---

## 🏞️ What is a Data Lake?

A **Data Lake** is a large, cost-effective storage system that can store **all types of raw data** — structured, semi-structured, and unstructured — in **cloud object storage** like AWS S3, Azure Data Lake Storage, or Google Cloud Storage.

### ✅ Key Features:
- Stores **everything**, raw to refined
- Supports **structured + unstructured** data
- Open file formats like **Parquet**, **JSON**, **CSV**
- Very **cost-effective**

### 🔁 ELT in Data Lake:
Data is stored first and then processed when needed.

**ELT = Extract → Load → Transform**

| Step | Example |
|------|---------|
| Extract | Pull data from various sources |
| Load | Store raw data as-is into data lake |
| Transform | Clean/format data later using tools like Spark |

### 📌 Use Cases:
- Data Science & Machine Learning
- Streaming analytics
- Staging layer for Lakehouse

---

## 🧬 What is a Data Lakehouse?

A **Data Lakehouse** combines the **scalability of Data Lakes** with the **structure and performance of Data Warehouses** using modern open table formats like **Delta Lake**, **Apache Iceberg**, or **Apache Hudi**.

### ✅ Core Benefits:
- ACID transactions
- Schema enforcement and evolution
- Time travel
- Unified batch and streaming
- Fast SQL queries on cloud storage
- Handles both ETL and ELT pipelines

### 🧱 Architecture Layers:

| Layer | Description |
|-------|-------------|
| **Ingestion Layer** | Brings raw data from APIs, databases, files |
| **Storage Layer** | Stores raw data in open formats like Parquet |
| **Delta/Iceberg Layer** | Adds versioning, transactions, and schema |
| **Compute Layer** | Spark, Databricks, Presto query and process data |
| **Query Layer** | BI tools and ML use cleaned data |
| **Consumer Layer** | Analysts, Scientists, Engineers use the data |

---

## 🔗 How Delta Lake, Databricks, and Others Connect

### 🔹 Cloud Storage (Data Lake)
- Base layer using S3, ADLS, or GCS to store raw files (Parquet, JSON)

### 🔹 Delta Lake / Apache Iceberg / Hudi
- Open source table formats that bring **database-like features** to files:
  - ACID transactions
  - Schema enforcement
  - Time travel
  - Indexing for fast reads

### 🔹 Databricks
- A cloud-based platform that natively supports **Delta Lake**
- Built on top of Apache Spark
- Supports:
  - Batch and streaming
  - Notebooks, SQL, ML, Dashboards

### 🔹 Compute Engines
- Apache Spark (used by Databricks, AWS Glue)
- Trino, Presto, Dremio for querying
- Airflow for orchestration

---

## ✅ ETL vs ELT – Summary Table

| Feature | Data Warehouse (ETL) | Data Lake (ELT) | Data Lakehouse (ETL + ELT) |
|--------|-----------------------|------------------|-----------------------------|
| Data Type | Structured only | Any type | Any type |
| Storage | Expensive | Cheap | Cheap |
| Processing | Transform before storing | Store first, transform later | Supports both |
| Query Speed | Very fast | Slower | Fast (Delta, Iceberg) |
| Schema | Fixed | Optional | Flexible |
| Format | Tables | Files (Parquet, JSON) | Tables + Files |
| Tools | Snowflake, Redshift | S3, ADLS | Delta Lake, Iceberg |
| Use Cases | BI, reports | ML, data staging | Unified analytics |

---

## 📘 Real-Life Example:

### Use Case: Building Flight Delay Dashboard

1. **Raw Data** (APIs) is stored into **S3** → ✅ Data Lake
2. Delta Lake adds **versioning & schema** → ✅ Lakehouse
3. Use **Spark (in Databricks)** to clean & transform → ELT pipeline
4. Query using **Databricks SQL** or **Power BI** → For dashboard/report

---

## 📌 Summary

| Concept | Meaning |
|--------|---------|
| **Data Warehouse** | Stores clean data only, using ETL |
| **Data Lake** | Stores everything raw, using ELT |
| **Data Lakehouse** | Combines both worlds with smart table formats |
| **ETL** | Clean data before storing |
| **ELT** | Store data first, clean it later |
| **Delta Lake** | Adds reliability and SQL capability to a data lake |
| **Databricks** | A platform that supports full Lakehouse workflow |

---

## 📎 Related Technologies

| Category | Tools |
|---------|-------|
| Storage | AWS S3, ADLS, GCS |
| Table Format | Delta Lake, Apache Iceberg, Apache Hudi |
| Compute | Apache Spark, Databricks, AWS Glue |
| Query | Databricks SQL, Presto, Trino |
| BI | Power BI, Tableau |
| Orchestration | Apache Airflow |
