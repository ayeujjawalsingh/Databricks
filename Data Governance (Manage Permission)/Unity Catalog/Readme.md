# 📘 Unity Catalog in Databricks - Data Governance & Permission Management

## 🔰 Introduction

**Unity Catalog** is a unified data governance solution in **Databricks** that allows organizations to **centrally manage permissions, data access policies, lineage, and discovery** across **all data assets** (Delta tables, files, models, dashboards, and more).

It simplifies and standardizes how data access and governance is implemented across **multiple workspaces and cloud providers (AWS, Azure, GCP)**.

---

## 🎯 Why Use Unity Catalog?

Before Unity Catalog, access control in Databricks was limited to per-workspace configurations using legacy Table ACLs. This made governance **complex, inconsistent, and hard to scale**.

### ✅ Unity Catalog provides:
- **Centralized Access Control**: One place to manage data permissions across workspaces.
- **Fine-Grained Permissions**: Control at table, row, and column level.
- **Lineage Tracking**: Track where data came from and where it is used.
- **Unified Discovery**: Organize and explore all data assets.
- **Cross-Cloud and Cross-Workspace Support**: Seamless control across cloud environments.

---

## 🧩 Key Features

### 1. 🛑 Centralized Governance

- Manage **permissions, policies, and data visibility** across all workspaces from a single governance layer.
- Enforces **consistent rules** across departments and projects.

### 2. 🎯 Fine-Grained Access Control

- Define access at the level of:
  - **Catalog**
  - **Schema (Database)**
  - **Table / View / Function**
- Supports SQL-based `GRANT` / `REVOKE` commands.
  
```sql
GRANT SELECT ON TABLE sales_data TO `analyst_group`;
REVOKE INSERT ON TABLE sales_data FROM `interns`;
```

* Example use-cases:

  * Allow only analysts to read a table.
  * Deny update access to interns.
  * Give developers access to only development schema.

---

### 3. 🔎 Data Lineage (Audit Trail)

* Automatically tracks:

  * Data source → Transformation → Output
  * Who accessed the data and when
* Useful for:

  * **Compliance & Auditing**
  * **Debugging Data Pipelines**
  * **Understanding Data Flow**

---

### 4. 📂 Discovery and Cataloging

* Organizes data into a **logical hierarchy**:

  ```
  Metastore
  └── Catalog
      └── Schema (Database)
          └── Tables / Views / Functions
  ```

* Users can:

  * Search datasets like a catalog
  * Easily understand relationships between datasets
  * Share common datasets across teams

---

### 5. 🔐 Row and Column-Level Security *(Advanced)*

* Control access to **specific rows** or **columns** based on user roles or attributes.

**Examples:**

```sql
-- Mask salary column
CREATE MASKING POLICY mask_salary
  AS (val STRING) -> CASE
    WHEN is_accountant() THEN val
    ELSE 'REDACTED'
  END;

-- Filter data by department
CREATE ROW FILTER filter_hr
  AS (department STRING) -> department = 'HR';
```

* Enables **zero-trust architecture** by minimizing data exposure.

---

### 6. 🌍 Cross-Workspace and Cross-Cloud Access

* Share catalogs and tables across:

  * Multiple **Databricks Workspaces**
  * **Cloud platforms** like AWS, Azure, and GCP
* Simplifies data collaboration and eliminates data duplication.

---

### 7. 📁 External Locations & Storage Credentials

* Supports **external cloud storage** like:

  * AWS S3
  * Azure Data Lake Storage (ADLS)
  * Google Cloud Storage (GCS)

* Define secure access with:

  * `External Location`
  * `Storage Credential`

```sql
CREATE EXTERNAL LOCATION sales_data_location
  URL 's3://company-data/sales'
  WITH CREDENTIAL aws_iam_role;
```

---

### 8. 🧱 Unity Catalog Permission Hierarchy

Unity Catalog permissions follow a **top-down inheritance model**:

```
Metastore
└── Catalog (e.g., prod_catalog)
    └── Schema (e.g., customer_db)
        └── Table/View/Function (e.g., orders, users)
```

Each level supports specific privileges:

| Object Type | Supported Privileges                       |
| ----------- | ------------------------------------------ |
| Metastore   | `CREATE CATALOG`, `MANAGE`                 |
| Catalog     | `USAGE`, `CREATE SCHEMA`, `OWN`            |
| Schema      | `USAGE`, `CREATE TABLE`, `MODIFY`          |
| Table/View  | `SELECT`, `MODIFY`, `OWN`, `READ_METADATA` |
| Functions   | `EXECUTE`                                  |

---

### 9. 🛠️ Managing Permissions

You can manage access using:

#### ✅ SQL Commands

```sql
GRANT USAGE ON CATALOG finance TO `finance_team`;
GRANT SELECT ON TABLE finance.transactions TO `auditors`;
REVOKE ALL PRIVILEGES ON TABLE finance.internal_budget FROM `contractors`;
```

#### ✅ Databricks UI

* Go to **Data > Catalog Explorer**
* Right-click on the object (Catalog/Schema/Table)
* Choose **Permissions** and assign roles

#### ✅ Terraform / REST API

* Infrastructure-as-code for automation
* Helps manage roles and privileges at scale

---

## ✅ Benefits Summary

| Feature                         | Benefit                                                |
| ------------------------------- | ------------------------------------------------------ |
| Centralized access control      | Simplifies governance across all data and AI assets    |
| SQL-style permission management | Familiar and developer-friendly                        |
| Full data lineage               | Enhances auditing and trust in data                    |
| Scalable security policies      | Supports enterprise-level security at schema/table/row |
| Cross-cloud support             | Works across AWS, Azure, and GCP                       |
| Unified discovery               | Easy data search and collaboration                     |

---

## 🔚 Conclusion

Unity Catalog is the **core governance layer** in Databricks for managing data securely, consistently, and at scale. It provides a **clean architecture for managing access**, auditing data usage, organizing assets, and complying with privacy requirements.

> Whether you're a data engineer, platform admin, or data analyst — Unity Catalog enables **secure, scalable, and discoverable** data usage.
