# 🚨 Defining SQL Alerts in Databricks SQL

**SQL Alerts** in Databricks SQL allow you to **monitor query results** and **trigger notifications** (email or webhook) when certain conditions are met.

Alerts help automate monitoring for **business KPIs**, **data quality checks**, and **critical failures** without manual effort.

---

## ✅ What is a SQL Alert?

A **SQL Alert** is tied to a **saved SQL query** that runs on a schedule.  
If the result of that query meets a certain **condition**, the alert is triggered and **notifications are sent**.

---

## 📌 Example Use Cases

| Use Case                               | Condition                             |
|----------------------------------------|----------------------------------------|
| Alert when daily sales drop below ₹1L  | `total_sales < 100000`                |
| Alert on data pipeline failure         | `error_count > 0`                      |
| Alert when new records are not ingested | `record_count = 0`                    |
| Alert when inventory is low            | `stock_level < 10`                     |

---

## 🛠️ Step-by-Step: How to Define a SQL Alert

---

### 1. 📝 Write and Save a SQL Query

- Go to the **SQL Editor**
- Write a query that returns **one row and one column**
  - Example:

```sql
SELECT SUM(sales) AS total_sales
FROM orders
WHERE order_date = current_date();
```

* Click **Run** and verify result is numeric or boolean
* Click **Save** and name the query (e.g., `Daily_Sales_Today`)

---

### 2. 🔔 Create an Alert

* Open the **saved query**
* Click on **"Alerts"** → **Create Alert**
* Configure the following:

#### 🔹 Alert Name

* Example: `Sales Drop Alert`

#### 🔹 Condition

* Choose a comparison rule:

  * `is less than`, `is equal to`, `is greater than`, etc.
* Set a threshold value

  * Example: `total_sales is less than 100000`

#### 🔹 Notification Channels

* Add **email addresses**
* (Optional) Add a **webhook URL** (for Slack, PagerDuty, etc.)

#### 🔹 Schedule

* Set **how often to run the query**:

  * Daily, Hourly, Custom Cron

---

### 3. 📬 Test and Activate the Alert

* Click **Test Alert** to simulate
* Click **Save Alert**
* Done! Alert will now monitor results on schedule and notify when triggered

---

## 🧠 How Alert Evaluation Works

* The query runs based on schedule
* The **first column, first row** value is evaluated
* If the condition is met → alert is triggered
* If not → alert does nothing

---

## 🔐 Permissions

* You must have:

  * Permission to **run the query**
  * Permission to **view or edit alerts**
* Admins can restrict who receives or manages alerts

---

## 🧪 Example Alert Conditions

```sql
SELECT COUNT(*) AS error_count
FROM etl_errors
WHERE error_date = current_date();
```

* Condition: `error_count > 0`
* Sends alert if any error occurred today

---

## ✅ Summary Table

| Component     | Description                            |
| ------------- | -------------------------------------- |
| Query         | Must return one row and one column     |
| Condition     | Numeric or Boolean condition (>, <, =) |
| Schedule      | How often to check the condition       |
| Notifications | Email, Webhook                         |
| Trigger       | When condition matches query result    |

---

## 💡 Best Practices

| Best Practice                   | Why Important                             |
| ------------------------------- | ----------------------------------------- |
| Return a single value           | Alerts only work with one-row, one-column |
| Name queries and alerts clearly | Easier maintenance and auditability       |
| Use meaningful thresholds       | Avoid spam and false positives            |
| Test before enabling            | Ensure the condition behaves as expected  |
| Use webhooks for automation     | Integrate with Slack, PagerDuty, etc.     |

---

## 📌 Real Example: Inventory Alert

```sql
SELECT MIN(stock_qty) AS lowest_stock
FROM product_inventory;
```

* Alert: `lowest_stock < 10`
* Notifies supply team to refill inventory
