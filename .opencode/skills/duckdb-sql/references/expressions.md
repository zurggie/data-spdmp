# Expressions & Operators

## CASE / SWITCH

### CASE (standard SQL)

```sql
SELECT
    name, salary,
    CASE WHEN salary > 100000 THEN 'High'
         WHEN salary > 50000 THEN 'Medium'
         ELSE 'Low'
    END AS salary_band
FROM employees;
```

### SWITCH (DuckDB shorthand)

Cleaner syntax for equality checks:

```sql
SELECT
    name, department,
    SWITCH department
        WHEN 'Engineering' THEN 'EN'
        WHEN 'Sales' THEN 'SA'
        WHEN 'Marketing' THEN 'MK'
        ELSE 'OT'
    END AS dept_code
FROM employees;
```

---

## Subqueries

### Scalar subquery

```sql
SELECT name, salary,
    (SELECT AVG(salary) FROM employees) AS company_avg
FROM employees;
```

### EXISTS / NOT EXISTS

```sql
SELECT name FROM employees e
WHERE EXISTS (SELECT 1 FROM sales s WHERE s.emp_id = e.id);
```

### IN / NOT IN (subquery)

```sql
SELECT name FROM employees
WHERE department IN (SELECT name FROM active_departments);
```

### ANY / ALL

```sql
SELECT name FROM employees
WHERE salary > ANY (SELECT salary FROM executives);
```

---

## IN Operator

```sql
SELECT * FROM employees WHERE department IN ('Engineering', 'Sales');
SELECT * FROM employees WHERE department NOT IN ('Intern', 'Contractor');
SELECT * FROM employees WHERE id IN (SELECT emp_id FROM bonus_list);
```

---

## TRY / TRY_CAST

Return NULL instead of throwing on error:

```sql
SELECT TRY(1 / 0) AS safe_divide;           -- NULL

SELECT TRY_CAST('not-a-number' AS INTEGER);  -- NULL
SELECT TRY_CAST('123' AS INTEGER);           -- 123
```

---

## Casting

```sql
-- Two syntaxes
SELECT CAST(salary AS DOUBLE) AS sal_double FROM employees;
SELECT salary::DOUBLE AS sal_double FROM employees;
SELECT salary::INTEGER AS sal_int FROM employees;
SELECT hire_date::DATE FROM employees;
```

---

## Logical & Comparison Operators

```sql
-- Logical
WHERE a > 10 AND b < 5
WHERE a > 10 OR b < 5
WHERE NOT a > 10

-- Comparison
WHERE a = 5, a != 5, a <> 5
WHERE a IS NULL, a IS NOT NULL
WHERE a BETWEEN 10 AND 20
WHERE a NOT BETWEEN 10 AND 20
WHERE name LIKE 'A%'
WHERE name NOT LIKE 'A%'
WHERE name ILIKE 'a%'  -- case-insensitive LIKE
```

---

## Pattern Matching / Regex

```sql
-- Glob patterns
SELECT * FROM employees WHERE name GLOB 'A*';
SELECT * FROM employees WHERE name NOT GLOB '[ab]*';

-- Regex
SELECT * FROM employees WHERE department ~ '^Eng';
SELECT * FROM employees WHERE department !~ '^Sales';

-- Regex functions
SELECT regexp_matches('hello 123', '\d+');
SELECT regexp_replace('hello 123', '\d+', 'world');
SELECT regexp_extract('hello 123', '(\d+)', 1);
```
