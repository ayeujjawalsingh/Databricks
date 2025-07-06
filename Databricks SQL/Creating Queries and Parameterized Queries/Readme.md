# 📘 Creating Queries and Parameterized Queries in Databricks SQL

Databricks SQL allows users to write SQL queries to explore and analyze data using the web-based Query Editor. You can also create **parameterized queries**, which are dynamic and reusable with different inputs — ideal for dashboards and reports.

---

## ✍️ 1. Creating SQL Queries in Databricks SQL

You can create SQL queries using the **Query Editor**. This is the main interface where users can write, execute, and save SQL commands.

### 🔹 Steps to Create a Query

1. Open **Databricks SQL** workspace.
2. Go to the **"SQL Editor"** tab.
3. Select a **SQL Warehouse** to run the query.
4. Choose the database/schema from the left panel.
5. Write your SQL query (e.g., `SELECT * FROM sales_data LIMIT 10`).
6. Click **Run** to execute.
7. Click **Save** to store the query for future use.

---

### 💾 Saving and Organizing Queries

- **Name** your query meaningfully (e.g., `Top_Customers_Last_30_Days`)
- You can assign **tags or folders** for better organization.
- Queries can be shared or kept private.

---

### 🧠 Features of the Query Editor

- Auto-suggestions and SQL syntax highlighting
- Result preview with filters and download options
- Option to export results as CSV, JSON
- Query execution plans for performance tuning

---

## 📦 2. Parameterized Queries

A **parameterized query** lets you pass input values into your SQL dynamically.

> 🔁 Example:  
> Instead of hardcoding a region like `WHERE region = 'North'`,  
> you can use a **parameter** like `{{region}}`.

---

### 🔹 Why Use Parameterized Queries?

- Makes queries **dynamic and reusable**
- Great for **dashboards** where user input is needed (like dropdowns)
- Avoids duplicating similar queries with different filters

---

### 🔹 How to Define Parameters

You define parameters in SQL using double curly braces:

```sql
SELECT *
FROM orders
WHERE region = '{{region}}'
  AND order_date >= '{{start_date}}'
  AND order_date <= '{{end_date}}'
```

> 🔸 These values will be asked **at runtime** or **configured in dashboards**.

---

### 🔹 Supported Parameter Types

| Type        | Example Value   |
| ----------- | --------------- |
| **String**  | `'North'`       |
| **Number**  | `1000`          |
| **Date**    | `'2024-01-01'`  |
| **Boolean** | `true`, `false` |

---

### 🧪 Parameter Input Options

* Input can be provided **manually** when running the query.
* If used inside a dashboard:

  * You can create **widgets** (dropdown, textbox, date picker)
  * Set **default values** or **user input**

---

## 🧠 Notes on Using Parameters

* Always enclose string/date values in `'{{parameter}}'`
* Test the query by manually entering parameter values
* Avoid using unquoted parameters directly in sensitive queries (SQL injection prevention)

---

## ✅ Example Use Case

```sql
SELECT customer_id, total_amount
FROM sales
WHERE order_date BETWEEN '{{start_date}}' AND '{{end_date}}'
  AND region = '{{region}}'
  AND total_amount > {{min_total}}
```

* `start_date`, `end_date` → Date picker
* `region` → Dropdown of regions
* `min_total` → Numeric input

---

## 🧩 Combine With Dashboards

* Use parameterized queries in dashboards to create **interactive filters**.
* When a user selects a filter (e.g., region = North), it automatically updates all widgets using that query.

---

## 🔐 Security Tip

Avoid allowing direct user input in **raw SQL fragments** (like table names or SQL expressions). Always validate or sanitize inputs if dynamic.

---

## 🧠 Best Practices

| Practice                         | Reason                                |
| -------------------------------- | ------------------------------------- |
| Use descriptive parameter names  | Improves readability and maintenance  |
| Set default values               | Prevents query failure on empty input |
| Use with dashboards              | Enables interactive reporting         |
| Avoid dynamic SQL where possible | For safety and performance reasons    |

---

## 📌 Summary

| Feature             | Description                                     |
| ------------------- | ----------------------------------------------- |
| Query Editor        | UI to write, run, and save SQL queries          |
| Saved Query         | Reusable query stored in workspace              |
| Parameterized Query | Query with input variables (e.g., `{{region}}`) |
| Dashboard Inputs    | Bind parameters to widgets for interactivity    |
