# Query Construction Pointers

This file is now a slim index. The Question→Reference pointer table lives in [SKILL.md](../SKILL.md). This file keeps only the cross-domain pointers and the multi-snapshot ATTACH pattern.

> **Always DESCRIBE first.** Run `DESCRIBE "TableName"` (or query `information_schema.columns`) before writing any query. Column names drift between monthly snapshots.

---

## Quick Lookup

| Question | See |
|----------|-----|
| Berapa murid mengikut negeri/kaum/jantina? | [SKILL.md → students.md](students.md) |
| Berapa jumlah murid, guru, sekolah, dan bilik darjah? | [SKILL.md → 4 base tables](../SKILL.md#how-to-construct-a-query) |
| Berapa sekolah/murid mengikut jenis sekolah? | [SKILL.md → schools.md](schools.md) |
| Jadual silang: kaum vs jantina vs negeri (PIVOT) | [SKILL.md → students.md](students.md) (use DuckDB PIVOT) |
| Taburan mengikut daerah/PPD/negeri/parlimen? | [SKILL.md → schools.md](schools.md) |
| Guru mengikut opsyen dan kelayakan? | [SKILL.md → teachers.md](teachers.md) |
| Status kemudahan asas sekolah? | [SKILL.md → infrastructure.md](infrastructure.md) |
| Murid dan kemudahan asrama? | [SKILL.md → schools.md](schools.md#asrama-hostel-schema-inspection) |
| Murid mengambil subjek bahasa tertentu? | [SKILL.md → subjects.md](subjects.md) |
| Perbandingan antara dua bulan/data set? | [§ ATTACH Multi-Year below](#attach-multi-year-time-series) |
| Restructure data untuk DOSM/quickfacts? | Use DuckDB `UNPIVOT` on your aggregated result |

---

## Pointers to Domain Files

These patterns live in their domain files. Use the pointers below instead of duplicating here.

| Topic | Home |
|---|---|
| Enrolment by negeri/PPD/sekolah/tahun-tingkatan | [students.md](students.md) |
| Enrolment by peringkat & jenis sekolah | [students.md](students.md) |
| Warganegara / Bukan Warganegara | [students.md](students.md) |
| Murid India / kaum | [students.md](students.md) (use `KODPENGELASAN='I'`) |
| SES / B40 / M40 / T20 | [students.md](students.md) (thresholds in SKILL.md Core Rules) |
| Prasekolah, T6, KV, Pendidikan Khas enrolment | [students.md](students.md) (filter by `KODTINGKATANTAHUN`) |
| Senarai murid (list, not count) | [students.md](students.md) |
| Murid by age | [students.md](students.md) (use `VIEW_DATA_MURID_EPRD.TKHLAHIR`) |
| DLP detection & template | [students.md](students.md) |
| Teacher PGB, GPK, lantikan, prasekolah, khas, STEM | [teachers.md](teachers.md) |
| Teacher opsyen vs bukan opsyen | [teachers.md](teachers.md) |
| Asrama / sumber elektrik / pedalaman | [schools.md](schools.md#asrama-hostel-schema-inspection) |
| Bilik darjah Pendidikan Khas | [infrastructure.md](infrastructure.md) |
| Bilangan kelas, kelas lebih kapasiti | [classes.md](classes.md) |
| STEM PIVOT categorisation | [subjects.md](subjects.md) |

---

## ATTACH Multi-Year Time Series

For "Bilangan sekolah, guru, murid 2022-2026" / multi-year enrolment comparison across DuckDB snapshots:

1. `ATTACH '{DB_2022}' AS y2022;` (and similarly for other years)
2. `SHOW TABLES` in each attached DB — pre-2022 snapshots do **not** include the `+PraIPG` suffix
3. Build a `WITH combined AS (... UNION ALL ...)` CTE across years, tagging each row with its year label
4. Apply the PraIPG filter per year (column names and filter values may differ)

Always keep the year label explicit. Refer to [SKILL.md § How to Construct a Query](../SKILL.md#how-to-construct-a-query) step 8.
