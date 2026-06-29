# DuckDB Friendly SQL Patterns

Quick reference for DuckDB's concise SQL syntax — shorter, cleaner queries using DuckDB-specific features.

---

## Aggregation

### GROUP BY ALL

```sql
SELECT department, gender, COUNT()
FROM employees
GROUP BY ALL
```

### count() Shorthand

```sql
SELECT department, count() FROM employees GROUP BY department;
```

### Top-N in Group

```sql
SELECT department, max(salary, 5) AS top_5_salaries
FROM employees
GROUP BY department;
```

### FILTER Clause

```sql
SELECT
    department,
    count() AS total,
    count() FILTER (WHERE salary > 100000) AS high_earners,
    avg(salary) FILTER (WHERE age < 30) AS avg_young_salary
FROM employees
GROUP BY ALL;
```

---

## Column Selection

### SELECT * EXCLUDE

```sql
SELECT * EXCLUDE (id, ssn, salary)
FROM employees LIMIT 10;
```

### SELECT * REPLACE

```sql
SELECT * REPLACE (
    UPPER(name) AS name,
    salary * 1.1 AS salary
)
FROM employees LIMIT 10;
```

### COLUMNS() with Regex

```sql
SELECT COLUMNS('salary|bonus')
FROM employees LIMIT 10;
```

---

## Quick Exploration

### FROM-First Syntax

```sql
FROM employees LIMIT 10;
```

### SUMMARIZE

```sql
SUMMARIZE employees;
SUMMARIZE (SELECT * FROM employees WHERE department = 'Engineering');
```

---

## Column Aliases

### Prefix Aliases

```sql
SELECT
    total: COUNT(*),
    dept: department
FROM employees
GROUP BY department;
```

### Lateral Column Aliases

```sql
SELECT
    salary + bonus AS total_comp,
    total_comp * 0.05 AS tax_estimate
FROM employees LIMIT 5;
```

---

## Conditional Logic

### SWITCH Expression

```sql
SELECT
    name,
    SWITCH department
        WHEN 'Engineering' THEN 'Tech'
        WHEN 'Sales' THEN 'Revenue'
        WHEN 'Marketing' THEN 'Growth'
        ELSE 'Other'
    END AS category
FROM employees LIMIT 10;
```

---

## Pivoting

### PIVOT

```sql
PIVOT employees
ON department
USING COUNT(*)
GROUP BY gender;
```

### UNPIVOT

```sql
UNPIVOT monthly_sales
ON jan, feb, mar
INTO NAME month VALUE revenue;
```

---

## Trailing Commas

```sql
SELECT
    name,
    department,
    salary,
FROM employees
WHERE department IN ('Engineering', 'Sales',)
GROUP BY ALL
ORDER BY department,;
```

---

## Common Patterns

### Count by category

```sql
SELECT department, gender, count()
FROM employees
GROUP BY ALL
ORDER BY department, gender;
```

### Table statistics

```sql
SUMMARIZE employees;
```

### Quick peek with column exclusion

```sql
SELECT * EXCLUDE (id)
FROM employees LIMIT 5;
```

### Grouping with friendly alias

```sql
SELECT
    dept: department,
    headcount: COUNT(*)
FROM employees
GROUP BY ALL
ORDER BY headcount DESC;
```
