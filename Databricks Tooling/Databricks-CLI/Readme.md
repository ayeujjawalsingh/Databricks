# 🧰 Databricks CLI - Complete Guide

This document explains how to install, configure, and use the **Databricks CLI (Command Line Interface)** to manage your Databricks workspace programmatically using terminal commands.

---

## 📘 What is Databricks CLI?

The **Databricks CLI** is a command-line tool that lets you **interact with Databricks** from your terminal. It is useful for:

- Automating tasks (e.g., jobs, clusters)
- Managing files (DBFS)
- Managing secrets and tokens
- Working with notebooks
- Deploying CI/CD pipelines
- Scripting repeatable workflows

---

## 🧱 Prerequisites

- Python 3.8 or later
- Pip (Python package manager)
- A Databricks account and workspace
- A **Personal Access Token** for authentication

---

## 🚀 Installation

Install the CLI using pip:

```bash
pip install databricks-cli
```

Verify installation:

```bash
databricks --version
```

---

## 🔐 Authentication (Config Setup)

You need to configure the CLI with your **Databricks workspace URL** and **Personal Access Token (PAT)**.

### Generate a Token:

1. Go to Databricks workspace
2. Click on **User Settings → Access Tokens**
3. Generate a token and copy it

### Configure CLI (Quick Setup):

```bash
databricks configure --token
```

It will prompt:

```
Databricks Host: https://<your-instance>.cloud.databricks.com
Token: <paste-your-token>
```

Configuration is saved in `~/.databrickscfg`.

---

## 🧩 Common CLI Commands

Below are the most used CLI command groups:

---

### 📂 1. DBFS (Databricks File System)

| Command                             | Description           |
| ----------------------------------- | --------------------- |
| `databricks fs ls dbfs:/`           | List files in DBFS    |
| `databricks fs cp local.txt dbfs:/` | Upload a file to DBFS |
| `databricks fs rm dbfs:/file.txt`   | Delete file from DBFS |

---

### 📓 2. Workspace (Manage Notebooks, Folders)

| Command                                               | Description                               |
| ----------------------------------------------------- | ----------------------------------------- |
| `databricks workspace ls /Users`                      | List contents of a workspace path         |
| `databricks workspace import notebook.py /Shared/etl` | Import local notebook/script to workspace |
| `databricks workspace export /Shared/etl notebook.py` | Export workspace notebook to local        |
| `databricks workspace mkdirs /Shared/new-folder`      | Create folders in workspace               |

---

### ⚙️ 3. Jobs (Manage Jobs via CLI)

| Command                                 | Description                   |
| --------------------------------------- | ----------------------------- |
| `databricks jobs list`                  | List all jobs                 |
| `databricks jobs get --job-id <id>`     | Get job details               |
| `databricks jobs run-now --job-id <id>` | Trigger a job run immediately |
| `databricks jobs delete --job-id <id>`  | Delete a job                  |

You can also use a JSON file to create jobs:

```bash
databricks jobs create --json-file job.json
```

---

### 🖥️ 4. Clusters

| Command                                      | Description         |
| -------------------------------------------- | ------------------- |
| `databricks clusters list`                   | List clusters       |
| `databricks clusters start --cluster-id ID`  | Start a cluster     |
| `databricks clusters delete --cluster-id ID` | Terminate a cluster |

---

### 🔐 5. Secrets

Manage secrets for securely storing credentials or tokens.

| Command                                                   | Description                |
| --------------------------------------------------------- | -------------------------- |
| `databricks secrets create-scope --scope my-scope`        | Create a secret scope      |
| `databricks secrets put --scope my-scope --key my-key`    | Store a new secret         |
| `databricks secrets list --scope my-scope`                | List secrets under a scope |
| `databricks secrets delete --scope my-scope --key my-key` | Delete a specific secret   |

---

### 🧪 6. Libraries

| Command                                                          | Description                |
| ---------------------------------------------------------------- | -------------------------- |
| `databricks libraries install --cluster-id <id> --whl mylib.whl` | Install library on cluster |
| `databricks libraries list --cluster-id <id>`                    | List installed libraries   |

---

### 🧮 7. SQL Warehouses

| Command                                   | Description                         |
| ----------------------------------------- | ----------------------------------- |
| `databricks sql warehouses list`          | List all SQL warehouses             |
| `databricks sql warehouses get --id <id>` | Get details of a specific warehouse |

---

### 🧾 8. Tokens (Manage PATs via CLI)

| Command                    | Description               |
| -------------------------- | ------------------------- |
| `databricks tokens create` | Generate new access token |
| `databricks tokens list`   | List active tokens        |
| `databricks tokens delete` | Revoke a token            |

---

## 🛠️ Example: Automating Job Deployment via CLI

### Step 1: Create JSON definition (`job.json`)

```json
{
  "name": "my-etl-job",
  "tasks": [
    {
      "task_key": "step1",
      "notebook_task": {
        "notebook_path": "/Shared/etl"
      },
      "new_cluster": {
        "spark_version": "12.2.x-scala2.12",
        "node_type_id": "Standard_DS3_v2",
        "num_workers": 2
      }
    }
  ]
}
```

### Step 2: Deploy with CLI

```bash
databricks jobs create --json-file job.json
```

---

## 📦 CI/CD Integration

You can use the CLI inside:

* **GitHub Actions**
* **GitLab CI**
* **Azure DevOps Pipelines**
* **Jenkins**

By storing credentials as secrets and running CLI commands in steps.

---

## ✅ Best Practices

| Tip                                  | Why it Matters                                    |
| ------------------------------------ | ------------------------------------------------- |
| Use `--profile` for multiple configs | Work across different environments (dev, prod)    |
| Store configs in `.databrickscfg`    | Avoid hardcoding tokens in scripts                |
| Use secrets API for credentials      | Securely handle DB passwords, keys, etc.          |
| Automate with CLI + JSON             | Reproducible, trackable deployments               |
| Use Git versioning                   | Keep job scripts and configs under source control |
