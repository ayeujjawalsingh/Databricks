# 🧰 Databricks SQL CLI – Command Line Interface

The **Databricks SQL CLI** (Command Line Interface) is a terminal-based tool that allows you to **run SQL queries**, **interact with SQL warehouses**, and **manage SQL workflows** in Databricks — all from your local machine or scripts.

---

## ✅ Why Use Databricks SQL CLI?

- Automate query execution in CI/CD pipelines
- Run ad-hoc queries without opening the UI
- Integrate with scripts for alerts, reports, or ingestion jobs
- Useful for headless operations (e.g., cronjobs, batch jobs)

---

## 🚀 Installation

Databricks SQL CLI is a **Python-based package**. Install it using `pip`.

```bash
pip install databricks-sql-cli
```

> 🔎 Python 3.8+ is recommended.

---

## 🛠️ Configuration (Databricks SQL CLI)

You must configure access to your Databricks workspace and SQL warehouse.

### 🔹 Step 1: Generate a Personal Access Token

* Go to Databricks UI → **User Settings** → **Access Tokens**
* Generate a new token and copy it.

### 🔹 Step 2: Set Up Profile Using CLI

```bash
databricks-sql configure --profile my-profile
```

Provide the following when prompted:

```
Databricks Host: https://<your-databricks-instance>
Databricks Token: <paste-your-token>
HTTP Path: /sql/1.0/warehouses/<warehouse-id>
```

> 💡 You can find the HTTP Path from SQL Warehouse settings.

---

## ⚙️ Running Queries

### 🔹 Basic Query

```bash
databricks-sql query "SELECT COUNT(*) FROM sales" --profile my-profile
```

### 🔹 Query from File

```bash
databricks-sql < my-query.sql --profile my-profile
```

---

## 📄 Common CLI Commands

| Command                        | Description                              |         |                         |
| ------------------------------ | ---------------------------------------- | ------- | ----------------------- |
| `databricks-sql configure`     | Configure CLI with host, token, and path |         |                         |
| `databricks-sql query "<SQL>"` | Run inline query                         |         |                         |
| `databricks-sql < file.sql`    | Run SQL script from file                 |         |                         |
| \`--output json                | csv                                      | table\` | Format the output style |
| `--profile <profile-name>`     | Specify config profile to use            |         |                         |

---

## 🧪 Output Formats

You can customize the output:

```bash
databricks-sql query "SELECT * FROM sales LIMIT 5" \
  --output table \
  --profile my-profile
```

Options:

* `table` (default)
* `json`
* `csv`

---

## 🔐 Profiles & Credentials Storage

* Profiles are saved in `~/.databricks-sql/config`
* Multiple environments (dev, staging, prod) can be managed with separate profiles

---

## 🧠 Tips & Best Practices

| Tip                                | Why It Helps                        |
| ---------------------------------- | ----------------------------------- |
| Use `--output json` for scripting  | Easy parsing in shell scripts       |
| Use multiple profiles              | Switch between environments easily  |
| Store queries in `.sql` files      | Improves readability and versioning |
| Automate with cron or CI pipelines | Schedule jobs without UI            |

---

## ✅ Summary

| Feature           | Description                            |
| ----------------- | -------------------------------------- |
| Installation      | `pip install databricks-sql-cli`       |
| Auth              | Personal access token + HTTP path      |
| Query Execution   | Inline or file-based SQL execution     |
| Output Formats    | table, JSON, CSV                       |
| Use Case Examples | Ad-hoc queries, monitoring, automation |

---

## 📌 Example Use Case: Run Daily Sales Summary

```bash
databricks-sql query "SELECT SUM(sales) FROM orders WHERE order_date = current_date()" \
  --output table \
  --profile prod
```

Schedule this using a cronjob to email or log daily revenue.

