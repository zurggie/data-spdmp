# Joins

## Standard Joins

```sql
-- INNER JOIN
SELECT e.name, d.name AS dept
FROM employees e INNER JOIN departments d ON e.dept_id = d.id;

-- LEFT / RIGHT / FULL OUTER
SELECT e.name, d.name
FROM employees e LEFT JOIN departments d ON e.dept_id = d.id;

-- CROSS JOIN
SELECT e.name, d.name
FROM employees e CROSS JOIN departments d;

-- NATURAL JOIN (auto-match same-named columns)
SELECT * FROM employees NATURAL JOIN departments;
```

### USING Clause

Shorthand when join columns share the same name:

```sql
SELECT e.name, d.name AS dept
FROM employees e JOIN departments d USING (dept_id);
```

### Column Name Shorthand

Specify join columns inline:

```sql
FROM employees JOIN departments USING (dept_id);
```

---

## ASOF Join

Match on nearest timestamp — useful for event timelines, sensor data, log joins:

```sql
SELECT e.name, s.sale_amount
FROM employees e ASOF JOIN sales s
  ON e.emp_id = s.emp_id AND e.hire_date <= s.sale_date;
```

Each row on the left matches the closest (strictly less-or-equal) key on the right.

---

## POSITIONAL Join

Merge tables row-by-row by position, no key:

```sql
SELECT *
FROM employees POSITIONAL JOIN bonuses;
```

## LATERAL Join

Reference columns from preceding FROM items inside subqueries:

```sql
SELECT e.name, top_sale.*
FROM employees e,
LATERAL (
    SELECT amount FROM sales
    WHERE sales.emp_id = e.id
    ORDER BY amount DESC LIMIT 5
) AS top_sale;
```

---

## VALUES in JOIN

```sql
SELECT e.*
FROM employees e
JOIN (VALUES ('Engineering'), ('Sales')) AS d(dept)
  ON e.department = d.dept;
```

## Subquery in FROM

```sql
SELECT dept, avg_salary
FROM (
    SELECT department AS dept, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) WHERE avg_salary > 50000;
```
