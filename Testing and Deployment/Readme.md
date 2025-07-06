# 📘 Testing and Deployment in Databricks

This guide covers how to **test, promote, and deploy** Databricks notebooks and jobs using **Repos**, **Git Integration**, and **CI/CD Pipelines**. 

---

## 🧪 1. Testing Databricks Notebooks

### ✅ Why Testing Is Important

Testing ensures:
- Data logic works correctly
- Pipelines are reliable
- Bugs are caught early
- Promotes clean modular development

---

### 🔍 Types of Testing

| Type             | Purpose                                                                 |
|------------------|-------------------------------------------------------------------------|
| **Unit Testing** | Test small components or logic (like functions or UDFs)                 |
| **Integration**  | Test interaction between multiple components (e.g., notebooks, tables)  |
| **E2E Testing**  | Simulate a full pipeline run from source to final destination           |
| **Data Quality** | Ensure schema and values meet business expectations                     |

---

### 🧰 How to Perform Testing

#### ✅ 1. Use Python Functions and `pytest` for Logic

Write modular Python functions in your notebooks:

```python
# transform.py
def normalize_name(name):
    return name.strip().lower()
```

Write unit tests using `pytest`:

```python
def test_normalize_name():
    assert normalize_name("  Alice ") == "alice"
```

Run tests locally or in notebook:

```bash
pytest test_transform.py
```

#### ✅ 2. Test Notebook Workflows with `dbutils.notebook.run()`

Create a test orchestration notebook:

```python
result = dbutils.notebook.run("/Repos/user/notebooks/transform", 60, {"env": "test"})
assert "success" in result
```

#### ✅ 3. Use SQL Temporary Views for Unit Testing SQL

```sql
CREATE OR REPLACE TEMP VIEW test_customers AS
SELECT 1 AS id, 'Alice' AS name;

SELECT * FROM test_customers WHERE name = 'Alice';
```

#### ✅ 4. Use Delta Live Table Expectations

Ensure data quality during ingestion/transformation:

```sql
CREATE OR REFRESH LIVE TABLE customers_cleaned AS
SELECT * FROM LIVE.customers_raw
EXPECT email IS NOT NULL ON VIOLATION DROP ROW
```

---

## 🔄 2. Promoting Code Using Repos (Git Integration)

### ✅ What Is Databricks Repos?

`Repos` allow you to:

* Sync notebooks with GitHub, GitLab, Bitbucket, or Azure DevOps
* Work with branches, commits, PRs
* Promote code between dev/stage/prod environments

---

### 🔗 Git Integration Setup

1. Go to **Repos tab** in Databricks UI
2. Click **Add Repo**
3. Paste Git URL (HTTPS)
4. Authenticate using **Personal Access Token (PAT)**
5. Repo is cloned into your workspace

---

### 🛠️ Typical Git Workflow

| Step           | Action                                             |
| -------------- | -------------------------------------------------- |
| Development    | Work in `dev` branch                               |
| Pull Updates   | Sync latest code using Pull                        |
| Commit Changes | Use Git sidebar in Databricks or CLI               |
| Push to Remote | Push changes to Git provider (GitHub/GitLab, etc.) |
| Open PR        | Merge `dev → stage` or `stage → prod` after review |

---

### 📂 Environment Folder Structure (Branch Mapping)

| Environment | Branch    | Example Folder                      |
| ----------- | --------- | ----------------------------------- |
| Dev         | `dev`     | `/Repos/ujjawal/dev-notebooks/`     |
| QA          | `staging` | `/Repos/ujjawal/staging-notebooks/` |
| Production  | `main`    | `/Repos/ujjawal/prod-notebooks/`    |

---

### 📌 Best Practices

* Use **feature branches** per developer
* Keep code modular in scripts
* Add **README.md** in each repo
* Use **notebook versioning** via Git
* Enable **branch protection** for production

---

## ⚙️ 3. CI/CD Pipeline for Databricks

### 🚀 Why Use CI/CD?

CI/CD allows:

* Automated testing on every commit
* Seamless deployment to staging/prod
* Integration with GitHub, Azure DevOps, GitLab

---

### 🧰 Tools Used

| Tool                | Purpose                            |
| ------------------- | ---------------------------------- |
| GitHub Actions      | CI/CD pipeline orchestration       |
| Databricks CLI      | Deploy notebooks/jobs via terminal |
| Databricks REST API | Deploy & manage via scripts        |
| `pytest`            | Run unit tests                     |

---

### ⚙️ Setup Databricks CLI

#### ✅ Install and Configure

```bash
pip install databricks-cli

databricks configure --token
```

Add:

* DATABRICKS\_HOST
* DATABRICKS\_TOKEN

#### ✅ Upload Notebooks to Workspace

```bash
databricks workspace import_dir ./notebooks /Repos/ujjawal/dev-notebooks -o
```

#### ✅ Create Job via JSON

`job.json` example:

```json
{
  "name": "etl-job",
  "new_cluster": {
    "spark_version": "13.3.x-scala2.12",
    "node_type_id": "i3.xlarge",
    "num_workers": 2
  },
  "notebook_task": {
    "notebook_path": "/Repos/ujjawal/dev-notebooks/transform"
  },
  "timeout_seconds": 3600
}
```

```bash
databricks jobs create --json-file job.json
```

---

### 🪄 GitHub Actions: CI/CD Pipeline Example

Create a `.github/workflows/deploy.yml`

```yaml
name: Deploy Databricks Jobs

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'

      - name: Install CLI
        run: pip install databricks-cli

      - name: Configure Databricks
        run: databricks configure --token
        env:
          DATABRICKS_HOST: ${{ secrets.DATABRICKS_HOST }}
          DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}

      - name: Upload Notebooks
        run: |
          databricks workspace import_dir ./notebooks /Repos/ujjawal/prod-notebooks -o

      - name: Deploy Job
        run: |
          databricks jobs create --json-file ./job.json
```

---

## ✅ Best Practices Summary

| Area           | Recommendation                                                    |
| -------------- | ----------------------------------------------------------------- |
| **Testing**    | Use `pytest`, `dbutils.notebook.run`, and Delta Live expectations |
| **Repos**      | Use GitFlow (dev → staging → prod) and feature branches           |
| **Deployment** | Automate with Databricks CLI and GitHub Actions                   |
| **Security**   | Use secrets for PATs and tokens                                   |
| **Monitoring** | Log test outputs, notebook results, and job runs                  |

---

## 📁 Suggested Project Structure

```
my_project/
│
├── notebooks/
│   ├── ingestion.py
│   ├── transform.py
│   ├── test_transform.py
│
├── configs/
│   └── job.json
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
└── README.md
```

---

## 📌 Final Tips

* Maintain modular, testable notebook code.
* Treat notebooks as code — version control is a must.
* Automate everything from tests to deployments.
* Don’t commit secrets or tokens.
* Use CI/CD to prevent manual errors in production.
