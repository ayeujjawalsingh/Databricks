# 📘 Databricks: DBFS, DBUTILS, Magic Commands, and Widgets

## 📂 1. DBFS (Databricks File System)

### 🔹 What is DBFS?

**DBFS** stands for **Databricks File System**, which is an abstraction over cloud object storage (like AWS S3, Azure Blob, or GCP Storage). It allows you to access, read, write, and manage files inside Databricks using a POSIX-like file path structure.

### 🔹 Key Points:

- DBFS is **auto-mounted** to cloud storage.
- All users have access to `/dbfs/` path.
- Useful for storing **data files**, **model files**, **scripts**, **init scripts**, etc.

### 🔹 Common DBFS Paths:

| Path | Description |
|------|-------------|
| `/dbfs/tmp/` | Temporary storage location |
| `/dbfs/FileStore/` | Public-access files (accessible via URL) |
| `/dbfs/user/` | User-specific space |
| `/dbfs/mnt/` | External cloud storage mounted here |

### 🔹 DBFS Access Methods:

#### Using `%fs` magic command:

```bash
%fs ls /FileStore/
%fs cp /FileStore/data.csv /tmp/data.csv
%fs rm -r /tmp/
```

#### Using Python:

```python
dbutils.fs.ls("/FileStore/")
dbutils.fs.cp("dbfs:/FileStore/file.csv", "dbfs:/tmp/file.csv")
```

> 🧠 Always prefix with `dbfs:/` in Python. `%fs` doesn't require it.

---

## 🛠️ 2. DBUTILS (Databricks Utilities)

### 🔹 What is `dbutils`?

`dbutils` is a powerful utility tool provided by Databricks to interact with the file system, manage secrets, chain notebooks, and more from within notebooks.

### 🔹 Commonly Used `dbutils` Modules:

| Module             | Purpose                                                        |
| ------------------ | -------------------------------------------------------------- |
| `dbutils.fs`       | Interact with DBFS (read/write/list files)                     |
| `dbutils.widgets`  | Create input widgets for parameterized notebooks               |
| `dbutils.notebook` | Call or chain other notebooks                                  |
| `dbutils.secrets`  | Access secrets from secret scopes                              |
| `dbutils.library`  | Manage Python/Scala/R libraries (deprecated in newer runtimes) |
| `dbutils.jobs`     | Access job context info like run ID, parent ID (in jobs)       |

---

### 🔹 Examples of `dbutils`:

#### 📁 DBFS Operations:

```python
dbutils.fs.ls("/mnt/")
dbutils.fs.rm("/mnt/test.csv", recurse=True)
dbutils.fs.cp("dbfs:/source.csv", "dbfs:/tmp/copy.csv")
```

#### 🔑 Secret Management:

```python
dbutils.secrets.get(scope="my-scope", key="my-key")
```

#### 🔄 Notebook Chaining:

```python
result = dbutils.notebook.run("ChildNotebook", timeout_seconds=60, arguments={"input": "value"})
```

---

## ✨ 3. Magic Commands in Databricks

### 🔹 What are Magic Commands?

Magic commands are special instructions prefixed with `%` or `%%` that allow switching between languages or performing system-level operations inside notebooks.

---

### 🔹 Common Magic Commands:

| Magic Command | Usage                                                |
| ------------- | ---------------------------------------------------- |
| `%python`     | Executes Python code                                 |
| `%sql`        | Executes SQL statements                              |
| `%scala`      | Executes Scala code                                  |
| `%r`          | Executes R code                                      |
| `%sh`         | Executes shell commands                              |
| `%fs`         | Runs DBFS file system commands (like `ls`, `cp`)     |
| `%run`        | Executes another notebook inline                     |
| `%pip`        | Installs Python packages temporarily in the notebook |

---

### 🔹 Examples:

```python
%sql
SELECT * FROM sales_table

%sh
ls -l /dbfs/FileStore/

%fs
ls /mnt/datalake/

%pip install pandas
```

---

> ⚠️ `%pip` and `%conda` are available only in Databricks Runtime 7.1+ for installing packages inside notebook sessions.

---

## 🧩 4. Widgets (Parameterized Inputs)

### 🔹 What are Widgets?

**Widgets** allow users to **input parameters** into notebooks. These are especially useful for **interactive dashboards** or **parameterized jobs**.

They can take various forms like text boxes, dropdowns, comboboxes, and multiselects.

---

### 🔹 Creating Widgets

| Type        | Code Example                                                                 |
| ----------- | ---------------------------------------------------------------------------- |
| Text        | `dbutils.widgets.text("input1", "default", "Input Label")`                   |
| Dropdown    | `dbutils.widgets.dropdown("choice", "A", ["A", "B", "C"], "Select Option")`  |
| Combobox    | `dbutils.widgets.combobox("combo", "A", ["A", "B", "C"], "Combo Option")`    |
| Multiselect | `dbutils.widgets.multiselect("multi", "A", ["A", "B", "C"], "Multi Select")` |

---

### 🔹 Accessing Widget Values

```python
value = dbutils.widgets.get("input1")
print("Input received:", value)
```

---

### 🔹 Removing Widgets

```python
dbutils.widgets.remove("input1")         # Remove a specific widget
dbutils.widgets.removeAll()              # Remove all widgets
```

---

## ✅ Summary

| Feature            | Description                                                 | Use Cases                                       |
| ------------------ | ----------------------------------------------------------- | ----------------------------------------------- |
| **DBFS**           | Cloud-backed file system in Databricks                      | Store CSVs, logs, models, configs               |
| **DBUTILS**        | Utilities for file, secret, notebook, and widget operations | Read files, chain notebooks, use secrets        |
| **Magic Commands** | Language and system commands inside notebooks               | Switch context, install packages, run notebooks |
| **Widgets**        | Create dynamic input forms in notebooks                     | Create parameterized ETL jobs, dashboards       |
