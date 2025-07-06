# 🛡️ Unity Catalog Governance - Users, Groups, Access Control, and Delta Sharing

---

## 👤 Manage Users and Groups in Unity Catalog

Unity Catalog integrates tightly with **identity providers (IdPs)** like **Azure Active Directory (AAD)**, **Okta**, or **SCIM-compatible providers** to manage **users and groups**.

### ✅ Key Concepts

- **Users**: Individual identities (like analysts, engineers).
- **Groups**: Collections of users (like `data_science_team`, `finance_team`).
- **Service Principals**: Non-human identities used for automation/scripts.

### 🔄 How Users and Groups Are Managed

| Platform | Description |
|----------|-------------|
| Azure Databricks | Uses **Azure AD** for user/group sync |
| AWS/GCP Databricks | Uses **SCIM APIs** or **Okta** to sync users/groups |
| Unity Catalog | Inherits users/groups from **Account-level Identity Federation** |

> Unity Catalog does **not store users/groups itself**. It **reads them from Databricks account**.

### 🛠️ Common User Management Tasks

#### 1. **Assign User to Group**
- Done in your Identity Provider (e.g., Azure AD).
- Groups are synced automatically into Databricks.

#### 2. **List All Groups**
In Databricks UI:  
`Admin Console → Groups`

#### 3. **Use Groups in SQL Permissions**
```sql
GRANT SELECT ON TABLE finance.revenue TO `finance_team`;
```

#### 4. **Use Service Principals for Automation**

* Create service principals with limited permissions
* Used in CI/CD, external tools, pipelines

---

## 🔐 Access Control in Unity Catalog

Access control is the heart of **data governance**. Unity Catalog provides **fine-grained access control** using SQL-style `GRANT` and `REVOKE`.

### 🎯 Types of Access Control

| Type                          | Description                              |
| ----------------------------- | ---------------------------------------- |
| Catalog-level                 | Access to use or create schemas          |
| Schema-level                  | Access to create tables/views            |
| Table/View-level              | Access to read, modify, or manage tables |
| Function-level                | Permission to execute UDFs               |
| Row/Column-level *(Advanced)* | Filter or mask data based on identity    |

---

### 🧱 Unity Catalog Permission Hierarchy

```
Metastore
└── Catalog
    └── Schema
        └── Table / View / Function
```

Each level can have specific privileges:

| Object     | Common Privileges                   |
| ---------- | ----------------------------------- |
| Catalog    | `USAGE`, `CREATE SCHEMA`            |
| Schema     | `USAGE`, `CREATE TABLE`, `MODIFY`   |
| Table/View | `SELECT`, `MODIFY`, `READ_METADATA` |
| Function   | `EXECUTE`                           |

---

### ⚙️ How to Assign Access

#### ✅ SQL Examples

```sql
-- Give usage access on catalog
GRANT USAGE ON CATALOG sales_catalog TO `analyst_group`;

-- Grant read access to a table
GRANT SELECT ON TABLE sales_catalog.q1.revenue TO `analyst_group`;

-- Revoke all privileges
REVOKE ALL PRIVILEGES ON TABLE sensitive_data FROM `temp_users`;
```

#### ✅ Databricks UI

* Go to **Data > Catalog Explorer**
* Right-click → **Permissions**
* Add users/groups and assign privileges

#### ✅ Automation with Terraform / API

* Recommended for large orgs
* Declarative permission setup

---

## 🔄 Delta Sharing (Secure Data Sharing Across Platforms)

Delta Sharing is **an open protocol** developed by Databricks to **securely share data across platforms, clouds, and even outside of Databricks** — without data duplication.

---

### 🌍 What Is Delta Sharing?

Delta Sharing allows you to:

* **Share live Delta tables** with external organizations
* **Consume shared data** in non-Databricks tools (e.g., Pandas, Power BI)
* **Avoid exporting data as files** (e.g., CSV/Parquet)

> Think of it as **“data API” access to Delta tables** without moving the data.

---

### 🔧 How Delta Sharing Works

1. You define **Shares** in Unity Catalog.
2. Add specific **tables** to the share.
3. Add **recipients** (other Databricks accounts or external users).
4. Recipient gets **read-only access** via a secure token.

---

### 🧱 Key Objects in Delta Sharing

| Object        | Description                                      |
| ------------- | ------------------------------------------------ |
| **Share**     | Logical container for sharing one or more tables |
| **Recipient** | External user or system receiving the data       |
| **Table**     | The Delta table you are sharing                  |

---

### 🛠️ Delta Sharing - SQL Setup Example

```sql
-- Create a share
CREATE SHARE monthly_sales;

-- Add a table to the share
ALTER SHARE monthly_sales ADD TABLE sales_catalog.q1.transactions;

-- Create a recipient
CREATE RECIPIENT partner_a USING IDENTITY '<public-key>' -- or 'TOKEN'

-- Grant the share
GRANT USAGE ON SHARE monthly_sales TO RECIPIENT partner_a;
```

---

### 📤 Delta Sharing Use Cases

* Share data with:

  * Partners or vendors
  * External auditors
  * Teams on other clouds

* Enable:

  * **Real-time access** without copies
  * **Secure, governed, read-only access**
  * **Interoperability with BI tools** using native connectors

---

### 📦 Tools That Support Delta Sharing

* **Databricks** (native)
* **Pandas / Python / Spark**
* **Power BI, Tableau** via connectors
* **Non-Databricks Delta Sharing clients**

---

## ✅ Summary

| Feature            | Manage Users/Groups       | Access Control      | Delta Sharing          |
| ------------------ | ------------------------- | ------------------- | ---------------------- |
| Purpose            | Organize identities       | Control data access | Share data securely    |
| Mechanism          | SCIM or Identity Provider | SQL GRANT/REVOKE    | Delta Sharing Protocol |
| Access Granularity | Group/User level          | Object & row/column | Table-level sharing    |
| Cross-platform     | Yes                       | Yes                 | Yes                    |
| Automation Ready   | Yes (SCIM, Terraform)     | Yes (Terraform, UI) | Yes (REST API / UI)    |
