# CTE & Set Operations

## WITH (Common Table Expressions)

```sql
WITH dept_stats AS (
    SELECT department, AVG(salary) AS avg_sal, COUNT() AS headcount
    FROM employees
    GROUP BY department
)
SELECT * FROM dept_stats WHERE headcount > 10 ORDER BY avg_sal DESC;
```

### Column Names in CTE

```sql
WITH top_earners(name, monthly) AS (
    SELECT name, salary / 12 FROM employees ORDER BY salary DESC LIMIT 5
)
SELECT * FROM top_earners;
```

### VALUES in CTE

```sql
WITH dept_list(dept) AS (
    VALUES ('Engineering'), ('Sales'), ('Marketing')
)
SELECT e.* FROM employees e JOIN dept_list d ON e.department = d.dept;
```

---

## Recursive CTE

```sql
WITH RECURSIVE ancestors AS (
    -- Anchor: top-level managers
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive: reports to reports
    SELECT e.id, e.name, e.manager_id, a.level + 1
    FROM employees e
    JOIN ancestors a ON e.manager_id = a.id
)
SELECT * FROM ancestors ORDER BY level, id;
```

---

## Set Operations

### UNION / UNION ALL

```sql
SELECT name, salary FROM current_employees
UNION ALL
SELECT name, salary FROM former_employees;

-- UNION (deduplicates)
SELECT department FROM engineering_team
UNION
SELECT department FROM sales_team;
```

### UNION ALL BY NAME

Align columns by name instead of position — handles different column orders:

```sql
SELECT name AS employee, salary FROM current_employees
UNION ALL BY NAME
SELECT salary, name AS employee FROM former_employees;
```

Columns are matched by name, not position.

### INTERSECT

Rows present in both results:

```sql
SELECT department FROM engineering_team
INTERSECT
SELECT department FROM sales_team;
```

### EXCEPT

Rows in first but not second:

```sql
SELECT department FROM all_departments
EXCEPT
SELECT department FROM inactive_departments;
```
