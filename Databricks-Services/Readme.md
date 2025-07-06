# 📘 Databricks Essentials for Data Engineers

This document provides detailed notes of **Databricks Workspace**, **Clusters**, **Notebooks**, and **Library Installation**.

---

## 🧱 1. Databricks Workspace

### 🔹 What is a Databricks Workspace?

A **Databricks workspace** is a centralized environment in the cloud for collaborative development and execution of data engineering, data science, and analytics workflows. It integrates tools like notebooks, jobs, clusters, libraries, and version control under one unified UI.

### 🔹 Key Components in a Workspace

| Component          | Description |
|--------------------|-------------|
| **Workspace Browser** | File management system for organizing notebooks, folders, libraries, etc. |
| **Notebooks**         | Interactive editor for writing and executing Python, SQL, Scala, or R code. |
| **Repos**             | Git-integrated section for version controlling code and collaboration. |
| **Clusters**          | Compute engines for executing notebooks and jobs. |
| **Jobs**              | Scheduled executions for automation pipelines. |
| **Data**              | UI for managing and querying data tables and files. |
| **Compute**           | Interface to create and manage clusters and SQL warehouses. |

> 🧠 Everything inside the workspace is fully managed and hosted in the cloud (AWS, Azure, or GCP).

---

## ⚙️ 2. Databricks Clusters

### 🔹 What is a Cluster?

A **cluster** in Databricks is a group of virtual machines used to run your data processing and analytics code. It provides the computing power required to process datasets, train ML models, and run ETL workflows.

---

### 🔹 Types of Clusters

| Cluster Type        | Use Case |
|---------------------|----------|
| **All-purpose Cluster** | Used for interactive analysis via notebooks. Ideal for development and debugging. |
| **Job Cluster**         | Automatically spun up when a scheduled job runs, then terminated. Optimized for production workloads and cost. |
| **SQL Warehouse**       | Specialized for running SQL workloads and dashboards. Formerly known as SQL Endpoints. |

---

## 🔧 3. Cluster Configuration & Creation

### 🔹 Steps to Create a Cluster

1. Navigate to **Compute** from the sidebar.
2. Click on **Create Cluster**.
3. Fill in the following details:

| Field | Description |
|-------|-------------|
| **Cluster Name** | Human-readable name like `etl-dev-cluster` |
| **Cluster Mode** | Choose between Standard or High Concurrency |
| **Databricks Runtime** | Select appropriate runtime (e.g. Runtime with ML, GPU support) |
| **Node Type** | VM type (e.g., `i3.xlarge`, `r5d.2xlarge`) |
| **Autoscaling** | Enable to dynamically scale worker nodes based on load |
| **Worker Nodes** | Set minimum and maximum node count |
| **Driver Node** | Acts as the master node managing jobs and distributing work |
| **Auto Termination** | Set an idle timeout (e.g., 30 min) to reduce cost when inactive |
| **Libraries** | Optionally install packages (e.g., pandas, numpy) during creation |

> ✅ **Recommendation**: Use autoscaling + auto termination in dev/test environments to manage cost efficiently.

---

## 📓 4. Creating Notebooks in Databricks

### 🔹 How to Create a Notebook

1. Go to **Workspace** section.
2. Right-click on a folder or select `Create → Notebook`.
3. Provide the following:
   - **Notebook Name**
   - **Default Language** (Python, SQL, Scala, R)
   - **Select a Cluster** to attach and execute code

> 📌 Notebooks are live and interactive, making them ideal for exploration, development, and monitoring.

### 🔹 Magic Commands in Notebooks

| Magic Command | Purpose |
|---------------|---------|
| `%python`     | Run Python code |
| `%sql`        | Run SQL queries |
| `%scala`      | Run Scala code |
| `%r`          | Run R code |
| `%sh`         | Execute shell commands (e.g., `ls`, `pip install`) |
| `%run`        | Import and run another notebook |

---

## 📦 5. Installing Python Libraries in Databricks

Databricks supports multiple options to install and manage libraries depending on your scope (temporary vs. persistent).

---

### 🔹 Option 1: Notebook-level Installation (Temporary)

You can install packages only for the current notebook session using:

```python
%pip install pandas
%pip install seaborn==0.11.2
```

> 🧹 These libraries are **not persistent**. They are removed once the cluster restarts.

---

### 🔹 Option 2: Cluster-wide Installation (Persistent via UI)

1. Go to **Compute** → Choose your Cluster → Click on **Libraries** tab.
2. Click **Install New**.
3. Choose from the following options:

| Source     | Description                                        |
| ---------- | -------------------------------------------------- |
| **PyPI**   | Install Python packages (e.g., pandas, matplotlib) |
| **Maven**  | Java/Scala libraries                               |
| **Upload** | Upload `.whl`, `.egg`, or `.jar` files manually    |

> 📌 Cluster-level installations persist until the cluster is deleted.

---

### 🔹 Option 3: Workspace-level Installation using Init Script

For production environments or CI/CD pipelines, you can configure **init scripts** that automatically install required libraries every time the cluster starts.

Sample `init.sh`:

```bash
#!/bin/bash
/databricks/python/bin/pip install pandas scikit-learn
```

Upload this script to DBFS and attach it in the cluster's advanced configuration.

---

## ✅ Summary: Key Takeaways for Data Engineers

| Topic                | Best Practices                                                                                |
| -------------------- | --------------------------------------------------------------------------------------------- |
| Workspace            | Organize notebooks logically in folders, use Git Repos for version control                    |
| Cluster Type         | Use **All-purpose** for exploration, **Job** for scheduled runs, and **SQL Warehouse** for BI |
| Cluster Config       | Use **autoscaling** and **auto-termination** to save costs                                    |
| Notebook             | Attach to the correct cluster; use `%run` and magic commands to modularize                    |
| Library Installation | Use `%pip` for dev, UI for persistent installs, or init scripts for production setups         |
