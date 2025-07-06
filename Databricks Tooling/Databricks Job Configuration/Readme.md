# ⚙️ Databricks Job Configuration

This document explains all the **configuration options available when creating a Databricks job**, including setup for execution, cluster usage, parameters, retries, notifications, and permissions.

---

## 📘 What is a Job in Databricks?

A **Databricks Job** is a way to **automate the execution** of a task like a notebook, Python script, JAR, SQL query, or Delta Live Table. Jobs can be **single-task** or **multi-task** workflows.

---

## 🧩 Job Configuration Overview

When creating or editing a job, Databricks allows you to configure the following components:

- **Job Name**
- **Tasks** (Notebook, Python, SQL, JAR, DLT)
- **Cluster Settings**
- **Parameters**
- **Schedule**
- **Timeouts and Retries**
- **Notifications**
- **Permissions**
- **Git Version Control**
- **Run As Identity**

---

## 1. 🏷️ Job Name

- A unique **identifier for your job**.
- Helps you distinguish between environments (e.g., `etl-prod`, `etl-dev`).

---

## 2. 📚 Tasks Configuration

For each task in a job (especially multi-task jobs), you configure:

| Option         | Description                                                             |
|----------------|-------------------------------------------------------------------------|
| **Task Name**   | Unique name for the task                                                |
| **Type**        | Notebook / Python / JAR / SQL / DLT                                     |
| **Path**        | File path to notebook/script/query or pipeline ID                      |
| **Parameters**  | Pass inputs via widgets, arguments, or key-value pairs                  |
| **Cluster**     | Choose between new cluster or existing shared job cluster               |
| **Dependencies**| Select if task should run after another (for multi-task workflows)      |

---

## 3. 🖥️ Cluster Settings

Choose how your job runs in terms of computing resources.

| Option                     | Description                                                                          |
|----------------------------|--------------------------------------------------------------------------------------|
| **New Job Cluster**         | Create a new cluster for each job/task execution                                    |
| **Existing Interactive Cluster** | Run job on an already running interactive cluster (less preferred for prod)       |
| **Shared Job Cluster**      | Use one cluster across multiple tasks to reduce cost and startup time              |
| **Cluster Specs**           | Define instance type, workers, auto-scaling, Spark version, libraries, init scripts |

---

## 4. 🧾 Parameters (Optional)

You can pass **parameters to notebooks, scripts, or queries**.

| Method                     | Description                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| **Notebook Widgets**        | Use `dbutils.widgets.text()` to create input parameters                     |
| **Named Parameters**        | Key-value pairs in the UI or API                                            |
| **Script Arguments**        | Python script receives arguments via `sys.argv`                             |
| **SQL Parameters**          | Can use parameters in SQL tasks using the `{{param_name}}` syntax           |

---

## 5. ⏰ Schedule Configuration (Optional)

Define when your job should run automatically.

| Option                 | Description                                                        |
|------------------------|--------------------------------------------------------------------|
| **Manual**              | Run the job manually using "Run Now"                              |
| **Scheduled**           | Use cron expression or UI to trigger jobs on a schedule           |
| **Pause/Resume**        | You can pause and resume scheduled jobs at any time               |

**Cron Examples:**
- `0 0 * * * ?` → every hour
- `0 0 0 * * ?` → every midnight

---

## 6. 🔁 Retry Policy

Configure automatic retries for failed tasks.

| Option         | Description                                                             |
|----------------|-------------------------------------------------------------------------|
| **Max Retries** | Number of retry attempts before marking job as failed (e.g., 3 times)   |
| **Retry Interval** | (Optional via API) wait time between retries                         |

---

## 7. ⌛ Timeout Settings

Define how long a task/job can run before it's forcefully stopped.

- Timeout is set in **minutes**
- Prevents long-running or stuck tasks from consuming resources

---

## 8. 📣 Notifications

Setup alerts to receive job status updates.

| Trigger Type           | Delivery Options                                              |
|-------------------------|--------------------------------------------------------------|
| **On Success**          | Email/Webhook after successful completion                    |
| **On Failure**          | Alert if job or task fails                                   |
| **On Start / Completion** | Optional info on job execution lifecycle                  |
| **Webhooks**            | Integrate with external systems like Slack, PagerDuty, etc.  |

---

## 9. 👥 Permissions & Access Control

Manage who can view, run, or edit the job.

| Role                | Capabilities                                                             |
|---------------------|--------------------------------------------------------------------------|
| **Owner**            | Full control: run, edit, delete, assign permissions                      |
| **Can Manage**       | Can modify configurations and permissions                               |
| **Can Run**          | Can trigger job but not modify it                                       |
| **Can View**         | Read-only access to logs, DAG, status                                   |

---

## 10. 🧑‍💼 Run As Option

Choose the identity under which the job runs:

| Option         | Use Case                                                             |
|----------------|----------------------------------------------------------------------|
| **Run as Owner** | Job always runs with owner's permission                             |
| **Run as User**  | Uses permissions of the person triggering the job                   |

---

## 11. 🧬 Git Version Control (For Notebooks)

Databricks supports versioning notebooks via Git.

- Supported providers: GitHub, GitLab, Bitbucket, Azure DevOps
- Link notebook directly to a branch
- Enables CI/CD integration
- Prevents accidental changes in production code

---

## 🔐 Advanced Options (Optional)

- **Access Modes**: Choose between shared, single-user, or no-isolation for cluster mode
- **Init Scripts**: Automate environment setup (e.g., Python lib installs)
- **Libraries**: Attach specific JAR, Egg, or Python libraries to the cluster
- **Environment Variables**: Set up env variables needed for tasks

---

## ✅ Summary

| Setting               | Purpose                                                                 |
|------------------------|-------------------------------------------------------------------------|
| Job Name               | Identify job uniquely                                                   |
| Tasks                 | Define work units (notebook/script/sql)                                 |
| Cluster               | Compute resource settings                                                |
| Parameters            | Input data for tasks                                                     |
| Schedule              | Automate job trigger via cron                                            |
| Retry/Timeout         | Manage failures or long-running jobs                                     |
| Notifications         | Alerting for success/failure                                             |
| Permissions           | Control access and visibility                                            |
| Run As                | Execute job as owner or triggering user                                  |
| Git Integration       | Enable source control for notebooks/scripts                              |
