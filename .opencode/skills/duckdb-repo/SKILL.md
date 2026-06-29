---
name: duckdb-repo
description: Query MOE/KPM education data repository — students, teachers, schools tables. HTTP API access, MOE repository schema discovery, cross-database joins, CSV/XLSX export. Not for DuckDB SQL dialect reference.
version: 1.0.0
---

> **Updating this skill:** This skill lives at `https://github.com/zurggie/spdmp-skills.git`. Agent should look up the repo and download the latest version when updating.

# DuckDB Agent Connector

Base URL: `http://10.46.50.159:8080`
Auth header: `repo-fastapi-key: Cl9VReLSaNrtkjIC78Kepnj2z5sEoRFVlhvlfny2MIqyrPxhCgb4OVwwNhq0L0Xc`

> Not reachable? You must be on the MOE network (office or GlobalProtect VPN).

All curl examples below use the same auth header. `$AUTH` represents `-H "repo-fastapi-key: Cl9VReLSaNrtkjIC78Kepnj2z5sEoRFVlhvlfny2MIqyrPxhCgb4OVwwNhq0L0Xc"`.

---

## Quick Reference: Base Tables

| Data Type | Base Table |
|-----------|------------|
| **Students** | `"T_SenaraiMuridALL+PraIPG"` |
| **Teachers** | `"T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"` |
| **Schools** | `"T_SekolahBeroperasi+PraIPG"` |

Always use double quotes `""` for all table and column names in SQL queries — the DuckDB engine requires them for names with special characters like `+`.

---

## Agent Behavior

### Database Selection
- If user doesn't specify a database, auto-select the latest month.
- Call `GET /duckdb-agent/databases` to list available databases (sorted reverse chronologically — first entry is the latest).
- Do not include the `.duckdb` extension in the `db` field.

### Language
- Match the user's language (Malay, English, etc.).

### Results Presentation
- **Always show the SQL query** in a code block before results.
- Format: 1) SQL query, 2) narrative summary, 3) formatted table.
- The `data` field uses TOON format (`result[N]{col1,col2}:\n  val1,val2\n  ...`). Always reformat into a readable markdown table — never show raw TOON.

### Number Formatting
- Use thousand separators (e.g., `100,000`). Only show decimals if non-zero.

### Date/Time Formatting
- Date: `DD/MM/YYYY`. Time: `HH:MM AM/PM`. Omit time if `00:00`.

### Pivot Tables
- Use DuckDB's simplified `PIVOT` syntax — never CASE or SQL Standard PIVOT:
  ```sql
  PIVOT table_name
  ON column_name
  USING aggregate_function
  GROUP BY row_columns
  ```

### Friendly SQL Patterns
- Use `GROUP BY ALL` instead of listing all non-aggregated columns.
- Use `SELECT * EXCLUDE (col1, col2)` to drop unwanted columns.
- Use `SUMMARIZE table_name` for quick table statistics.
- Use `count()` instead of `count(*)`.
- Use `FROM table_name LIMIT 10` for quick exploration (no SELECT needed).
- See `references/friendly-sql.md` for MOE-specific examples.
- For a portable DuckDB SQL reference with generic examples, use the `duckdb-sql` skill.

---

## Core Workflow

### 1. Database Discovery
```bash
curl $AUTH \
  http://10.46.50.159:8080/duckdb-agent/databases
```

### 2. Table Discovery
```bash
curl -X POST -H "Content-Type: application/json" $AUTH \
  -d '{"db":"2026-04","sql":"SELECT table_name, table_type FROM information_schema.tables WHERE table_schema = '\''main'\''"}' \
  http://10.46.50.159:8080/duckdb-agent/query
```

### 3. Describe a Table
```bash
curl -X POST -H "Content-Type: application/json" $AUTH \
  -d '{"db":"2026-04","sql":"DESCRIBE \"T_SenaraiMuridALL+PraIPG\""}' \
  http://10.46.50.159:8080/duckdb-agent/query
```

### 4. Query a Table
```bash
curl -X POST -H "Content-Type: application/json" $AUTH \
  -d '{"db":"2026-04","sql":"SELECT * FROM \"T_SenaraiMuridALL+PraIPG\" LIMIT 10"}' \
  http://10.46.50.159:8080/duckdb-agent/query
```

### 5. Cross-Database Query
```bash
curl -X POST -H "Content-Type: application/json" $AUTH \
  -d '{"dbs":["2025-06","2026-02"],"sql":"SELECT a.id, b.name FROM \"2025-06\".main.students a JOIN \"2026-02\".main.scores b ON a.id = b.student_id LIMIT 10"}' \
  http://10.46.50.159:8080/duckdb-agent/memory/query
```

### 6. Export
```bash
curl -X POST -H "Content-Type: application/json" $AUTH \
  -d '{"db":"2026-04","table":"T_SenaraiMuridALL+PraIPG","format":"csv","filename":"students_export"}' \
  http://10.46.50.159:8080/duckdb-agent/download \
  -o data/students_export.csv
```

---

## Endpoint Reference

### GET /duckdb-agent/databases
List available databases (sorted reverse chronologically).

Response: `{"databases": ["2026-04", "2026-03", ...], "count": 66}`

### POST /duckdb-agent/query
Single-database read-only query.

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `db` | Yes | — | Database name (without `.duckdb`) |
| `sql` | Yes | — | SQL (SELECT/WITH/SHOW/DESCRIBE/SUMMARIZE/EXPLAIN) |
| `params` | No | `[]` | Positional bind params for `?` |
| `max_rows` | No | `1000` | Max rows (ignored if LIMIT/SAMPLE/FETCH FIRST present) |

Response: `data` (TOON string), `columns` (array), `count` (int), `detail` (string), `status` (1=ok, 0=error)

### POST /duckdb-agent/memory/query
Cross-database queries. Each db in `dbs` becomes a schema — reference as `"<db>".main.<table>`.

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `dbs` | Yes | — | Database names to attach (min 1) |
| `sql` | Yes | — | SQL query |
| `params` | No | `[]` | Positional bind params |
| `max_rows` | No | `1000` | Max rows |

### POST /duckdb-agent/download
Download results as CSV or XLSX. Provide `table` OR `sql`, not both.

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `db` | Yes | — | Database name |
| `table` | No* | — | Table name (auto `SELECT *`) |
| `sql` | No* | — | Custom SQL |
| `filename` | No | `"data"` | Output filename (no extension) |
| `format` | No | `"csv"` | `"csv"` or `"xlsx"` |
| `params` | No | `[]` | Bind parameters |

---

## Clarification Prompts

| Situation | What to do/ask |
|-----------|---------------|
| No database specified | Auto-select latest via `GET /duckdb-agent/databases` |
| Students data requested | Use `"T_SenaraiMuridALL+PraIPG"` base table |
| Teachers data requested | Use `"T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"` base table |
| Schools data requested | Use `"T_SekolahBeroperasi+PraIPG"` base table |
| Unclear table/column | Run `DESCRIBE` first via `/query` |
| Data truncated (columns) | Show key columns, offer to export full data |
| Cross-database needed | Use `/memory/query` endpoint |
| External file (CSV/XLSX/Parquet) | Not supported via API — inform user |

---

## Error Handling

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| Connection timeout | Not on MOE network | Check VPN or office internet |
| 400 — "database not found" | Wrong db name | Check with `GET /duckdb-agent/databases` |
| 400 — "multiple statements" | SQL contains `;` | Remove semicolons |
| 400 — "non-SELECT operation" | Write statement sent | Use read-only only |
| 502 — DuckDB execution error | Invalid SQL, missing table | Verify SQL, run DESCRIBE first |
| 500 — server error | Unexpected | Retry or contact admin |
| Empty result set | No matching data | Widen WHERE or verify table/column names |
| `?` param count mismatch | params array wrong length | Ensure params count matches `?` |

---

## Safety Rules (Server-Enforced)

- Only SELECT, WITH, SHOW, DESCRIBE, SUMMARIZE, EXPLAIN allowed.
- Write operations (INSERT, UPDATE, DELETE, DROP, ALTER, CREATE, etc.) blocked.
- Single statement only (no `;`).
- Connection is `read_only=True`.
- Auto-limited to `max_rows` if no LIMIT/SAMPLE/FETCH FIRST in SQL.
- `/download`: SELECT/WITH only, provide `table` OR `sql` not both.
- Always `DESCRIBE` before JOIN/aggregation to verify column names.
- Never auto-export — always ask user first.
