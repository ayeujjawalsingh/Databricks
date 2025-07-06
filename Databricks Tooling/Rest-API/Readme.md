# 🌐 Databricks Tooling: REST API Guide (Jobs, Clusters, Workflows)

This guide provides a complete overview of using **Databricks REST API** for **job orchestration, scheduling, cluster management**, and other tooling capabilities.

The REST API allows full programmatic access to the Databricks environment — useful for **CI/CD**, **automation**, and **custom integration** with other platforms.

---

## 📘 Overview of REST API

- Databricks REST APIs are available for:
  - **Jobs**
  - **Clusters**
  - **Libraries**
  - **Secrets**
  - **DBFS**
  - **Token Auth**
  - **SQL Warehouses**
  - **Unity Catalog**
- Uses **HTTPS-based endpoints**
- Can be used via:
  - `curl` commands
  - Python scripts
  - Postman
  - Custom tooling
  - CI/CD pipelines

---

## 🔐 Authentication

Databricks REST API uses **PAT (Personal Access Token)** for authentication.

### 🪪 Generate Token:
1. Go to Databricks → User Settings → **Access Tokens**
2. Generate a token and copy it

### 🔐 Set Header for API Requests:
```http
Authorization: Bearer <YOUR_TOKEN>
Content-Type: application/json
```

---

## 📡 Base URL Format

```http
https://<databricks-instance>/api/2.0/
```

Example:

```
https://adb-1234567890123456.7.azuredatabricks.net/api/2.0/jobs/runs/list
```

---

## ⚙️ Common Job APIs

### 1. 🔍 List All Jobs

```http
GET /api/2.1/jobs/list
```

Returns metadata about all jobs created in your workspace.

---

### 2. 📦 Create a New Job

```http
POST /api/2.1/jobs/create
```

#### Sample Payload:

```json
{
  "name": "daily-etl-pipeline",
  "tasks": [
    {
      "task_key": "ingest",
      "notebook_task": {
        "notebook_path": "/Shared/ingest_raw_data",
        "base_parameters": {
          "input_path": "s3://raw-data/"
        }
      },
      "job_cluster_key": "shared_cluster"
    },
    {
      "task_key": "transform",
      "depends_on": [
        { "task_key": "ingest" }
      ],
      "notebook_task": {
        "notebook_path": "/Shared/transform_data"
      },
      "job_cluster_key": "shared_cluster"
    }
  ],
  "job_clusters": [
    {
      "job_cluster_key": "shared_cluster",
      "new_cluster": {
        "spark_version": "12.2.x-scala2.12",
        "node_type_id": "Standard_DS3_v2",
        "num_workers": 2
      }
    }
  ]
}
```

---

### 3. 🚀 Run a Job Immediately

```http
POST /api/2.1/jobs/run-now
```

#### Payload:

```json
{
  "job_id": 123456,
  "notebook_params": {
    "input_date": "2024-01-01"
  }
}
```

---

### 4. 📜 List Past Job Runs

```http
GET /api/2.1/jobs/runs/list
```

Query Parameters:

* `job_id` (optional)
* `limit`, `offset`

---

### 5. 👀 Get Run Status (Job Monitoring)

```http
GET /api/2.1/jobs/runs/get?run_id=1234
```

Returns status like:

* `PENDING`
* `RUNNING`
* `SUCCESS`
* `FAILED`
* `SKIPPED`

---

### 6. 🛑 Cancel or Stop a Job Run

```http
POST /api/2.1/jobs/runs/cancel
```

Payload:

```json
{
  "run_id": 1234
}
```

---

### 7. 🗑️ Delete a Job

```http
POST /api/2.1/jobs/delete
```

Payload:

```json
{
  "job_id": 123456
}
```

---

## ⚙️ Cluster API (Useful for Custom Cluster Control)

### Create a Cluster

```http
POST /api/2.0/clusters/create
```

### Terminate a Cluster

```http
POST /api/2.0/clusters/delete
```

### List Clusters

```http
GET /api/2.0/clusters/list
```

---

## 💡 Useful Headers and Tools

| Tool      | Usage                                |
| --------- | ------------------------------------ |
| `curl`    | Command-line API calls               |
| Postman   | API testing and automation           |
| Python    | Use `requests` library for scripting |
| Terraform | Provision jobs and clusters as IaC   |

---

## 📁 DBFS API Example

### Upload File to DBFS

```http
POST /api/2.0/dbfs/put
```

Payload:

```json
{
  "path": "/tmp/sample.json",
  "overwrite": true,
  "contents": "<base64-encoded-content>"
}
```

---

## 📘 Tips and Best Practices

| Best Practice                                  | Reason                                            |
| ---------------------------------------------- | ------------------------------------------------- |
| Use **retry logic** in custom automation       | Handle transient failures gracefully              |
| Store **job IDs/configs in source control**    | Helps maintain and replicate pipelines            |
| Use **base parameters** with widgets           | Makes jobs reusable for different inputs or dates |
| Keep **tokens secure** (e.g., secrets manager) | Never hardcode in scripts or CI/CD pipelines      |
| Use **job versioning** with Git integration    | Maintain reproducibility and rollback             |

---

## ✅ Summary

| Component        | Purpose                                                  |
| ---------------- | -------------------------------------------------------- |
| Jobs API         | Create, run, monitor, and delete jobs                    |
| Cluster API      | Start/stop clusters manually or for on-demand processing |
| DBFS API         | Upload/download data and artifacts to/from DBFS          |
| Auth             | Use bearer tokens for secure calls                       |
| Tool Integration | Use with CI/CD (GitHub Actions, Jenkins, Azure DevOps)   |
