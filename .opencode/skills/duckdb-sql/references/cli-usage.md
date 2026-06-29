# DuckDB CLI Usage

Querying local files using the DuckDB command line interface.

---

## Starting the CLI

```bash
duckdb                 # transient in-memory database
duckdb my_db.duckdb   # open or create a persistent database
```

---

## Querying Local Files Directly

DuckDB can read CSV, Parquet, JSON, and XLSX files directly from the CLI using replacement scans:

```bash
duckdb :memory: "SELECT * FROM 'data.csv'"
duckdb :memory: "SELECT * FROM read_parquet('data.parquet')"
duckdb :memory: "SELECT name, COUNT() FROM 'sales.csv' GROUP BY name"
```

No `CREATE TABLE` or `IMPORT` needed — just point to the file path.

---

## Non-Interactive Execution

### Single statement with `-c`

```bash
duckdb -c "SELECT 42 AS answer"
duckdb -c "SELECT * FROM 'data.csv' LIMIT 10"
duckdb my_db.duckdb -c "SELECT COUNT() FROM employees"
```

### SQL from a file

```bash
duckdb < query.sql
duckdb :memory: < script.sql
```

### Inline with `-c` and pipe

```bash
cat data.csv | duckdb -c "SELECT * FROM read_csv('/dev/stdin')"
```

Note: `/dev/stdin` is Unix-only. On Windows, use PowerShell pipelines with `Get-Content`.

---

## Output Formats

Control how results are displayed using `.mode`:

```sql
.mode duckbox       -- default table format
.mode csv           -- CSV output
.mode json          -- JSON output
.mode markdown      -- Markdown table
.mode latex         -- LaTeX table
.mode insert        -- SQL INSERT statements
.mode line          -- One value per line
.mode column        -- Columnar format
.mode box           -- Unicode box drawing
```

Set before running a query:

```bash
duckdb -c ".mode csv SELECT * FROM 'data.csv'"
```

---

## Writing Results to a File

```sql
-- All subsequent output
.output results.csv
SELECT * FROM 'data.csv';

-- Next query only
.once output.md
.mode markdown
SELECT * FROM 'data.csv';
```

---

## Running SQL Scripts

```sql
-- Execute a file of SQL/dot commands
.read analysis.sql
```

### Startup scripts

```bash
# Run init file on startup
duckdb -init setup.sql

# Auto-loaded config file
# ~/.duckdbrc runs on every CLI start
```

Example `~/.duckdbrc`:

```
.mode duckbox
.prompt "{sql:select current_database()} > "
```

---

## Dot Commands Reference

| Command | Description |
|---------|-------------|
| `.help` | List all dot commands |
| `.open FILE` | Open a database file |
| `.exit` / `.quit` | Exit the CLI |
| `.mode MODE` | Set output format |
| `.output FILE` | Write output to file |
| `.once FILE` | Write next query output |
| `.read FILE` | Execute commands from file |
| `.tables` | List tables |
| `.schema [TABLE]` | Show table schema |
| `.timer on\|off` | Toggle query timing |
| `.maxrows N` | Limit displayed rows |

---

## Environment Variables

Read environment variables via `getenv()`:

```sql
SELECT getenv('HOME') AS home;
SELECT getenv('USER') AS user;
```

Restricted by `enable_external_access` setting (default: on). Only available in CLI, not in other clients.

---

## Loading Extensions

```sql
INSTALL excel;
LOAD excel;
```

Extensions are needed for XLSX, spatial, full-text search, etc. Autoloaded extensions load automatically on first use.

---

## Interactive CLI Features

- **Multi-line input**: omit the semicolon to continue on the next line
- **Autocomplete**: Tab-completion for SQL keywords and table/column names
- **Syntax highlighting**: Enabled on macOS, Linux, and Windows terminals
- **Up arrow**: Recall previous commands
- **`.` prefix**: All dot commands start with `.` (no semicolon needed)
