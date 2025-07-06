# 📘 Databricks Tooling: Orchestration and Scheduling with Multi-Task Jobs

This document covers the complete overview of **orchestrating and scheduling workflows** using **Multi-Task Jobs** in **Databricks**, written in simple and professional terms for engineering and data teams.

---

## 🧭 What is Orchestration?

**Orchestration** in Databricks refers to managing and controlling the flow of tasks (like a pipeline) — making sure steps such as ingestion, transformation, and reporting run in the correct order with proper error handling and dependencies.

---

## 📅 What is Scheduling?

**Scheduling** is the process of automatically running workflows or jobs **based on time-based triggers** (e.g., hourly, daily, weekly) using built-in scheduling options.

---

## 🔗 What are Multi-Task Jobs?

A **Multi-Task Job** is a **workflow feature** in Databricks that allows you to run **multiple tasks** (e.g., notebooks, scripts, queries) in a single job with **task dependencies**, parallelism, and retries.

It creates a **DAG (Directed Acyclic Graph)** where you define **which task depends on which**.

---

## ✅ Key Features of Multi-Task Jobs

| Feature               | Description                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| Task Dependencies      | Control the execution order using DAG structure                             |
| Multiple Task Types    | Supports notebooks, Python scripts, JARs, SQL, DLT                          |
| Shared or New Clusters | Choose between sharing a job cluster or creating new per task               |
| Parameters             | Pass values into notebooks/scripts using widgets or config                  |
| Retries & Alerts       | Automatic retry on failures and notifications via email/webhook             |
| Scheduling             | Cron or fixed interval schedules                                            |
| Monitoring UI          | Visual DAG view to track status of each task                                |

---

## 🛠️ Supported Task Types

| Task Type        | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| Notebook         | Runs Databricks notebooks with optional parameter input                     |
| Python Script    | Executes standalone `.py` files on cluster                                  |
| JAR              | Runs Scala/Java applications packaged in JAR format                         |
| SQL              | Executes SQL queries using SQL Warehouse or clusters                        |
| Delta Live Table | Triggers DLT pipelines as part of job workflow                              |

---

## 🧱 Example: Multi-Task ETL Flow

```text
Task 1: Ingest Raw Data (Notebook) 
   ↓
Task 2: Clean & Transform (Python Script)
   ↓
Task 3: Load to Gold Table (SQL)
   ↓
Task 4: Refresh Dashboard (Notebook)
```

Each task only runs **after the previous one completes successfully**.

---

## ⚙️ Creating a Multi-Task Job (Step-by-Step)

1. Go to **Jobs** tab in Databricks workspace.
2. Click on **“Create Job”**.
3. Provide a **Job Name**.
4. Click **“+ Add Task”** and configure:

   * Task Name
   * Task Type (Notebook, SQL, Python, JAR, etc.)
   * Cluster (select shared or new)
   * Parameters (optional)
5. Add more tasks and define **dependencies** (e.g., Task B runs after Task A).
6. Define **Schedule** under “Schedule” tab:

   * UI-based or Cron Expression
7. Optionally configure:

   * **Retries** on failure
   * **Timeouts**
   * **Alerts** (email, webhook)
8. Save the job and click **“Run Now”** or wait for scheduled trigger.

---

## ⏰ Scheduling Options

Databricks allows **time-based scheduling** using either:

* Simple UI (e.g., daily, hourly)
* **Quartz Cron Expressions**

### Cron Examples:

| Cron Expression | Description           |
| --------------- | --------------------- |
| `0 0 * * * ?`   | Run every hour        |
| `0 0 0 * * ?`   | Run daily at midnight |
| `0 0 12 * * ?`  | Run daily at noon     |

---

## 🧠 Important Concepts

| Concept               | Description                                              |
| --------------------- | -------------------------------------------------------- |
| **Task Dependencies** | Define run order using DAG (A → B → C)                   |
| **Parameters**        | Use widgets or CLI to pass values to notebook or scripts |
| **Retries**           | Set retry count on failure (e.g., 3 retries)             |
| **Timeouts**          | Set max duration for a task to avoid hanging jobs        |
| **Shared Cluster**    | Use the same cluster across tasks for efficiency         |
| **Run As Owner**      | Option to run the job as owner or triggering user        |
| **Alerts**            | Setup notifications for failure, success, or completion  |

---

## 📊 Monitoring and Logs

Databricks provides a **real-time monitoring UI**:

* Visual **DAG View** to see progress
* Each task has its own **log output**
* Shows retries, duration, and success/failure
* Allows **re-run of failed tasks** only
* Full access to **stderr/stdout**, and debug output

---

## 🧩 Integration with Other Tools

| Integration               | Use Case                                            |
| ------------------------- | --------------------------------------------------- |
| **REST API**              | Automate job creation/runs via external tools       |
| **CLI (databricks jobs)** | Trigger jobs in CI/CD pipelines                     |
| **Terraform**             | Infrastructure as code for managing job definitions |
| **Git Integration**       | Version control for notebooks used in tasks         |

---

## ✅ Best Practices

| Practice                             | Why?                                         |
| ------------------------------------ | -------------------------------------------- |
| Use **shared cluster**               | Reduces cost and startup time                |
| **Modularize jobs**                  | Break big flows into smaller reusable jobs   |
| **Parameterize everything**          | Reusability across environments and datasets |
| Enable **alerts**                    | Get notified on failures                     |
| Set **timeouts & retries**           | Avoid runaway or flaky executions            |
| Use **version-controlled notebooks** | Better audit and collaboration               |

---

## 🚀 Summary

| Topic          | Summary                                                 |
| -------------- | ------------------------------------------------------- |
| Orchestration  | Control the order and flow of tasks in a workflow       |
| Scheduling     | Automate job runs using cron or time-based triggers     |
| Multi-Task Job | Create jobs with multiple, dependent, or parallel tasks |
| Monitoring     | Visual UI, logs, retry info, and re-run options         |
| Integration    | Works with API, CLI, Terraform, Git                     |
