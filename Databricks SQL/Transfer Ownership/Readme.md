# 🔄 Transfer Ownership in Databricks SQL

Ownership in Databricks SQL controls **who has full control over a resource**, including editing, deleting, sharing, and managing permissions.

You may need to **transfer ownership** of SQL objects such as:
- Queries
- Dashboards
- Alerts
- Warehouses

This is common during **team changes**, **offboarding**, or **handoffs** to a new project owner.

---

## ✅ Who Can Transfer Ownership?

You must meet **one** of the following:
- Be the **current owner** of the object
- Be a **workspace admin**
- Have **CAN MANAGE** permission on that object

---

## 🛠️ Transfer Ownership: Step-by-Step

### 🔹 1. Go to the Object

- Navigate to:
  - **Saved Query**
  - **Dashboard**
  - **SQL Alert**
  - **SQL Warehouse**
- Open the object page

---

### 🔹 2. Open Permissions Settings

- Click the **three-dot menu** (⋮) or **"Share"** button
- Select **Permissions** or **Manage Permissions**

---

### 🔹 3. Transfer Ownership

- In the permission panel:
  - Find the **"Owner"** section
  - Click on the **edit icon** ✏️ next to the current owner
- Select a **new user or group** from the dropdown
- Click **Transfer Ownership**

---

## 🔐 What Happens After Transfer?

- The **new owner** gets full control of the object
- The **old owner** becomes a normal user (unless granted additional permissions)
- Ownership change is **audited** in workspace logs (if enabled)

---

## 📦 Transfer Ownership for These Object Types

| Object Type       | Ownership Transfer Supported? |
|-------------------|-------------------------------|
| Saved Query       | ✅ Yes                         |
| Dashboard         | ✅ Yes                         |
| SQL Alert         | ✅ Yes                         |
| SQL Warehouse     | ✅ Yes                         |
| Table (Unity Catalog) | ✅ Yes, using SQL `ALTER`  |
| Notebook          | ✅ Yes                         |

---

## ✨ Bonus: Transfer via SQL (Unity Catalog)

For Unity Catalog objects (tables, views, etc.), use SQL:

```sql
ALTER TABLE catalog.schema.table
OWNER TO `new_owner_email@databricks.com`;
```

* Works for:

  * Tables
  * Views
  * Schemas
  * Catalogs

---

## ✅ Summary Table

| Step             | Description                                   |
| ---------------- | --------------------------------------------- |
| Open Permissions | Go to the object and open permission settings |
| Choose New Owner | Select a new user or group                    |
| Confirm Transfer | Apply the change and save                     |

---

## 💡 Best Practices

| Best Practice                            | Why It Matters                         |
| ---------------------------------------- | -------------------------------------- |
| Always assign clear ownership            | Prevent orphaned resources             |
| Use groups for ownership (when possible) | Easier to manage permissions over time |
| Reassign during offboarding              | Ensures smooth project handoff         |
| Keep audit logs enabled                  | For traceability of ownership changes  |

---

## 📌 Example Scenario

**Problem**: A team member leaves the company, but owns dashboards & alerts.

**Solution**:

1. Workspace admin opens each resource.
2. Reassigns ownership to new team lead.
3. Updates alert emails or dashboard permissions.

---
