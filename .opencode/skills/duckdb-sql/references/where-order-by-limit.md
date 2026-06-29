# WHERE, ORDER BY, LIMIT, SAMPLE

## WHERE

Standard filtering. Column aliases from SELECT are usable:

```sql
SELECT salary / 12 AS monthly
FROM employees
WHERE monthly > 4000;
```

---

## ORDER BY

```sql
SELECT name, salary FROM employees ORDER BY salary DESC;

-- ORDER BY ALL — sort by every column in SELECT
SELECT * FROM employees ORDER BY ALL;

-- NULLS control
SELECT name, salary FROM employees ORDER BY salary NULLS LAST;
SELECT name, salary FROM employees ORDER BY salary NULLS FIRST;
```

---

## LIMIT / OFFSET

```sql
SELECT * FROM employees LIMIT 10;
SELECT * FROM employees LIMIT 10 OFFSET 20;

-- Percentage-based LIMIT
SELECT * FROM employees LIMIT 10%;
SELECT * FROM employees LIMIT 10% OFFSET 5;

-- FETCH FIRST syntax (SQL standard)
SELECT * FROM employees FETCH FIRST 10 ROWS ONLY;
SELECT * FROM employees FETCH FIRST 10 PERCENT ROWS ONLY;
```

---

## SAMPLE Clause

Placed after FROM / JOIN, before WHERE:

```sql
-- By percentage
SELECT * FROM employees USING SAMPLE 10%;
SELECT * FROM employees USING SAMPLE 10 PERCENT;

-- By row count
SELECT * FROM employees USING SAMPLE 100 ROWS;

-- Reservoir sampling (default — exact count, random order)
SELECT * FROM employees USING SAMPLE reservoir(100 ROWS);

-- System sampling (page-level — faster, less random)
SELECT * FROM employees USING SAMPLE system(10 PERCENT);

-- Bernoulli sampling (row-level — slower, more random)
SELECT * FROM employees USING SAMPLE bernoulli(10 PERCENT);
```

`SAMPLE` can also be used as a top-level statement:

```sql
SELECT * FROM employees SAMPLE 10%;
```
