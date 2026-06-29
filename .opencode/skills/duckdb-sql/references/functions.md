# Functions Quick Reference

## Text Functions

| Expression | Result |
|------------|--------|
| `UPPER('hello')` | `HELLO` |
| `LOWER('HELLO')` | `hello` |
| `LENGTH('hello')` | `5` |
| `TRIM('  hi  ')` | `hi` |
| `CONCAT('a', 'b')` or `'a' \|\| 'b'` | `ab` |
| `CONCAT_WS(', ', 'a', 'b')` | `a, b` |
| `SUBSTRING('hello', 2, 3)` | `ell` |
| `LEFT('hello', 2)` | `he` |
| `RIGHT('hello', 2)` | `lo` |
| `REPLACE('hello', 'l', 'x')` | `hexxo` |
| `REVERSE('hello')` | `olleh` |
| `REPEAT('ha', 3)` | `hahaha` |
| `SPLIT_PART('a,b,c', ',', 2)` | `b` |
| `FORMAT('Hello {}', name)` | `Hello Alice` |
| `PRINTF('%s has %d', name, count)` | `Alice has 5` |

---

## Date / Timestamp Functions

| Expression | Result |
|------------|--------|
| `NOW()` | Current timestamp |
| `CURRENT_DATE` | Today's date |
| `CURRENT_TIMESTAMP` | Current timestamp |
| `DATE_TRUNC('month', hire_date)` | First of month |
| `DATE_PART('year', hire_date)` | Year component |
| `DATE_DIFF('day', start, end)` | Days between |
| `EXTRACT(YEAR FROM hire_date)` | Year component |
| `YEAR(hire_date)`, `MONTH(...)`, `DAY(...)` | Component extraction |
| `hire_date + INTERVAL 1 DAY` | Date arithmetic |
| `AGE(birth_date)` | Age in years |
| `'2024-01-15'::DATE` | String to date |
| `TO_DATE('2024-01-15', 'YYYY-MM-DD')` | String to date |

---

## Numeric Functions

| Expression | Result |
|------------|--------|
| `ABS(-5)` | `5` |
| `CEIL(4.2)` | `5` |
| `FLOOR(4.8)` | `4` |
| `ROUND(4.567, 2)` | `4.57` |
| `POWER(2, 3)` | `8` |
| `SQRT(9)` | `3` |
| `5 % 2` or `MOD(5, 2)` | `1` |
| `SIGN(-5)` | `-1` |
| `GREATEST(3, 7, 2)` | `7` |
| `LEAST(3, 7, 2)` | `2` |
| `RANDOM()` | Random [0, 1) |
| `UNIFORM(0, 100)` | Random integer [lo, hi] |

---

## Nested / List / Struct Functions

| Expression | Result |
|------------|--------|
| `[1, 2, 3]` | List literal |
| `list_value(1, 2, 3)` | List from values |
| `list_agg(col)` | Collect into list |
| `list_sort(l)` | Sorted list |
| `list_distinct(l)` | Unique elements |
| `list_filter(l, x -> x > 1)` | Filtered list |
| `list_transform(l, x -> x * 2)` | Transformed list |
| `list_contains(l, 3)` | Contains check |
| `l[2]` | Index (1-based) |
| `l[-1]` | Last element |
| `l[1:3]` | Slice |

| Expression | Result |
|------------|--------|
| `{a: 1, b: 2}` | Struct literal |
| `row(1, 'a')` | Row/struct |
| `struct_pack(a := 1, b := 2)` | Struct from key-value |
| `s.a` | Field access |
| `struct_extract(s, 'a')` | Field extraction |

| Expression | Result |
|------------|--------|
| `map({'a': 1, 'b': 2})` | Map literal |
| `m['a']` | Value lookup |

---

## Lambda Functions

```sql
-- Filter
SELECT list_filter([1, 2, 3, 4], x -> x > 2);   -- [3, 4]

-- Transform
SELECT list_transform([1, 2, 3], x -> x * 10);   -- [10, 20, 30]

-- Reduce
SELECT list_reduce([1, 2, 3], (x, y) -> x + y);  -- 6

-- Combine
SELECT list_transform(
    list_filter([1, 2, 3, 4], x -> x % 2 = 0),
    x -> x * x
);                                                 -- [4, 16]
```

---

## Utility Functions

| Expression | Description |
|------------|-------------|
| `typeof(expr)` | Return data type name |
| `hash(expr)` | Hash value |
| `least(a, b, ...)` | Minimum of args |
| `greatest(a, b, ...)` | Maximum of args |
| `coalesce(a, b, ...)` | First non-null |
| `ifnull(a, b)` | Same as coalesce |
| `nullif(a, b)` | NULL if a = b |
| `typeof(expr)` | Data type as text |
| `uuid()` | Random UUID |
| `gen_random_uuid()` | Random UUID (same) |
| `seq8()` / `seq4()` / `seq2()` | Sequential integers |
