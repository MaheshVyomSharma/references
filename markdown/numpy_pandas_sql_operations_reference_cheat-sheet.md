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
- [Wide ↔ Long format conversion](#wide--long-format-conversion)
- [Handling duplicates](#handling-duplicates)
- [Type conversion](#type-conversion)
- [String operations](#string-operations)
- [Appendix A — Best practices](#appendix-a--best-practices)
- [Appendix B — Concept mapping](#appendix-b--concept-mapping)
- [Appendix C — SQL caveats](#appendix-c--sql-caveats)

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

## Quick comparison cheat sheet

This section compresses the core operations into side‑by‑side comparisons for fast exam revision.

### Load / create data

| NumPy | pandas | SQL |
|---|---|---|
| `np.array([...], dtype=object)` | `pd.DataFrame({...})` | `SELECT * FROM employees;` |

---

### Select rows and columns

| Task | NumPy | pandas | SQL |
|---|---|---|---|
| Select columns | `employees_np[:, [1,4]]` | `employees_pd[["name","salary"]]` | `SELECT name, salary FROM employees;` |
| Select rows | `employees_np[1:4]` | `employees_pd.iloc[1:4]` | Not positional in generic SQL |
| Rows + columns | `employees_np[1:4,[1,4]]` | `employees_pd.iloc[1:4][["name","salary"]]` | Requires ordering + row numbering |

---

### Filter rows

| NumPy | pandas | SQL |
|---|---|---|
| `employees_np[employees_np[:,4] > 65000]` | `employees_pd[employees_pd["salary"] > 65000]` | `SELECT * FROM employees WHERE salary > 65000;` |

---

### Sort data

| NumPy | pandas | SQL |
|---|---|---|
| `employees_np[np.argsort(employees_np[:,4])]` | `employees_pd.sort_values("salary")` | `SELECT * FROM employees ORDER BY salary;` |

---

### Aggregation

| Task | NumPy | pandas | SQL |
|---|---|---|---|
| Average salary | `np.mean(employees_np[:,4].astype(int))` | `employees_pd["salary"].mean()` | `SELECT AVG(salary) FROM employees;` |
| Total salary | `np.sum(employees_np[:,4].astype(int))` | `employees_pd["salary"].sum()` | `SELECT SUM(salary) FROM employees;` |
| Row count | `employees_np.shape[0]` | `employees_pd.shape[0]` | `SELECT COUNT(*) FROM employees;` |

---

### Group by

| NumPy | pandas | SQL |
|---|---|---|
| Manual filtering per group | `employees_pd.groupby("dept_id")["salary"].mean()` | `SELECT dept_id, AVG(salary) FROM employees GROUP BY dept_id;` |

---

### Merge / join

| NumPy | pandas | SQL |
|---|---|---|
| Manual matching logic | `employees_pd.merge(departments_pd, on="dept_id")` | `SELECT e.*, d.dept_name FROM employees e JOIN departments d ON e.dept_id=d.dept_id;` |

---

### Handling missing values

| NumPy | pandas | SQL |
|---|---|---|
| `np.isnan(col)` | `employees_pd["salary"].isna()` | `WHERE salary IS NULL` |
| Replace missing | manual assignment | `fillna(0)` | `COALESCE(salary,0)` |

---

### Derived columns

| NumPy | pandas | SQL |
|---|---|---|
| `np.column_stack((employees_np, salary_k))` | `employees_pd["salary_k"] = employees_pd["salary"]/1000` | `SELECT *, salary/1000 AS salary_k FROM employees;` |

---

### Wide ↔ Long reshape

| Transformation | pandas | SQL | NumPy |
|---|---|---|---|
| Wide → Long | `pd.melt()` | `UNION ALL` or `UNPIVOT` | manual reconstruction |
| Long → Wide | `pivot()` | `CASE + GROUP BY` | manual reconstruction |

---

### Handling duplicates

| NumPy | pandas | SQL |
|---|---|---|
| `np.unique(arr, axis=0)` | `drop_duplicates()` | `SELECT DISTINCT *` |

---

### Type conversion

| NumPy | pandas | SQL |
|---|---|---|
| `astype(int)` | `astype(int)` | `CAST(column AS INT)` |

---

### String operations

| NumPy | pandas | SQL |
|---|---|---|
| `np.char.upper()` | `.str.upper()` | `UPPER(column)` |

---

## Appendix A — Best practices

### pandas row and column selection

Simplified example used earlier:

```python
employees_pd.iloc[1:4][["name","salary"]]
```

Preferred approach:

```python
employees_pd.loc[1:3, ["name","salary"]]
```

Reason:
- avoids chained indexing
- clearer semantics

---

### NumPy sorting with argsort

```python
employees_np[np.argsort(employees_np[:,4])]
```

Explanation:
- `argsort()` returns the index order of sorted values
- allows row-wise reordering

---

### SQL row ordering

SQL tables have **no guaranteed row order**.

Correct pattern:

```sql
SELECT *
FROM employees
ORDER BY salary;
```

---

## Appendix B — Concept mapping

| Concept | NumPy | pandas | SQL |
|---|---|---|---|
| Table | 2D array | DataFrame | Table |
| Column | array slice | Series | Column |
| Row | array row | row | row |
| Index / identity | position | index | primary key |

---

## Appendix C — SQL caveats

Some SQL features depend on the database vendor.

| Feature | ANSI SQL | Vendor specific |
|---|---|---|
| PIVOT | not standard | SQL Server / Oracle |
| LIMIT | MySQL/Postgres | not ANSI |
| TOP | SQL Server | not portable |

