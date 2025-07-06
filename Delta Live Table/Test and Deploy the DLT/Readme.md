# 📦 Delta Live Tables - How to Test and Deploy

This guide explains how to **test** and **deploy** Delta Live Tables (DLT) pipelines in Databricks using simple steps.

---

## 🧪 Step 1: How to Test Delta Live Tables

Before using a pipeline in production, always test it using **Development Mode**.

### 🔧 1. Create a Pipeline in Development Mode

Go to:
```

Workspace > Delta Live Tables > Create Pipeline

```

Fill the form with:

- **Name**: `my-pipeline-dev`
- **Notebook Path**: Where your DLT SQL/Python notebook is saved
- **Storage Location**: Temporary path like `/mnt/test-dlt-output`
- **Target Schema**: A testing database like `dev_dlt_db`
- **Pipeline Mode**: Select **Development**

✅ Development mode helps you:
- Test changes without affecting real data
- Use sample/small data
- Quickly debug and re-run

---

### ▶️ 2. Run the Pipeline

Click **Start** on the DLT UI.

You’ll see a **graph view** showing the flow of your tables.

Watch for:
- ❌ Errors
- ✅ Successful steps
- 📉 Dropped rows (if you added data quality checks)

---

### 🔍 3. Check and Validate Output

Use SQL to view the test output:

```sql
SELECT * FROM dev_dlt_db.clean_orders;
````

Make sure:

* Data is cleaned and transformed correctly
* No broken rows are included
* Your logic (filters, joins, etc.) is working as expected

---

## 🚀 Step 2: How to Deploy Delta Live Tables

Once your testing is done, you can safely deploy it using **Production Mode**.

### ⚙️ 1. Create a Pipeline in Production Mode

Create a new pipeline with:

* **Name**: `my-pipeline-prod`
* **Notebook Path**: Same DLT notebook as test
* **Storage Location**: `/mnt/prod-dlt-output`
* **Target Schema**: `prod_dlt_db`
* **Pipeline Mode**: Select **Production**

🔒 Production mode is:

* More stable and reliable
* Ideal for real, scheduled data processing

---

### 🕒 2. Schedule the Pipeline (Optional)

* You can **run manually**, or
* Set up a **schedule** (e.g., every 30 mins), or
* Trigger it using **Databricks Jobs** or **REST API**

---

### 📊 3. Monitor the Pipeline

After deployment, Databricks shows a **monitoring UI** with:

| Tool             | What It Shows                    |
| ---------------- | -------------------------------- |
| 🧬 Lineage Graph | Flow between DLT tables          |
| 📝 Run Logs      | Status, error logs, duration     |
| 🛡️ Data Quality | Dropped rows and reasons         |
| 📈 Metrics       | Rows processed, time taken, etc. |

---

## 💡 Best Practices

| ✅ Do This                   | 🔍 Why It Helps                                        |
| --------------------------- | ------------------------------------------------------ |
| Use Dev Mode First          | To test logic and catch errors early                   |
| Separate Dev & Prod Schemas | Prevents accidental overwrite of real data             |
| Use `cloud_files()`         | Helps with streaming and auto-loading new data         |
| Add Constraints             | Cleans bad data before going to next table             |
| Use Git for Versioning      | Helps track changes in DLT logic                       |
| Monitor After Deployment    | Ensure data is flowing correctly and nothing is broken |

---

## 🧪 Example: Add a Constraint for Data Quality

```sql
CREATE LIVE TABLE valid_orders
  CONSTRAINT valid_id EXPECT (order_id IS NOT NULL) ON VIOLATION DROP ROW
AS
  SELECT * FROM live.clean_orders;
```

* This rule removes any row where `order_id` is empty
* Dropped rows are logged in the UI for tracking

---

## ✅ Summary

* 🔁 Use **Development Mode** to test pipelines safely
* 🚀 Use **Production Mode** for scheduled, reliable runs
* 🔍 Monitor the pipelines using the DLT UI
* 🛡️ Add data rules to make your pipeline more robust
* 📂 Separate environments for dev and prod are a must

> DLT makes your data pipeline development easier, safer, and more reliable — just test properly before deploying! ✅
