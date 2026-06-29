---
name: duckdb-sql
description: DuckDB SQL dialect reference for querying — SELECT, FROM-first, star expressions, GROUP BY ALL, PIVOT, window functions, CTEs, joins, expressions, meta queries, inline file reading. Not for MOE/KPM warehouse operations.
version: 1.0.0
---

# DuckDB SQL Reference

DuckDB's SQL dialect reference for **querying only** — SELECT, VIEWs, and MACROs that help queries, but no mutation statements (INSERT, UPDATE, DELETE, DROP, ALTER, CREATE TABLE).

All example tables use generic names like `employees`, `sales`, `products`.

Quoted identifiers use double quotes `"like this"`.

---

## Reference Files

| Topic | File | Covers |
|-------|------|--------|
| SELECT syntax | `references/select-syntax.md` | Clause order, FROM-first, column aliases, star expressions, `COLUMNS()`, DISTINCT, column indexes, trailing commas, dot operator, VALUES |
| Joins | `references/joins.md` | INNER/OUTER/CROSS, ASOF, POSITIONAL, LATERAL, NATURAL, USING, column name shorthands |
| Grouping & aggregation | `references/group-by-and-aggregation.md` | `GROUP BY ALL`, grouping sets, `FILTER`, aggregate functions, top-N, window functions, `QUALIFY` |
| WHERE, ORDER BY, LIMIT, SAMPLE | `references/where-order-by-limit.md` | `WHERE`, column aliases, `ORDER BY ALL`, `LIMIT`, percentage, `OFFSET`, `SAMPLE` clause |
| PIVOT / UNPIVOT | `references/pivot-unpivot.md` | Long-to-wide and wide-to-long transformations |
| CTE & set operations | `references/cte-and-setops.md` | `WITH`, recursive CTE, `UNION BY NAME`, `INTERSECT`, `EXCEPT` |
| Meta queries | `references/meta-queries.md` | `DESCRIBE`, `SUMMARIZE`, `SHOW`, `EXPLAIN ANALYZE`, information_schema, metadata functions |
| Expressions & operators | `references/expressions.md` | `CASE`/`SWITCH`, subqueries, `IN`, `TRY`/`TRY_CAST`, casting, logical ops, pattern matching, regex |
| Functions | `references/functions.md` | Text, date/time, numeric, nested/list/struct, lambda, and utility functions |
| File I/O, views, macros | `references/file-io-views-macros.md` | Inline CSV/Parquet/JSON/XLSX reading, globbing, `CREATE VIEW`, `CREATE MACRO`, `SET VARIABLE` |
| CLI usage | `references/cli-usage.md` | Starting DuckDB, querying local files, non-interactive execution, output formats, dot commands, scripts |
| Friendly SQL patterns | `references/friendly-sql.md` | Concise DuckDB shorthand patterns with examples |
