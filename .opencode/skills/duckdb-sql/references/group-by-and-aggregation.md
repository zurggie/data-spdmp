# Grouping & Aggregation

## GROUP BY ALL

Infer non-aggregated columns from SELECT list:

```sql
SELECT department, gender, COUNT()
FROM employees
GROUP BY ALL
```

vs standard:

```sql
SELECT department, gender, COUNT()
FROM employees
GROUP BY department, gender
```

---

## GROUPING SETS / CUBE / ROLLUP

```sql
-- Multiple grouping sets
SELECT department, gender, COUNT()
FROM employees
GROUP BY GROUPING SETS ((department), (gender), ());

-- CUBE — all combinations
SELECT department, gender, COUNT()
FROM employees
GROUP BY CUBE (department, gender);

-- ROLLUP — hierarchical aggregation
SELECT department, gender, COUNT()
FROM employees
GROUP BY ROLLUP (department, gender);
```

---

## HAVING

```sql
SELECT department, COUNT()
FROM employees
GROUP BY department
HAVING COUNT() > 10;
```

Column aliases from SELECT are usable in HAVING:

```sql
SELECT department, COUNT() AS cnt
FROM employees
GROUP BY department
HAVING cnt > 10;
```

---

## FILTER Clause

Conditional aggregation without CASE:

```sql
SELECT
    department,
    COUNT() AS total,
    COUNT() FILTER (WHERE salary > 50000) AS above_50k,
    AVG(salary) FILTER (WHERE age < 30) AS avg_salary_young
FROM employees
GROUP BY ALL;
```

---

## Aggregate Functions Quick Reference

| Function | Description |
|----------|-------------|
| `count()` | Row count (`count(*)` equivalent) |
| `count(DISTINCT col)` | Distinct count |
| `sum(col)`, `avg(col)` | Sum, average |
| `min(col)`, `max(col)` | Minimum, maximum |
| `list(col)` or `array_agg(col)` | Collect into list |
| `string_agg(col, ',')` | Concatenate strings |
| `bool_and(col)`, `bool_or(col)` | Boolean aggregation |
| `mode(col)` | Most frequent value |
| `median(col)` | Median value |
| `quantile_cont(col, 0.5)` | Continuous quantile |
| `approx_count_distinct(col)` | Approximate distinct count |
| `stats(col)` | Summary statistics as struct |

---

## Top-N in Group

Without window functions:

```sql
SELECT department, max(salary, 3) AS top_3_salaries
FROM employees
GROUP BY department;
```

Returns an array of the top N values per group. Also available:
- `min(arg, n)`
- `arg_max(arg, val, n)`, `arg_min(arg, val, n)`
- `max_by(arg, val, n)`, `min_by(arg, val, n)`

---

## WINDOW Clause

```sql
SELECT
    name, department, salary,
    RANK() OVER w AS rank_in_dept,
    AVG(salary) OVER w AS avg_dept_salary
FROM employees
WINDOW w AS (PARTITION BY department ORDER BY salary DESC);
```

### Window Functions

| Function | Purpose |
|----------|---------|
| `row_number()` | Sequential row number |
| `rank()`, `dense_rank()` | Ranking with gaps / no gaps |
| `lag(col, n)`, `lead(col, n)` | Previous / next row value |
| `first_value(col)`, `last_value(col)` | First / last in window frame |
| `nth_value(col, n)` | Nth value in window frame |
| `ntile(n)` | Bucket rows into n buckets |
| `cume_dist()`, `percent_rank()` | Relative rank |
| `LAG/LEAD(expr IGNORE NULLS)` | Skip nulls |

Any aggregate function can also be used with `OVER (...)`.

---

## QUALIFY Clause

Filter on window function results (like HAVING for windows):

```sql
SELECT name, department, salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS r
FROM employees
QUALIFY r <= 3;
```
