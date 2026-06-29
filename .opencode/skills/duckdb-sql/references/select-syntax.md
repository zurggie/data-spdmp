# SELECT Syntax

## Canonical Clause Order

```
WITH cte AS (...)
SELECT select_list
FROM table_source
    SAMPLE expr
WHERE condition
GROUP BY groups
HAVING group_filter
    WINDOW window_expr
    QUALIFY qualify_filter
ORDER BY order_expr
LIMIT n OFFSET m
```

Not all clauses are required — only `FROM` is mandatory (SELECT is optional).

---

## FROM-First Syntax

Omit `SELECT *` entirely:

```sql
FROM employees LIMIT 10;
```

```sql
FROM employees WHERE department = 'Engineering';
```

## Column Aliases

**Standard** — `AS` is optional:

```sql
SELECT salary * 1.1 AS raised_salary, hire_date d FROM employees;
```

**Prefix aliases** — `name:` instead of `AS name`:

```sql
SELECT salary * 1.1: raised_salary, department: department FROM employees;
```

**Lateral aliases** — reuse computed columns in the same SELECT:

```sql
SELECT salary + bonus AS total, total * 0.05 AS tax FROM employees LIMIT 5;
```

---

## Star Expressions

### `SELECT * EXCLUDE`

Drop specific columns:

```sql
SELECT * EXCLUDE (id, ssn, salary) FROM employees;
```

### `SELECT * REPLACE`

Transform columns inline:

```sql
SELECT * REPLACE (UPPER(name) AS name, salary * 1.1 AS salary) FROM employees;
```

### `COLUMNS()` with Regex

Apply expression to columns matching a pattern:

```sql
SELECT COLUMNS('salary|bonus') FROM employees;
```

### `COLUMNS()` with Lambda

```sql
SELECT COLUMNS('salary|bonus' => c -> c * 1.1) FROM employees;
```

### `COLUMNS()` with EXCLUDE/REPLACE

```sql
SELECT COLUMNS(*) EXCLUDE (id, ssn) FROM employees;
SELECT COLUMNS(*) REPLACE (salary * 1.1 AS salary) FROM employees;
```

---

## DISTINCT

```sql
SELECT DISTINCT department FROM employees;
SELECT DISTINCT ON (department) department, name, salary FROM employees ORDER BY department, salary DESC;
```

---

## Column Indexes

Reference columns by position (1-indexed):

```sql
SELECT #1, #3 FROM employees;
```

---

## Trailing Commas

Allowed in SELECT lists, VALUES, function arguments, and list literals:

```sql
SELECT
    name,
    department,
    salary,
FROM employees
WHERE department IN ('Engineering', 'Sales',)
```

---

## Dot Operator (Function Chaining)

```sql
SELECT ('hello world').upper().reverse() AS reversed;
```

---

## VALUES Clause

Standalone query:

```sql
VALUES (1, 'Alice'), (2, 'Bob'), (3, 'Charlie');
```

In FROM with column names:

```sql
SELECT * FROM (VALUES (1, 'Alice'), (2, 'Bob')) AS t(id, name);
```

In JOIN:

```sql
SELECT e.*
FROM employees e
JOIN (VALUES ('Engineering'), ('Sales')) AS d(dept) ON e.department = d.dept;
```
