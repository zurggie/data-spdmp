# DuckDB Friendly SQL Reference

Quick reference for DuckDB's concise SQL syntax. Use these patterns to write cleaner queries.

---

## Aggregations

### GROUP BY ALL
Omit listing all non-aggregated columns:
```sql
SELECT negeri, jantina, COUNT(*) 
FROM "T_SenaraiMuridALL+PraIPG" 
GROUP BY ALL
```

### count() Shorthand
`count()` = `count(*)`:
```sql
SELECT negeri, count() 
FROM "T_SenaraiMuridALL+PraIPG" 
GROUP BY negeri
```

### Top-N in Group
Get top N values per group without window functions:
```sql
SELECT negeri, max(gaji, 5) AS top_5_gaji
FROM "T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"
GROUP BY negeri
```

---

## Column Selection

### SELECT * EXCLUDE
Drop specific columns from `*`:
```sql
SELECT * EXCLUDE (IDMURID, IDKELAS, STATUSOKU)
FROM "T_SenaraiMuridALL+PraIPG"
LIMIT 10
```

### SELECT * REPLACE
Transform columns inline:
```sql
SELECT * REPLACE (
  UPPER(negeri) AS negeri,
  KETERANGANALIRAN AS aliran
)
FROM "T_SenaraiMuridALL+PraIPG"
LIMIT 10
```

### COLUMNS() with Regex
Apply expressions to matching columns:
```sql
SELECT COLUMNS('KOD.*')
FROM "T_SenaraiMuridALL+PraIPG"
LIMIT 10
```

---

## Quick Exploration

### FROM-First Syntax
Skip `SELECT *`:
```sql
FROM "T_SenaraiMuridALL+PraIPG"
LIMIT 10
```

### SUMMARIZE
Get table statistics (min, max, avg, nulls, etc.):
```sql
SUMMARIZE "T_SenaraiMuridALL+PraIPG"
```

Or for a query:
```sql
SUMMARIZE SELECT * FROM "T_SenaraiMuridALL+PraIPG" WHERE negeri = 'SELANGOR'
```

---

## Column Aliases

### Prefix Aliases
Write `x: value` instead of `value AS x`:
```sql
SELECT 
  total_murid: COUNT(*),
  negeri: negeri
FROM "T_SenaraiMuridALL+PraIPG"
GROUP BY negeri
```

### Lateral Column Aliases
Reuse computed columns in same SELECT:
```sql
SELECT 
  tahun + 1 AS next_year,
  next_year + 1 AS year_after
FROM "T_SenaraiMuridALL+PraIPG"
LIMIT 5
```

---

## Conditional Logic

### SWITCH Expression
Cleaner than CASE for multiple conditions:
```sql
SELECT 
  KODJANTINA,
  SWITCH KODJANTINA
    WHEN 'L' THEN 'Lelaki'
    WHEN 'P' THEN 'Perempuan'
    ELSE 'Tidak diketahui'
  END AS jantina_text
FROM "T_SenaraiMuridALL+PraIPG"
LIMIT 10
```

---

## Pivoting

### PIVOT
Long to wide:
```sql
PIVOT "T_SenaraiMuridALL+PraIPG"
ON KETERANGANALIRAN
USING COUNT(*)
GROUP BY negeri
```

### UNPIVOT
Wide to long:
```sql
UNPIVOT (
  SELECT negeri, tahun_2023, tahun_2024, tahun_2025
  FROM school_counts
)
ON tahun_2023, tahun_2024, tahun_2025
INTO NAME tahun VALUE bilangan
```

---

## Trailing Commas
DuckDB allows trailing commas (like JavaScript):
```sql
SELECT
  negeri,
  jantina,
  COUNT(*) AS total,
FROM "T_SenaraiMuridALL+PraIPG"
GROUP BY ALL
```

---

## Common Patterns for Education Data

### Student counts by state and gender
```sql
SELECT negeri, jantina, count()
FROM "T_SenaraiMuridALL+PraIPG"
GROUP BY ALL
ORDER BY negeri, jantina
```

### Teacher counts excluding internal IDs
```sql
SELECT * EXCLUDE (IDGURU, IDMURID)
FROM "T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"
LIMIT 20
```

### School summary statistics
```sql
SUMMARIZE "T_SekolahBeroperasi+PraIPG"
```

### Student distribution by aliran (with friendly alias)
```sql
SELECT 
  aliran: KETERANGANALIRAN,
  bilangan: COUNT(*)
FROM "T_SenaraiMuridALL+PraIPG"
GROUP BY ALL
ORDER BY bilangan DESC
```
