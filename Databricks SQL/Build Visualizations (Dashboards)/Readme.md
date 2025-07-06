# 📊 Build Visualizations (Dashboards) in Databricks SQL

Databricks SQL provides a powerful way to create dashboards with visual charts, KPIs, and tables using SQL query results. These dashboards help in **reporting**, **monitoring**, and **business decision-making**.

---

## ✅ Overview

A **Dashboard** is a collection of visualizations (charts, counters, tables) created from one or more SQL queries. It supports **interactive filtering**, **parameter inputs**, and **scheduled sharing**.

---

## 🛠️ Step-by-Step: How to Build a Dashboard

---

### 1. ✍️ Write and Save a SQL Query

- Go to the **SQL Editor**
- Select your **SQL Warehouse**
- Write a SQL query, for example:

```sql
SELECT region, SUM(sales) AS total_sales
FROM orders
WHERE order_date BETWEEN '{{start_date}}' AND '{{end_date}}'
GROUP BY region
ORDER BY total_sales DESC;
```

* Click **Run** to preview the result
* Click **Save** and name your query (e.g., `Sales_By_Region`)

---

### 2. 📋 Create a New Dashboard

* Go to **Dashboards** tab
* Click **Create Dashboard**
* Provide:

  * **Name**: `Monthly Sales Dashboard`
  * **Description** (optional)

---

### 3. 📈 Create a Visualization

* Open the saved query
* Click **+ New Visualization**
* Choose the visualization type:

  * **Table**
  * **Bar Chart**
  * **Line Chart**
  * **Pie Chart**
  * **Counter (KPI)**
  * **Map**
* Customize:

  * X and Y axes
  * Labels, colors, grouping
  * Sorting and filters
* Click **Save Visualization**

---

### 4. 📌 Add the Visualization to Dashboard

* After saving the visualization:

  * Click **Add to Dashboard**
  * Select the dashboard name you created earlier
  * The visual will be pinned as a widget

---

### 5. 🎛️ Add Parameters and Widgets (Optional)

If your query uses parameters like `{{start_date}}` or `{{region}}`, Databricks will prompt you to provide values.

* In the Dashboard:

  * Click **Add Widget**
  * Choose widget type: **Date Picker**, **Dropdown**, **Textbox**
  * Bind each widget to a parameter
* This enables **interactive filtering** of charts on the dashboard

---

### 6. 🧩 Arrange and Customize Layout

* Resize and drag widgets as needed
* Add headers or markdown text blocks
* Group related visuals together
* Use full/half/quarter width for layout clarity

---

### 7. 🔒 Share and Schedule Dashboards

* Click **Share** (top-right)
* Choose who can:

  * **View**
  * **Edit**
* Optional: Schedule **email delivery** of dashboards as PDF or snapshot

---

## 📦 Visualization Types & Use Cases

| Visualization Type | Use Case Example                    |
| ------------------ | ----------------------------------- |
| Bar Chart          | Sales by region                     |
| Line Chart         | Revenue trends over time            |
| Pie Chart          | Order breakdown by status           |
| Counter (KPI)      | Total orders, revenue, active users |
| Table              | Top 10 customers or products        |
| Map                | Sales by geo location               |

---

## 💡 Best Practices

| Practice                  | Benefit                                  |
| ------------------------- | ---------------------------------------- |
| Use parameterized queries | Enable interactive filtering             |
| Keep queries optimized    | Faster dashboard loading                 |
| Use descriptive names     | Easier to organize and maintain          |
| Auto-stop SQL warehouse   | Save cost when dashboards not in use     |
| Schedule email delivery   | Keep stakeholders informed automatically |

---

## ✅ Summary

| Step                     | Action                                   |
| ------------------------ | ---------------------------------------- |
| Write SQL Query          | Save and reuse the query                 |
| Create Visualization     | Use charts, tables, KPIs from query      |
| Create Dashboard         | Group multiple visualizations together   |
| Add Widgets (Parameters) | Enable runtime filters for interactivity |
| Share & Schedule         | Email reports and control access         |

---

## 📌 Example Scenario

**Dashboard Name**: `Executive Sales Dashboard`

* Query 1: `Sales by Region` → Bar Chart
* Query 2: `Daily Sales` → Line Chart
* Query 3: `Total Revenue` → Counter KPI
* Parameters: `start_date`, `end_date`, `region` → bound to widgets
* Shared with: Sales Team and Executives
* Schedule: Email every Monday at 8 AM
