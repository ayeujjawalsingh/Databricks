# 📘 Databricks SQL – Components Explained

Databricks SQL is a tool inside Databricks that allows you to use **SQL to query, visualize, and analyze data** from your data lake. It's ideal for **data analysts, engineers, and business users** who prefer working with SQL and dashboards instead of code notebooks.

---

## 🔧 Components of Databricks SQL

### 1. ✅ SQL Warehouse (Previously SQL Endpoints)
- The **compute engine** that runs your SQL queries.
- You configure it with CPU, memory, and scaling options.
- Supports **auto-scaling** and **auto-pausing** when idle.
- You must have a warehouse running to execute SQL queries.

> Think of it as the backend machine that powers your SQL.

---

### 2. ✍️ Query Editor
- A built-in editor for **writing and executing SQL queries**.
- Features include:
  - Syntax highlighting
  - Auto-complete
  - Preview query results
- You can run queries and view results immediately.

---

### 3. 📊 Dashboards
- Used to **visualize SQL query results** using charts and graphs.
- Can include:
  - Bar charts, pie charts, line graphs, counters, etc.
- Dashboards can be **shared across teams** for business reporting.

---

### 4. 💾 Saved Queries
- Allows you to **save useful SQL queries** for reuse.
- You can:
  - Re-run queries anytime
  - Schedule queries to run periodically
  - Link them to dashboards or alerts

---

### 5. 🔔 Alerts
- Set up **automated notifications** based on query results.
- Example: 
  > Alert me if daily sales fall below ₹1,00,000.
- Alerts can send **emails or webhook messages**.

---

### 6. 🕓 Query History
- Maintains a **log of all executed SQL queries**.
- Includes:
  - Who ran it
  - When it was run
  - Duration and success/failure
- Helps with **debugging and performance monitoring**.

---

### 7. 📂 Data Explorer
- A graphical tool to **browse databases, tables, and views**.
- Lets users:
  - Preview sample data
  - See schema and data types
  - Auto-generate sample queries

---

### 8. 🧱 Views
- **Virtual tables** created from SQL queries.
- Types of views:
  - Temporary Views (for session)
  - Global Temp Views (across sessions)
  - Managed Views (stored in metastore)
- Useful for abstracting business logic.

---

### 9. 🚀 Materialized Views (Optional)
- Like regular views, but **data is pre-computed and cached**.
- Automatically or manually refreshed.
- Great for improving performance on repeated queries.

---

### 10. 🔣 Functions / UDFs
- Built-in SQL functions like:
  - `SUM()`, `COUNT()`, `DATE_ADD()`, etc.
- Custom logic using:
  - **User Defined Functions (UDFs)** in SQL or Python

---

### 11. 📦 Delta Lake (Storage Layer)
- All queries in Databricks SQL work with **Delta tables**.
- Delta provides:
  - **ACID transactions**
  - **Time travel**
  - **Schema validation**
  - **Faster reads using Delta Logs**

> Delta Lake is the foundation layer for storage and reliability.

---

### 12. 🔐 Access Control and Security
- Role-based access control to manage:
  - Who can read/write tables, queries, dashboards
  - Fine-grained permissions using **Unity Catalog**
- Ensures **secure and compliant** access to data.

---

## ✅ Summary

| Component            | Description                                           |
|----------------------|-------------------------------------------------------|
| SQL Warehouse        | Compute engine for SQL queries                        |
| Query Editor         | UI to write and run SQL queries                       |
| Dashboards           | Visualize query results                               |
| Saved Queries        | Store and reuse SQL queries                           |
| Alerts               | Trigger notifications based on query outcomes        |
| Query History        | Logs of executed queries                              |
| Data Explorer        | Browse schemas, tables, and views                     |
| Views                | Virtual tables from SQL queries                       |
| Materialized Views   | Cached version of views for better performance        |
| Functions/UDFs       | Built-in and custom logic extensions                  |
| Delta Lake           | Reliable storage layer with transaction support       |
| Access Control       | Secure and permissioned access to data                |

---

## 🔍 Use Cases
- Ad-hoc data exploration using SQL
- Business reporting and dashboards
- Automated alerts for data quality and KPIs
- Building semantic layers with views and materialized views

---

## 📌 Pro Tips
- Always pause unused SQL Warehouses to save costs.
- Use Materialized Views for slow or heavy joins.
- Set alerts on critical business metrics.
- Use Unity Catalog for fine-grained access control.

---

