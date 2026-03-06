# NumPy — pandas — SQL Operations Reference

## Table of contents

- [Sample dataset](#sample-dataset)
- [Assumption for NumPy examples](#assumption-for-numpy-examples)
- [Load / create data](#load--create-data)
- [Select rows and columns](#select-rows-and-columns)
- [Filter rows by condition](#filter-rows-by-condition)
- [Sort data](#sort-data)
- [Aggregation / summary statistics](#aggregation--summary-statistics)
- [Group by](#group-by)
- [Merge / Join tables](#merge--join-tables)
- [Handling missing values](#handling-missing-values)
- [Creating derived / new columns](#creating-derived--new-columns)

---

## Sample dataset

### Employees
| emp_id | name  | dept_id | age | salary |
|---|---|---:|---:|---:|
| 101 | Asha  | 10 | 29 | 60000 |
| 102 | Ravi  | 20 | 35 | 75000 |
| 103 | Meera | 10 | 31 | 68000 |
| 104 | Kiran | 30 | 28 | 52000 |
| 105 | Zoya  | 20 | 41 | 82000 |

### Departments
| dept_id | dept_name |
|---:|---|
| 10 | Engineering |
| 20 | HR |
| 30 | Finance |

### Dependents
| dep_id | emp_id | dependent_name | relation |
|---:|---:|---|---|
| 1 | 101 | Anya  | Child |
| 2 | 102 | Neha  | Spouse |
| 3 | 102 | Rohan | Child |
| 4 | 105 | Sara  | Spouse |

---

## Assumption for NumPy examples

`employees_np` is a **2D object array** with columns in this order:

- `0 = emp_id`
- `1 = name`
- `2 = dept_id`
- `3 = age`
- `4 = salary`

`employees_pd` is a pandas DataFrame with column names matching the Employees table.

---

## Load / create data

### Mental note
- **NumPy:** create a 2D array, usually position-based.
- **pandas:** create a DataFrame with named columns.
- **SQL:** data is assumed to already exist in tables; loading usually means querying from an existing table.

### NumPy
```python
employees_np = np.array([
    [101, "Asha", 10, 29, 60000],
    [102, "Ravi", 20, 35, 75000],
    [103, "Meera", 10, 31, 68000],
    [104, "Kiran", 30, 28, 52000],
    [105, "Zoya", 20, 41, 82000]
], dtype=object)
```

### pandas
```python
employees_pd = pd.DataFrame({
    "emp_id": [101, 102, 103, 104, 105],
    "name": ["Asha", "Ravi", "Meera", "Kiran", "Zoya"],
    "dept_id": [10, 20, 10, 30, 20],
    "age": [29, 35, 31, 28, 41],
    "salary": [60000, 75000, 68000, 52000, 82000]
})
```

### SQL
```sql
SELECT *
FROM employees;
```

### Shape note
- **NumPy:** 2D array with shape `(5, 5)`
- **pandas:** DataFrame with `5 rows × 5 columns`
- **SQL:** result set with `5 rows × 5 columns` for this sample data

---

## Select rows and columns

### Mental note
- **NumPy:** selection is by **integer position** using row and column indexes.
- **pandas:** selection can be **label-based** or **position-based**.
- **SQL:** selection is naturally column-based; row slicing by position is not part of core generic SQL.

### Select columns: name and salary

#### NumPy
```python
employees_np[:, [1, 4]]
```

#### pandas
```python
employees_pd[["name", "salary"]]
```

#### SQL
```sql
SELECT name, salary
FROM employees;
```

### Shape note
- **NumPy:** shape `(5, 2)`
- **pandas:** `5 rows × 2 columns`
- **SQL:** result set with `5 rows × 2 columns`

### Select rows: index 1 to 3

#### NumPy
```python
employees_np[1:4]
```

#### pandas
```python
employees_pd.iloc[1:4]
```

#### SQL
```sql
-- No exact generic SQL equivalent by row index.
-- SQL tables are unordered unless an ORDER BY is applied.
```

### Feasibility note
- **Why no exact SQL equivalent?** SQL works with unordered sets of rows unless an explicit ordering is defined. “Rows 1 to 3” is a positional idea that fits arrays/DataFrames more naturally than generic SQL.

### Shape note
- **NumPy:** shape `(3, 5)`
- **pandas:** `3 rows × 5 columns`
- **SQL:** not applicable in generic form

### Select rows and columns together
Example: rows `1:4`, columns `name` and `salary`

#### NumPy
```python
employees_np[1:4, [1, 4]]
```

#### pandas
```python
employees_pd.iloc[1:4][["name", "salary"]]
```

#### SQL
```sql
-- No exact generic SQL equivalent for row positions 1:4.
-- Closest idea needs an explicit ordering plus row-number logic,
-- which goes beyond simple generic SQL reference style.
```

### Feasibility note
- **Why only a partial SQL mapping?** Selecting columns is easy in SQL, but selecting a positional row slice is not a clean generic equivalent unless we introduce extra constructs such as ordering and row numbering.

### Shape note
- **NumPy:** shape `(3, 2)`
- **pandas:** `3 rows × 2 columns`
- **SQL:** not applicable in clean generic form

---

## Filter rows by condition

### Mental note
- **NumPy:** filtering is usually done with a boolean mask.
- **pandas:** filtering also uses a boolean mask, but column access is more readable.
- **SQL:** filtering is done with `WHERE`.

### Filter employees with salary > 65000

#### NumPy
```python
employees_np[employees_np[:, 4] > 65000]
```

#### pandas
```python
employees_pd[employees_pd["salary"] > 65000]
```

#### SQL
```sql
SELECT *
FROM employees
WHERE salary > 65000;
```

### Shape note
- **NumPy:** shape `(3, 5)` for this sample data
- **pandas:** `3 rows × 5 columns` for this sample data
- **SQL:** result set with `3 rows × 5 columns` for this sample data

### Filter employees in dept_id == 10

#### NumPy
```python
employees_np[employees_np[:, 2] == 10]
```

#### pandas
```python
employees_pd[employees_pd["dept_id"] == 10]
```

#### SQL
```sql
SELECT *
FROM employees
WHERE dept_id = 10;
```

### Shape note
- **NumPy:** shape `(2, 5)` for this sample data
- **pandas:** `2 rows × 5 columns` for this sample data
- **SQL:** result set with `2 rows × 5 columns` for this sample data

### Filter with multiple conditions
Example: `dept_id == 20` and `age > 35`

#### NumPy
```python
employees_np[(employees_np[:, 2] == 20) & (employees_np[:, 3] > 35)]
```

#### pandas
```python
employees_pd[(employees_pd["dept_id"] == 20) & (employees_pd["age"] > 35)]
```

#### SQL
```sql
SELECT *
FROM employees
WHERE dept_id = 20 AND age > 35;
```

### Shape note
- **NumPy:** shape `(1, 5)` for this sample data
- **pandas:** `1 row × 5 columns` for this sample data
- **SQL:** result set with `1 row × 5 columns` for this sample data


---

## Sort data

### Mental note
- **NumPy:** sorting typically uses `np.argsort()` to reorder rows.
- **pandas:** sorting is straightforward with `sort_values()`.
- **SQL:** sorting is done with `ORDER BY`.

### Sort employees by salary (ascending)

#### NumPy
```python
employees_np[np.argsort(employees_np[:, 4])]
```

#### pandas
```python
employees_pd.sort_values("salary")
```

#### SQL
```sql
SELECT *
FROM employees
ORDER BY salary ASC;
```

### Shape note
- **NumPy:** shape `(5, 5)` (rows reordered)
- **pandas:** `5 rows × 5 columns`
- **SQL:** result set with `5 rows × 5 columns`

### Sort employees by age (descending)

#### NumPy
```python
employees_np[np.argsort(-employees_np[:, 3].astype(int))]
```

#### pandas
```python
employees_pd.sort_values("age", ascending=False)
```

#### SQL
```sql
SELECT *
FROM employees
ORDER BY age DESC;
```

---

## Aggregation / summary statistics

### Mental note
- **NumPy:** aggregation functions operate directly on arrays.
- **pandas:** aggregation operates on Series or DataFrame columns.
- **SQL:** aggregation uses functions like `AVG`, `SUM`, `COUNT`.

### Average salary

#### NumPy
```python
np.mean(employees_np[:, 4].astype(int))
```

#### pandas
```python
employees_pd["salary"].mean()
```

#### SQL
```sql
SELECT AVG(salary)
FROM employees;
```

### Total salary expense

#### NumPy
```python
np.sum(employees_np[:, 4].astype(int))
```

#### pandas
```python
employees_pd["salary"].sum()
```

#### SQL
```sql
SELECT SUM(salary)
FROM employees;
```

### Employee count

#### NumPy
```python
employees_np.shape[0]
```

#### pandas
```python
employees_pd.shape[0]
```

#### SQL
```sql
SELECT COUNT(*)
FROM employees;
```

---

## Group by

### Mental note
- **NumPy:** grouping is not native; typically requires masking or helper logic.
- **pandas:** `groupby()` provides powerful grouped operations.
- **SQL:** grouping uses `GROUP BY`.

### Average salary per department

#### NumPy
```python
np.mean(employees_np[employees_np[:, 2] == 10][:, 4].astype(int))
```

*(Example shown only for one department; NumPy requires manual filtering for each group.)*

#### pandas
```python
employees_pd.groupby("dept_id")["salary"].mean()
```

#### SQL
```sql
SELECT dept_id, AVG(salary)
FROM employees
GROUP BY dept_id;
```

### Employee count per department

#### NumPy
```python
np.sum(employees_np[:, 2] == 10)
```

*(Manual approach again required for each department.)*

#### pandas
```python
employees_pd.groupby("dept_id").size()
```

#### SQL
```sql
SELECT dept_id, COUNT(*)
FROM employees
GROUP BY dept_id;
```


---

## Merge / Join tables

### Mental note
- **NumPy:** no built-in relational join capability; joins require manual matching logic.
- **pandas:** `merge()` provides SQL-like joins directly on DataFrames.
- **SQL:** joins are a core relational operation using `JOIN` clauses.

### Join employees with departments (get department name for each employee)

#### NumPy
```python
# Conceptual example (manual lookup approach)
result = []
for emp in employees_np:
    dept = departments_np[departments_np[:,0] == emp[2]][0]
    result.append(list(emp) + [dept[1]])

np.array(result, dtype=object)
```

*(Requires manual matching on dept_id; no built‑in join operation.)*

#### pandas
```python
employees_pd.merge(departments_pd, on="dept_id")
```

#### SQL
```sql
SELECT e.*, d.dept_name
FROM employees e
JOIN departments d
ON e.dept_id = d.dept_id;
```

### Shape note
- **NumPy:** shape `(5, 6)` after appending department name
- **pandas:** `5 rows × 6 columns`
- **SQL:** result set with `5 rows × 6 columns`

---

### Join employees with dependents

*(Shows one‑to‑many relationship: one employee may have multiple dependents.)*

#### NumPy
```python
# Conceptual manual join
result = []
for emp in employees_np:
    matches = dependents_np[dependents_np[:,1] == emp[0]]
    for dep in matches:
        result.append(list(emp) + list(dep))

np.array(result, dtype=object)
```

*(Requires nested loops and manual matching logic.)*

#### pandas
```python
employees_pd.merge(dependents_pd, on="emp_id")
```

#### SQL
```sql
SELECT e.*, d.dependent_name, d.relation
FROM employees e
JOIN dependents d
ON e.emp_id = d.emp_id;
```

### Shape note
- **NumPy:** number of rows expands depending on dependents
- **pandas:** rows expand for one‑to‑many matches
- **SQL:** result set expands similarly

---

### Left join example

Get **all employees**, even if they have **no dependents**.

#### NumPy
```python
# Requires custom logic to insert rows when no match exists
# No native left join functionality
```

#### pandas
```python
employees_pd.merge(dependents_pd, on="emp_id", how="left")
```

#### SQL
```sql
SELECT e.*, d.dependent_name, d.relation
FROM employees e
LEFT JOIN dependents d
ON e.emp_id = d.emp_id;
```

### Mental takeaway
- **NumPy:** joins are cumbersome and not idiomatic.
- **pandas:** designed to handle relational-style data analysis.
- **SQL:** native environment for relational joins.

---

## Handling missing values

### Mental note
- **NumPy:** missing values are typically represented with `np.nan` (for numeric arrays).
- **pandas:** has rich utilities like `isna()`, `dropna()`, and `fillna()`.
- **SQL:** missing values are represented using `NULL`.

### Detect missing values in salary

#### NumPy
```python
np.isnan(employees_np[:, 4].astype(float))
```

#### pandas
```python
employees_pd["salary"].isna()
```

#### SQL
```sql
SELECT *
FROM employees
WHERE salary IS NULL;
```

### Replace missing salary with 0

#### NumPy
```python
col = employees_np[:, 4].astype(float)
col[np.isnan(col)] = 0
```

#### pandas
```python
employees_pd["salary"].fillna(0)
```

#### SQL
```sql
SELECT emp_id, name, COALESCE(salary, 0) AS salary
FROM employees;
```

### Shape note
Handling missing values does not change row/column count unless rows are dropped.

---

## Creating derived / new columns

### Mental note
- **NumPy:** new columns require constructing a new array or stacking columns.
- **pandas:** very easy via vectorized column assignment.
- **SQL:** new columns are typically created in the `SELECT` clause using expressions.

### Create a new column: salary in thousands

#### NumPy
```python
salary_k = employees_np[:, 4].astype(int) / 1000
np.column_stack((employees_np, salary_k))
```

#### pandas
```python
employees_pd["salary_k"] = employees_pd["salary"] / 1000
```

#### SQL
```sql
SELECT *, salary/1000 AS salary_k
FROM employees;
```

### Shape note
- **NumPy:** shape becomes `(5, 6)` after stacking new column
- **pandas:** DataFrame becomes `5 rows × 6 columns`
- **SQL:** result set includes the computed column but does not modify the base table

