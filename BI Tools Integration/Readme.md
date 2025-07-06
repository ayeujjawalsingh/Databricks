# 📊 BI Tools Integration with Databricks

This document provides a complete guide to connect BI tools like **Power BI** and **Tableau** with **Databricks** using **JDBC/ODBC Drivers** and **Partner Connect**. These integrations allow business users and analysts to run real-time queries, create dashboards, and visualize data stored in Databricks Lakehouse using interactive tools.

---

## 📌 Table of Contents

- [🔰 Overview](#-overview)
- [🔌 Supported BI Tools & Protocols](#-supported-bi-tools--protocols)
- [🧩 Power BI Integration](#-power-bi-integration)
  - [Method 1: Using Partner Connect](#method-1-using-partner-connect)
  - [Method 2: Using ODBC Driver (Manual)](#method-2-using-odbc-driver-manual)
- [📊 Tableau Integration](#-tableau-integration)
  - [Method 1: Using Partner Connect](#method-1-using-partner-connect-1)
  - [Method 2: Using JDBC Driver (Manual)](#method-2-using-jdbc-driver-manual)
- [⚙️ SQL Warehouse Requirements](#️-sql-warehouse-requirements)
- [🔐 Authentication & Security](#-authentication--security)
- [🧠 Best Practices](#-best-practices)
- [🚑 Troubleshooting Tips](#-troubleshooting-tips)
- [📎 References](#-references)

---

## 🔰 Overview

Databricks supports direct integration with Business Intelligence (BI) tools for:
- Interactive dashboarding
- Real-time reporting
- Self-service analytics
- Visual exploration of lakehouse data

BI tools connect to **Databricks SQL Warehouses**, not clusters. Integration is done via:
- JDBC/ODBC drivers
- Partner Connect (UI-based setup)

---

## 🔌 Supported BI Tools & Protocols

| BI Tool    | Connection Protocol | Driver Used     | Partner Connect Support |
|------------|---------------------|------------------|--------------------------|
| Power BI   | ODBC (Simba)        | Simba ODBC Driver | ✅ Yes                  |
| Tableau    | JDBC (Simba)        | Simba JDBC Driver | ✅ Yes                  |

---

## 🧩 Power BI Integration

### Method 1: Using Partner Connect

Partner Connect offers a no-code, guided way to connect Power BI with Databricks.

#### 🔧 Steps:
1. Open **Databricks Workspace UI**.
2. Go to the **Partner Connect** tab on the left sidebar.
3. Click on **Power BI**.
4. Choose a **SQL Warehouse** to connect.
5. Databricks auto-generates:
   - Hostname
   - Port (443)
   - HTTP Path
   - Access token
6. Power BI Desktop will launch or provide a download link.
7. The connector is pre-configured — just start building dashboards.

> ✅ **Recommended for quick setup** with minimum configuration.

---

### Method 2: Using ODBC Driver (Manual)

Use this when more customization is needed or when Partner Connect is not available.

#### 🔧 Prerequisites:
- Install **Power BI Desktop**
- Download and install **Simba ODBC Driver**:
  [https://www.databricks.com/spark/odbc-driver-download](https://www.databricks.com/spark/odbc-driver-download)
- Generate **Databricks Personal Access Token**

#### 🔧 Steps:
1. Create and start a **SQL Warehouse** in Databricks.
2. Get:
   - **Server Hostname** (`adb-xxxxxxxxx.databricks.com`)
   - **HTTP Path** (from warehouse settings)
3. Open Power BI Desktop.
4. Go to **Get Data → ODBC**.
5. Select the configured **DSN** (ODBC Data Source).
6. Enter your:
   - Server hostname
   - HTTP path
   - Access token
7. Load the table or write SQL query to fetch data.

---

## 📊 Tableau Integration

### Method 1: Using Partner Connect

#### 🔧 Steps:
1. Open **Databricks Workspace**.
2. Navigate to **Partner Connect**.
3. Select **Tableau** from available partners.
4. Choose your **SQL Warehouse**.
5. A Tableau `.tdc` connection file is generated.
6. Tableau Desktop launches with connection details pre-filled.
7. Enter your access token and click **Connect**.

> ✅ Best for Tableau users with minimal technical background.

---

### Method 2: Using JDBC Driver (Manual)

#### 🔧 Prerequisites:
- Install **Tableau Desktop**
- Download **Simba JDBC Driver**:
  [https://www.databricks.com/spark/jdbc-driver-download](https://www.databricks.com/spark/jdbc-driver-download)

#### 🔧 Steps:
1. Launch Tableau Desktop.
2. Choose **More → Databricks** under Connect.
3. Input:
   - **Host**: `adb-xxxxxxxxx.databricks.com`
   - **Port**: `443`
   - **HTTP Path**: From SQL Warehouse
   - **Authentication**: Personal Access Token
4. Click Connect → Choose your catalog/schema/table.
5. Start creating dashboards.

---

## ⚙️ SQL Warehouse Requirements

- Must be **running** before connecting BI tools.
- Databricks **SQL endpoint (warehouse)** should be configured with:
  - Auto-stop timeout (cost control)
  - Adequate cluster size for queries
  - Permissions to access catalogs, schemas, and tables
- Use **Serverless SQL Warehouses** for better performance and scalability.

---

## 🔐 Authentication & Security

### 🔑 Personal Access Token
- Required for JDBC/ODBC-based connections.
- Generate via:
  - Databricks UI → User Settings → Access Tokens → Generate New Token

### 🧾 Required Permissions
- `SELECT` on target tables/views
- Unity Catalog ACLs (if UC is enabled)

---

## 🧠 Best Practices

- ✅ Always connect BI tools to **SQL Warehouses**, not interactive clusters.
- ✅ Use **materialized views** or **Delta Live Tables** for reporting layers.
- ✅ Optimize BI queries using **Z-Ordering** and **Data Skipping**.
- ✅ Leverage **Unity Catalog** for row/column-level access control.
- ✅ Enable **query history** logging for auditing and debugging.
- ✅ Use **auto-stop** on SQL Warehouses to avoid idle costs.
- ✅ Set appropriate **concurrency** limits on the warehouse.
- ✅ Separate **development** and **production** warehouses for isolation.

---

## 🚑 Troubleshooting Tips

| Issue                              | Resolution                                                                 |
|-----------------------------------|----------------------------------------------------------------------------|
| Invalid token/auth error          | Re-generate token, check token expiration                                  |
| No tables or schema visible       | Check user permissions and catalog access                                 |
| Connection refused/timeout        | Ensure SQL Warehouse is running and accessible                            |
| Tableau connection fails          | Verify HTTP path, hostname, port (443), and token                         |
| Poor performance or timeout       | Upgrade SQL Warehouse size; cache or materialize views                    |
