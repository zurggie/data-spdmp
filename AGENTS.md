# AGENTS.md

## Project

MOE/KPM education data querying using DuckDB — both **local** (`.duckdb` files in `databases/`) and **remote** (warehouse API). Access student, teacher, school, infrastructure, and subject data.

## Two Ways to Query

### Local (CLI with `duckdb.exe`)

Use `duckdb.exe` in the project root to query `.duckdb` files in the `databases/` folder.

**Discover available databases:**
```
.\duckdb.exe
```

**Interactive mode:**
```powershell
.\duckdb.exe databases\2021-06.duckdb
```
Then type SQL. `.quit` to exit.

**One-shot query:**
```powershell
.\duckdb.exe -noheader -csv databases\2021-06.duckdb "SELECT * FROM information_schema.tables WHERE table_schema = 'main' LIMIT 5"
```

**Piped SQL (multi-line):**
```powershell
"SELECT COUNT(*) FROM `"T_SenaraiMuridALL+PraIPG`"" | .\duckdb.exe databases\2021-06.duckdb
```

For local queries, only **kpm-data-model** (what to query) and **duckdb-sql** (how to write SQL) are needed. duckdb-repo is not required.

### Remote (warehouse API)

When querying the MOE warehouse via HTTP, follow the workflow in **duckdb-repo** — discover databases, DESCRIBE tables, query, export. Requires MOE network/VPN.

## Skills

| Skill | Path | Use For |
|-------|------|---------|
| **duckdb-repo** | `.opencode/skills/duckdb-repo/` | API endpoints, auth, databases discovery, curl mechanics |
| **kpm-data-model** | `.opencode/skills/kpm-data-model/` | Table catalog, column names, join patterns, domain recipes |
| **duckdb-sql** | `.opencode/skills/duckdb-sql/` | DuckDB SQL dialect (PIVOT, GROUP BY ALL, functions) |

**Load all three** before querying. `duckdb-repo` tells you *how*, `kpm-data-model` tells you *what*, `duckdb-sql` tells you *syntax*.

## Critical Conventions

- **Exclude PraIPG** (school type `408`) by default in school/student queries
- **Double-quote** all table and column names: `"NEGERI"`, `"T_SenaraiMuridALL+PraIPG"`
- Use `GROUP BY ALL` instead of listing all non-aggregated columns
- **Never auto-show SQL** — show only if user asks
- **Never auto-export raw data** — only export if user explicitly asks  
- Export/import data goes in `files/` folder (CSV, Parquet, XLSX)
- **When asked to export to XLSX**, design it professionally with styling (headers, borders, formatting) suitable for formal presentation
- **Always DESCRIBE** a table before joining to verify column names
- Single statement only (no `;`)
- **Plain language first with audit trail**: When showing results, explain in **plain natural language** what was queried and why. Be explicit about: (a) which tables were joined and how (join keys, join type), (b) every filter applied and why, (c) how the numbers relate across tables (e.g. "this count is lower because we filtered out school type 408"). The goal is to surface any **anomaly in filtering or joining** so the user can catch mistakes early. Only show raw SQL if asked.
- **Always cleanup temp scripts** — delete any temporary scripts/files after use.
- **Before creating any new temp script**, check for old leftover temp scripts that were forgotten; delete them first.
- **Never sum/aggregate using the LLM** — when results contain totals, sums, counts, or any aggregation, always compute them in SQL (`SUM()`, `COUNT()`, etc.) or via a tool (e.g. `count`). Never sum, average, or tally numbers from row output by hand. The LLM is unreliable for arithmetic over many rows.
- **Always use DuckDB to query CSV** — never use PowerShell string-search tools (`Select-String`, `findstr`, `grep`) on CSV data. Use DuckDB's native CSV reading (e.g. `SELECT * FROM read_csv_auto('files/...')`) to query, filter, aggregate, and join. Python/pandas is acceptable.

## Pre-Flight

- [ ] Connected to MOE network (office or GlobalProtect VPN) — **remote only**
- [ ] Base URL: `http://10.46.50.159:8080`
- [ ] Read-only: no INSERT, UPDATE, DELETE, DROP, ALTER, CREATE

Skills provide specialized instructions and workflows for specific tasks.
Use the skill tool to load a skill when a task matches its description.
