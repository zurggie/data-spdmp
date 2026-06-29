# Meta Queries

## DESCRIBE

Show column schema for a table or query:

```sql
DESCRIBE employees;
```

```sql
DESCRIBE SELECT name, salary / 12 AS monthly FROM employees;
```

---

## SUMMARIZE

Compute summary statistics (min, max, mean, null count, distinct count, etc.):

```sql
SUMMARIZE employees;
```

```sql
SUMMARIZE (SELECT * FROM employees WHERE department = 'Engineering');
```

---

## SHOW

List objects in the catalog:

```sql
SHOW TABLES;
SHOW DATABASES;
SHOW SCHEMAS;
SHOW ALL TABLES;
SHOW COLUMNS FROM employees;
```

---

## EXPLAIN

View the query plan:

```sql
EXPLAIN SELECT department, AVG(salary) FROM employees GROUP BY department;
```

Profile execution:

```sql
EXPLAIN ANALYZE SELECT department, AVG(salary) FROM employees GROUP BY department;
```

---

## Information Schema

Standard SQL metadata:

```sql
SELECT * FROM information_schema.tables WHERE table_schema = 'main';
SELECT * FROM information_schema.columns WHERE table_name = 'employees';
SELECT * FROM information_schema.schemata;
SELECT * FROM information_schema.views;
```

---

## Metadata Table Functions

DuckDB-specific introspection:

```sql
SELECT * FROM duckdb_tables();
SELECT * FROM duckdb_columns();
SELECT * FROM duckdb_views();
SELECT * FROM duckdb_indexes();
SELECT * FROM duckdb_schemas();
SELECT * FROM duckdb_databases();
SELECT * FROM duckdb_extensions();
SELECT * FROM duckdb_settings();
```
