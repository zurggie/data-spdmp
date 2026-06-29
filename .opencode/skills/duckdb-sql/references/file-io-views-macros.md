# File I/O, Views & Macros

## Inline File Reading

Read CSV, Parquet, JSON directly without loading into a table:

### CSV

```sql
SELECT * FROM 'data.csv';
SELECT * FROM read_csv('data.csv');
SELECT * FROM read_csv_auto('data.csv');

-- With options
SELECT * FROM read_csv('data.csv', header := true, sep := '|');
```

### Parquet

```sql
SELECT * FROM 'data.parquet';
SELECT * FROM read_parquet('data.parquet');

-- Query specific columns
SELECT name, salary FROM 'data.parquet' WHERE salary > 50000;
```

### JSON

```sql
SELECT * FROM read_json('data.json');
SELECT * FROM read_json_auto('data.json');
```

### Excel (XLSX)

The `excel` extension is autoloaded on first use. XLSX reading via replacement scan:

```sql
-- Auto-detected from extension
SELECT * FROM 'data.xlsx';

-- With read_xlsx function
SELECT * FROM read_xlsx('data.xlsx');

-- Read a specific sheet
SELECT * FROM read_xlsx('data.xlsx', sheet := 'Sheet1');

-- Read a range
SELECT * FROM read_xlsx('data.xlsx', sheet := 'Sheet1', range := 'A1:D100');

-- Options: header, all_varchar, ignore_errors, stop_at_empty, empty_as_varchar
SELECT * FROM read_xlsx('data.xlsx', header := false, all_varchar := true, sheet := 'Students');
```

Write query results to XLSX:

```sql
-- Write to a new file
COPY (
  SELECT name, department, salary FROM employees WHERE salary > 50000
) TO 'output.xlsx' WITH (FORMAT xlsx, HEADER true);

-- Write with custom sheet name
COPY (
  SELECT * FROM high_earners
) TO 'output.xlsx' WITH (FORMAT xlsx, SHEET 'High Earners', HEADER true);
```

Note: `.xls` files (legacy Excel format) are **not** supported by DuckDB.

---

## Globbing

Read multiple files with pattern matching:

```sql
-- Single directory
SELECT * FROM 'data/part-*.parquet';
SELECT * FROM 'data/*.csv';

-- Recursive
SELECT * FROM 'data/**/*.parquet';

-- Multiple explicit files
SELECT * FROM read_csv(['file1.csv', 'file2.csv']);

-- XLSX glob
SELECT * FROM st_load('data/*.xlsx');
```

---

## Replacement Scans

DuckDB auto-detects format from file extension. Just use the filename:

```sql
-- All of these work without read_* functions
FROM 'data.csv';
FROM 'data.csv.gz';
FROM 'data.parquet';
FROM 'data.json';
FROM 'data.xlsx';    -- autoloaded via excel extension
```

---

## CREATE VIEW

Persist a query as a virtual table:

```sql
CREATE VIEW high_earners AS
SELECT name, department, salary
FROM employees
WHERE salary > 100000;

-- Create or replace
CREATE OR REPLACE VIEW high_earners AS
SELECT name, department, salary
FROM employees
WHERE salary > 120000;

-- Query like a table
SELECT * FROM high_earners;
```

---

## CREATE MACRO

Define reusable scalar expressions or table functions:

```sql
-- Scalar macro
CREATE MACRO add(a, b) AS a + b;
SELECT add(2, 3);  -- 5

-- Expression macro
CREATE MACRO annual_salary(monthly) AS monthly * 12;
SELECT name, annual_salary(salary / 12) FROM employees;

-- Table macro (returns a query)
CREATE MACRO top_n(n) AS TABLE
SELECT * FROM employees ORDER BY salary DESC LIMIT n;

SELECT * FROM top_n(5);
```

---

## SET VARIABLE

Session-scoped variables for queries:

```sql
SET VARIABLE min_salary = 50000;
SET VARIABLE dept = 'Engineering';

SELECT name, salary
FROM employees
WHERE salary >= $min_salary AND department = $dept;

-- Remove variable
RESET VARIABLE min_salary;
```
