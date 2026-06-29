# PIVOT / UNPIVOT

## PIVOT

Long table → wide table. DuckDB uses simplified syntax — never CASE or SQL Standard PIVOT.

### Basic

```sql
PIVOT employees
ON department
USING COUNT(*)
GROUP BY gender;
```

### With explicit values

```sql
PIVOT employees
ON department IN ('Engineering', 'Sales', 'Marketing')
USING AVG(salary) AS avg_salary
GROUP BY gender;
```

### Multiple aggregates

```sql
PIVOT employees
ON department
USING AVG(salary) AS avg_salary,
      COUNT(*) AS headcount
GROUP BY gender;
```

### Multiple ON columns

```sql
PIVOT employees
ON department, gender
USING COUNT(*);
```

### PIVOT Long (nested)

```sql
PIVOT employees
ON department
USING COUNT(*)
GROUP BY gender
ORDER BY gender;
```

### Inline in SELECT

```sql
SELECT *
FROM employees
PIVOT (
    COUNT(*) FOR department IN ('Engineering', 'Sales', 'Marketing')
) AS p;
```

---

## UNPIVOT

Wide table → long table.

### Basic

```sql
UNPIVOT monthly_sales
ON jan, feb, mar
INTO NAME month VALUE revenue;
```

### Multiple value columns

```sql
UNPIVOT monthly_sales
ON jan, feb, mar
INTO NAME month VALUE revenue
GROUP BY region;
```

### Inline in SELECT

```sql
SELECT *
FROM monthly_sales
UNPIVOT (
    revenue FOR month IN (jan, feb, mar)
) AS u;
```

### Include NULLs

By default, UNPIVOT excludes NULLs. Include them with `INCLUDE NULLS`:

```sql
UNPIVOT monthly_sales
ON jan, feb, mar
INTO NAME month VALUE revenue
INCLUDE NULLS;
```
