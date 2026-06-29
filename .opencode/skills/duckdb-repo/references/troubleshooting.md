# DuckDB Agent — Troubleshooting Guide

## Network Connectivity

### "Connection refused" or timeout
The API at `http://10.46.50.159:8080` is only reachable from within the MOE network.

**Checklist:**
1. Are you connected to the office internet? (direct LAN or Wi-Fi)
2. Are you using GlobalProtect VPN? The VPN must be connected and active.
3. Can you ping the host? `ping 10.46.50.159` — no reply means you're not on the network.
4. Is the API server running? If you get no response, the service may be down — inform the user.

### Intermittent connectivity
- VPN may have disconnected or reconnected. Ask the user to reconnect.
- Office Wi-Fi networks can be unstable. Suggest a wired connection or retry.

---

## API Errors

### 400 — "Database not found"
**Cause:** The `db` field contains a name that doesn't match any `.duckdb` file in the `data/` directory.
**Fix:** Run `GET /duckdb-agent/databases` to see available databases. Database names are date-based (e.g., `2025-06`). Do not include the `.duckdb` extension.

### 400 — "Multiple statements are not allowed"
**Cause:** SQL string contains `;` which separates statements.
**Fix:** Remove all semicolons. Only one statement per request.

### 400 — "Only read-only operations are allowed"
**Cause:** You sent a write statement (INSERT, UPDATE, DELETE, DROP, ALTER, CREATE, etc.).
**Fix:** DuckDB agent is read-only by design. Use SELECT, WITH, SHOW, DESCRIBE, SUMMARIZE, or EXPLAIN only.

### 400 — "Empty dbs list"
**Cause:** The `dbs` array in `/memory/query` is empty.
**Fix:** Provide at least one database name in the `dbs` array.

### 502 — "DuckDB execution error"
**Causes:**
- Invalid SQL syntax (wrong column name, table name, missing keyword)
- Table or column doesn't exist in the specified database
- Bind parameter count doesn't match `?` placeholders
- Division by zero or other runtime SQL error

**Fix:**
1. Review the SQL for syntax errors.
2. Verify that `params` array count matches `?` placeholders.
3. Check table and column names with `DESCRIBE` or `information_schema`.
4. Test with simpler query first (e.g., `SELECT * FROM table LIMIT 1`).
5. Confirm the database name is correct.

### 500 — Internal Server Error
**Cause:** Unexpected server-side issue (not your SQL).
**Fix:** Retry the request. If persistent, inform the user that the API may be down.

### Empty result set
**Causes:**
- No data matches the WHERE conditions
- Wrong table or column name (check spelling and case)
- Wrong database

**Fix:**
1. Run `SELECT * FROM table LIMIT 3` to verify the table has data.
2. Check table columns with `DESCRIBE`.
3. Widen search conditions (remove WHERE, use LIKE or ILIKE instead of =).

---

## Authentication

### "Invalid API key" / 401
The API key in the header `repo-fastapi-key` is hardcoded in the skill. If it stops working:
1. The key may have been rotated — contact the API administrator for a new key.
2. Check that the header name is exactly `repo-fastapi-key`.

---

## Query Issues

### Cross-database query fails with "Table not found"
**Cause:** Tables in attached databases must be referenced with fully qualified names.
**Fix:** Use `"<db_name>".main.<table_name>` syntax. Example:
```sql
SELECT * FROM "2025-06".main.students
```

### Unexpected row limit
**Cause:** Auto-limit of `max_rows` (default 1000) was applied because no LIMIT/SAMPLE/FETCH FIRST clause was present.
**Fix:** Add an explicit `LIMIT N` to control the row count, or increase `max_rows` in the request body.

### Download returns empty file
**Cause:** The SQL or table name doesn't match any data.
**Fix:** First test the query with `POST /duckdb-agent/query` to verify it returns results, then use the same parameters for download.

---

## Best Practices

1. **Always start with discovery** — list databases, list tables, describe tables before querying.
2. **Use bind parameters** — use `params` array with `?` placeholders, never interpolate values into SQL strings.
3. **Qualify table names** — in cross-database queries, always use `"<db>".main.<table>` syntax.
4. **Check row counts first** — before potentially large queries, run `SELECT COUNT(*)` to set expectations.
5. **Handle errors gracefully** — if an API call fails, show the error to the user and suggest the next step.
6. **Prefer `/memory/query` for joins** — when joining data from multiple databases, use the memory endpoint instead of separate queries.
