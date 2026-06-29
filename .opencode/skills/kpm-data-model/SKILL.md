---
name: kpm-data-model
description: KPM/MOE education data: table catalog, key columns, joins, code values, and how to construct queries for murid, guru, sekolah, enrolmen, B40/SES, STEM, DLP, kelas, asrama, prasekolah, T6, KV in DuckDB. Use when querying education data by negeri/PPD/sekolah, or by kaum/jantina/warganegara. Other skills may reach this for warehouse schema. Not for API mechanics or DuckDB SQL syntax.
version: 1.0.0
---

> **Updating this skill:** This skill lives at `https://github.com/zurggie/spdmp-skills.git`. Agent should look up the repo and download the latest version when updating.

# MOE Education Data — Data Model Reference

Domain pointer reference for MOE/KPM education data in DuckDB. Use this skill to find the **right base table, the right join keys, the right filter, and the right code values** for a request about Malaysian school data. No rigid SQL recipes — construct queries from the pointers here.

## Core Rules

- Use **read-only** access for analysis.
- Use **double quotes** for table and column names in DuckDB SQL.
- Start from the **base table** for the entity being counted or listed:
  - Murid: `"T_SenaraiMuridALL+PraIPG"`
  - Guru: `"T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"`
  - Sekolah: `"T_SekolahBeroperasi+PraIPG"`
- Count people and schools with **distinct IDs**:
  - Murid: `COUNT(DISTINCT "IDMURID")`
  - Guru: `COUNT(DISTINCT "KPUTAMA")`
  - Sekolah: `COUNT(DISTINCT "KODSEKOLAH")`
  - Kelas: `COUNT(DISTINCT "IDKELAS")`
- Join `"T_SekolahBeroperasi+PraIPG"` on `"KODSEKOLAH"` when output needs `NEGERI`, `PPD`, `Peringkat`, `JENISSEKOLAH`, `NAMASEKOLAH`, DUN, Parlimen, or active-school context.
- **Exclude PraIPG** (preschool/IPG programs) in school/student queries unless told otherwise:
  - `"T_SekolahBeroperasi+PraIPG".KODJENISSEKOLAH != '408'` (filters out 'PROGRAM PRA SEKOLAH DI IPG')
  - or `"T_SekolahBeroperasi+PraIPG".Peringkat IN ('RENDAH', 'MENENGAH')`
  - or for older datasets: `"Sp_Thp_KodTingkatanTahun".Peringkat_PPM <> 'prasekolah'`
- **Always join with the base tables** to get active data: `"T_SekolahBeroperasi+PraIPG"`, `"T_SenaraiMuridALL+PraIPG"`, `"T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"`. Other tables like `TSSEKOLAH`, `TGSTAF`, `T_DataSemuaMurid` may include inactive or historical records.
- **Run `DESCRIBE` first** to verify column names exist in the current dataset — they vary between monthly snapshots.
- **Inspect schema first** for DLP, asrama, elektrik/internet, pedalaman fields — column and table names vary across snapshots.
- **Default ethnic grouping** uses `LTKAUM.KODPENGELASAN` (Bumiputera: A/M/X/Y, India: I, Cina: C, Lain: L).
- **SES thresholds** (from `VIEW_DATA_MURID_EPRD.PENDAPATAN_PENJAGA1/2`): 0 = Tiada Maklumat, <5250 = B40, <11820 = M40, >=11820 = T20.

## How to Construct a Query

The skill is pointer-based, not recipe-based. Build queries from these building blocks:

1. **Pick the base table** for the entity (see Quick Reference below).
2. **Add the school join** to `T_SekolahBeroperasi+PraIPG` if you need `NEGERI` / `PPD` / `JENISSEKOLAH` / `Peringkat`.
3. **Add lookup joins** for human-readable labels — see [lookups.md](references/lookups.md) for the right `LT_` / `TK_` table and its join column.
4. **Apply the PraIPG filter** unless the question is specifically about prasekolah/IPG.
5. **Use domain code values** for filters — e.g. kaum `KODPENGELASAN`, grade `KODTINGKATANTAHUN`, school type `KODJENISSEKOLAH`. Code lists live in the domain reference files.
6. **Aggregate with `COUNT(DISTINCT <id>)`** for people/schools/classes, `SUM(b."Bil")` for room counts.
7. **Always `DESCRIBE` / `information_schema.columns` first** — column names drift between monthly snapshots. The pointers below tell you the *likely* column, not a guarantee.
8. **For multi-snapshot queries** (e.g. 2022 vs 2024 vs 2026 enrolment), use `ATTACH` to load multiple DBs and `UNION ALL` across years. Always `SHOW TABLES` first — pre-2022 snapshots do not include the `+PraIPG` suffix.

## Quick Reference: Base Tables

| Entity | Base Table | Key Columns |
|--------|-----------|-------------|
| **Students** | `"T_SenaraiMuridALL+PraIPG"` | `IDMURID`, `KODSEKOLAH`, `KODKAUM`, `KODJANTINA`, `KODTINGKATANTAHUN`, `IDKELAS` |
| **Teachers** | `"T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"` | `KPUTAMA`, `KODSEKOLAH`, `JANTINA`, `Kaum`, `KODJAWATAN`, `KETUA` |
| **Schools** | `"T_SekolahBeroperasi+PraIPG"` | `KODSEKOLAH`, `NAMASEKOLAH`, `KODJENISSEKOLAH`, `NEGERI`, `PPD`, `Peringkat`, `JENISSEKOLAH`, `SESI` |

---

## Domain Reference

| Domain | What it answers | Reference |
|--------|----------------|-----------|
| **Students** | Enrolment, demographics, citizenship, OKU, SES, prasekolah, T6, KV, PK, DLP | [students.md](references/students.md) |
| **Teachers** | Staff count, qualifications, opsyen, positions, PGB, GPK, lantikan, STEM | [teachers.md](references/teachers.md) |
| **Schools** | School list, type, location, geography chain, asrama, elektrik, pedalaman | [schools.md](references/schools.md) |
| **Classes** | Class lists, stream, grade level, capacity, kelas lebih | [classes.md](references/classes.md) |
| **Subjects** | Subject enrolment, STEM categorisation, assessment, dropout, DLP detection | [subjects.md](references/subjects.md) |
| **Infrastructure** | Classrooms, sanitation, water, electricity, assets, land, buildings | [infrastructure.md](references/infrastructure.md) |
| **Lookups** | Decode codes to labels (ethnicity, school type, opsyen, geography, etc.) | [lookups.md](references/lookups.md) |

---

## Question → Reference Pointer

Match the question to the domain file. The file gives table catalog, join keys, and code values.

| Question keywords (BM / EN) | See |
|------------------------------|-----|
| Bilangan / enrolmen murid, kaum, jantina, warganegara, SES, B40, India, prasekolah, T6, KV, pendidikan khas, DLP, senarai murid | [students.md](references/students.md) |
| Bilangan / opsyen / kelayakan / PGB / GPK / lantikan / guru prasekolah / STEM guru / guru khas | [teachers.md](references/teachers.md) |
| Senarai sekolah, jenis sekolah, lokasi, asrama, elektrik, pedalaman, DUN, Parlimen, PPD, daerah | [schools.md](references/schools.md) |
| Bilangan kelas, kelas lebih kapasiti, senarai kelas | [classes.md](references/classes.md) |
| Subjek, STEM, mata pelajaran, DLP detection | [subjects.md](references/subjects.md) |
| Bilik darjah, tandas, air, elektrik, bangunan, tanah, inventori | [infrastructure.md](references/infrastructure.md) |
| Decode `KODKAUM` / `KODJENISSEKOLAH` / `KODOPSYEN` / `KODNEGERI` / etc. to labels | [lookups.md](references/lookups.md) |

---

## Common Placeholders

`{SNAPSHOT_DB}`, `{NEGERI_LIST}`, `{KODSEKOLAH_LIST}`, `{KODTING_LIST}`, `{SUBJECT_CODES}`, `{SUBJECT_NAMES}`, `{COUNTRY_LIST}`, `{AS_OF_DATE}`, `{CLASS_SIZE_THRESHOLD}`, `{OPSYEN_CODES}`.
