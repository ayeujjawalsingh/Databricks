# 🔐 Databricks Security Overview

This document provides a detailed explanation of various security features in Databricks, including:

- Databricks Security Overview
- Secret Management
- Row-Level Security (RLS)
- Column-Level Security (CLS)
- Dynamic Views
- Unity Catalog-based Governance (Bonus)

---

## 🧱 1. Databricks Security Overview

Databricks provides an **enterprise-grade security model** to protect data, workloads, and infrastructure. It is built with several layers of protection, enabling secure data processing, access management, and compliance.

### 🔑 Key Components of Databricks Security:

| Component        | Description |
|------------------|-------------|
| **Authentication** | Validates user identity using systems like SSO, personal access tokens, or OAuth. |
| **Authorization**  | Controls what users or groups can do with specific resources (tables, notebooks, clusters, etc.). |
| **Data Protection**| Encrypts data both in transit (TLS/HTTPS) and at rest (cloud KMS). |
| **Network Security** | Includes IP access lists, private link support, and secure VPC peering. |
| **Auditing & Monitoring** | Captures logs of user activities and resource usage for security review. |

---

## 🔐 2. Secret Management in Databricks

Secret Management allows you to **store and access sensitive credentials securely**, without exposing them in code or notebooks.

### ✅ Use Cases:
- API Keys for 3rd-party services (e.g., Stripe, AWS)
- Database usernames and passwords
- Private tokens or certificates

### 📁 Secret Scope:
A secret scope is like a secure folder that stores your secrets.

### 🧰 Creating a Secret Scope:
You can create secret scopes using the Databricks CLI:
```bash
databricks secrets create-scope --scope my-scope
```

### 🔑 Adding Secrets to Scope:

```bash
databricks secrets put --scope my-scope --key my-secret-key
```

### 🔎 Accessing Secrets in Notebooks:

```python
dbutils.secrets.get(scope="my-scope", key="my-secret-key")
```

### 🔐 Security Notes:

* Secrets are stored encrypted.
* Secrets are not visible when printed.
* You can set **access control** on secret scopes.

---

## 📊 3. Row-Level Security (RLS)

**Row-Level Security** controls access to **specific rows in a table** based on the user who is querying the data.

### 🔍 Why Use RLS?

To make sure users only see data they are authorized to see, such as:

* A sales manager only sees their region’s data.
* An HR user only sees their department’s records.

### 🧩 Example Using Dynamic View:

```sql
CREATE OR REPLACE VIEW secure_orders AS
SELECT *
FROM orders
WHERE region = current_user_region();  -- Assume this function maps user to region
```

### 🧠 Implementation Strategy:

* Create a mapping of users to access rights (region, department, etc.)
* Apply filtering logic in a **SQL View** or **Dynamic View**

---

## 🧱 4. Column-Level Security (CLS)

**Column-Level Security** restricts access to **specific columns** for different users or groups. This is important for protecting sensitive information such as:

* Salary
* Personal Identification Numbers (SSN, PAN, etc.)
* Health records

### 🛠 Approaches to Implement CLS:

1. **Create separate views** for each user group.
2. **Mask sensitive columns** based on user identity using logic in views.
3. **Use Unity Catalog** to grant fine-grained column-level permissions.

### 📌 Example:

```sql
CREATE OR REPLACE VIEW secure_employees AS
SELECT 
  employee_id,
  name,
  CASE 
    WHEN current_user() = 'admin@databricks.com' THEN salary
    ELSE NULL
  END AS salary
FROM employees;
```

---

## 👁️ 5. Dynamic Views

Dynamic Views are **SQL views that return different results based on who is running the query**. They use context functions like `current_user()` or `is_member()`.

### 🔧 Key Functions:

* `current_user()` – Returns the username of the person running the query.
* `is_member('group_name')` – Returns true if the user is in the specified group.

### ✅ Use Case:

You want a single view that filters or masks data for different users.

### 🧪 Example:

```sql
CREATE OR REPLACE VIEW secure_sales AS
SELECT
  order_id,
  region,
  CASE 
    WHEN is_member('managers') THEN revenue
    ELSE NULL
  END AS revenue
FROM sales;
```

This view shows revenue only to users in the `managers` group.

---

## 📚 6. Unity Catalog Security (Bonus)

Unity Catalog is Databricks' **centralized data governance layer** that provides fine-grained access control and auditing across all workspaces.

### 🔐 Key Features:

* Manage permissions at **catalog**, **schema**, **table**, and **column** level.
* Define **Access Control Lists (ACLs)** for all resources.
* Use **Attribute-Based Access Control (ABAC)** with tags.
* Integrated **data lineage**, showing who accessed what and when.

### 🧱 Unity Catalog Permissions Model:

| Level        | Resource          | Example                                                               |
| ------------ | ----------------- | --------------------------------------------------------------------- |
| Catalog      | `finance_catalog` | `GRANT USAGE ON CATALOG finance_catalog TO group_data_team`           |
| Schema       | `sales_schema`    | `GRANT SELECT ON SCHEMA sales_schema TO analysts`                     |
| Table/Column | `sales.orders`    | `GRANT SELECT(order_id, region) ON TABLE sales.orders TO group_sales` |

---

## ✅ Summary Table

| Security Feature          | Description                | Example                                      |
| ------------------------- | -------------------------- | -------------------------------------------- |
| **Secret Management**     | Store credentials securely | `dbutils.secrets.get(...)`                   |
| **Row-Level Security**    | Filter rows based on user  | `WHERE region = user_region`                 |
| **Column-Level Security** | Hide or mask columns       | `CASE WHEN is_admin THEN salary ELSE NULL`   |
| **Dynamic Views**         | Context-based data access  | `current_user(), is_member()`                |
| **Unity Catalog**         | Central governance         | Catalog/table-level ACLs, column permissions |

---

## 📌 Best Practices

* Avoid hardcoding secrets; always use Secret Scopes.
* Design centralized mappings for RLS (user → region).
* Mask sensitive data instead of restricting full table access.
* Use Unity Catalog for enterprise-grade governance.
* Test views by impersonating different users (if allowed).

---

## 🧠 Final Note

Databricks’ layered security and governance features provide flexibility and fine-grained control over data access. With the help of secret scopes, dynamic views, and Unity Catalog, organizations can securely scale analytics across teams while ensuring data privacy and compliance.

```
