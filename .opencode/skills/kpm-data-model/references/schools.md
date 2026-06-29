# Schools (Sekolah)

School master data covering identification, type, location, geography, infrastructure status, and contact information.

> **Always `DESCRIBE` first.** Column names drift between monthly snapshots — verify before writing any query.

---

## `"T_SekolahBeroperasi+PraIPG"`

**Purpose**: Primary school table. Lists all **operating** schools (including Prasekolah and IPG programs). This is the main table for school-level queries and joins.

**Key columns (28):**

| Column | Type | Description |
|--------|------|-------------|
| `NEGERI` | VARCHAR | State name (direct, no code needed) |
| `PPD` | VARCHAR | PPD / District Education Office name |
| `PARLIMEN` | VARCHAR | Parliament constituency name |
| `DUN` | VARCHAR | State constituency name |
| `Peringkat` | VARCHAR | Level (RENDAH / MENENGAH) |
| `KODSEKOLAH` | VARCHAR | **Primary Key** — school code |
| `NAMASEKOLAH` | VARCHAR | School name |
| `JENISSEKOLAH` | VARCHAR | School type name (e.g., SK, SMK, SJKC) |
| `LOKASI` | VARCHAR | Location (Bandar / Luar Bandar) |
| `GRED` | VARCHAR | School grade |
| `KODJENISSEKOLAH` | VARCHAR | School type code — joins to `TKJENISSEKOLAH` |
| `Pedalaman` | VARCHAR | Interior/remote classification |
| `BANTUAN` | VARCHAR | Aid type (Kerajaan / Bantuan) |
| `SESI` | VARCHAR | School session (e.g. `'2 Sesi'` for 2-session schools) |
| `SKM` | VARCHAR | School cluster (SKM) status |
| `KOORDINATXX` | DOUBLE | Latitude |
| `KOORDINATYY` | DOUBLE | Longitude |
| `TARIKHTUBUHSEKOLAH` | TIMESTAMP | Establishment date |
| `KODTUMPANG` | VARCHAR | Host school code (if tumpang) |

**When to use**: Most school queries. Join point for students, teachers, and infrastructure tables.

**Joins with:**
- `"T_SenaraiMuridALL+PraIPG"` ON `KODSEKOLAH`
- `"T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"` ON `KODSEKOLAH`
- `"TSSEKOLAH"` ON `KODSEKOLAH` (for master data beyond operating schools)
- `"TKJENISSEKOLAH"` ON `KODJENISSEKOLAH`
- `"TKDAERAH"`, `"TKNEGERI"`, `"TKPPD"`, `"TKPARLIMEN"` (geographical lookups)

> Note: `NEGERI`, `PPD`, `PARLIMEN`, `DUN` are denormalised (stored as names, not codes). For code-based joins, use `TSSEKOLAH` instead.
>
> **Exclude PraIPG** — see [SKILL.md Core Rules](../SKILL.md) for the canonical filter patterns (`KODJENISSEKOLAH != '408'`, `Peringkat IN ('RENDAH', 'MENENGAH')`, or grade-level table for older datasets).

---

## `"TSSEKOLAH"`

**Purpose**: School master data with 94 columns — the most detailed school table. Contains all schools (including non-operating, closed, and shelved schools). Use for detailed school attributes, geographical codes, and infrastructure status.

**Key columns (94 total):** `KODSEKOLAH`, `NAMASEKOLAH`, `ALAMATLOKASISEKOLAH`, `BANDARLOKASISEKOLAH`, `POSKODLOKASISEKOLAH`, `KODNEGERILOKASISEKOLAH`, `KODSTATUSSEKOLAH`, `KODJENISSEKOLAH`, `GREDSEKOLAH`, `KODJENISBANTUAN`, `KODPERINGKAT`, `KODLOKASI`, `KODSUMBERAIR`, `KODSUMBERELEKTRIK`, `KODDAERAH`, `KODPARLIMEN`, `KODDUN`, `KODPPD`, `KODKERAJAANTEMPATAN`, `KODJENISASRAMA`, `KODSEJARAHSEKOLAH`, `KOORDINATXX`, `KOORDINATYY`, `NOTELEFON`, `EMAIL`

**When to use**: When you need geographical codes (KODDAERAH, KODPPD, KODPARLIMEN), infrastructure codes (KODSUMBERAIR, KODSUMBERELEKTRIK), or school master data beyond the operating schools list.

**Difference from `T_SekolahBeroperasi+PraIPG`:**

| Aspect | `T_SekolahBeroperasi+PraIPG` | `TSSEKOLAH` |
|--------|------------------------|-------------|
| Scope | Operating schools only | All schools (incl. closed) |
| Geography | Denormalised names | Codes (join to lookups) |
| Infrastructure | Basic | Full (water, electricity, etc.) |
| Row count | ~10,275 | ~10,529 |

---

## `"SenaraiSekolahWeb"`

**Purpose**: School list formatted for web display. Includes enrolment numbers, teacher counts, and preschool/integration info in a simplified view.

**Key columns (30):** `NEGERI`, `PPD`, `PARLIMEN`, `DUN`, `PERINGKAT`, `JENIS/LABEL`, `KODSEKOLAH`, `NAMASEKOLAH`, `LOKASI`, `GRED`, `BANTUAN`, `ENROLMEN`, `ENROLMEN KHAS`, `GURU`, `PRASEKOLAH`, `ENROLMEN PRASEKOLAH`, `INTEGRASI`, `KOORDINATXX`, `KOORDINATYY`

**When to use**: When you need a quick school overview with enrolment and teacher counts in a single table.

---

## Geographical Hierarchy

The geographical chain from finest to broadest:

```
KODDAERAH → KODPPD → KODNEGERI → KODPARLIMEN → KODDUN
(District)  (PPD)    (State)     (Parliament)  (Constituency)
```

Use `TSSEKOLAH` for code-based joins to lookup tables (`TKDAERAH`, `TKPPD`, `TKNEGERI`, `TKPARLIMEN`). Use `T_SekolahBeroperasi+PraIPG` for denormalised name-based filtering.

---

## School Type Code Values

From `TKJENISSEKOLAH` — see [lookups.md](lookups.md#tkjenissekolah) for full mapping.

| Code | Type | Level | Quickfacts |
|------|------|-------|------------|
| 101 | SK | Rendah | SK |
| 103 | SJK(C) | Rendah | SJKC |
| 104 | SJK(T) | Rendah | SJKT |
| 201 | SMK | Menengah | SMK |
| 203 | SM Teknik | Menengah | SMT |
| 206 | SM Berasrama Penuh | Menengah | SBP |
| 211 | SBJK | Menengah+Rendah | SBJK |
| 408 | Program Pra di IPG | Program | — (excluded by PraIPG filter) |

---

## Pointer: Senarai Sekolah Mengikut Jenis / Peringkat / Lokasi

Use when user asks:
- `DATA SEKOLAH`
- `SENARAI SEKOLAH BUKAN SK SMK`
- `SENARAI SEKOLAH MODEL KHAS`
- `MAKLUMAT LOKASI SEKOLAH`

**Construction**:
- Source: `T_SekolahBeroperasi+PraIPG`.
- Apply PraIPG exclusion.
- Filter by `JENISSEKOLAH` / `Peringkat` / `LOKASI` (or `KODJENISSEKOLAH` for code-based filters).
- `ORDER BY NEGERI, PPD, JENISSEKOLAH, NAMASEKOLAH`.

---

## Pointer: Bilangan Sekolah

Use when user asks:
- `BILANGAN SEKOLAH`
- `BILANGAN SEKOLAH 2022-2024`
- `BIL SEKOLAH RENDAH DAN MENENGAH`

**Construction**:
- `COUNT(DISTINCT KODSEKOLAH)` grouped by `NEGERI`, `Peringkat`, `JENISSEKOLAH`.

---

## Pointer: Sekolah 2 Sesi

Filter `SESI = '2 Sesi'`. Returns school rows directly.

---

## Pointer: Asrama (Hostel) Schema Inspection

Use when user asks:
- `SEKOLAH_BERASRAMA_RENDAH_MENENGAH`
- `SENARAI SEKOLAH BERASRAMA`
- `BILANGAN MURID ASRAMA DAN WARDEN`

**Construction**:
- Field names vary across snapshots/tables. **Always inspect first**:
  - `SELECT table_name, column_name FROM information_schema.columns WHERE upper(column_name) LIKE '%ASRAMA%' OR upper(table_name) LIKE '%ASRAMA%'`
- Likely sources: `TSSEKOLAH.KODJENISASRAMA` (joins to `TKJENISASRAMA`) or `VIEW_DATA_MURID_EPRD.ASRAMA` (per-student flag).
- After confirming fields, build the query against the right table.

---

## Pointer: Sumber Elektrik / Internet / Pedalaman

Use when user asks:
- `SENARAI SEKOLAH SUMBER ELEKTRIK`
- `sekolah_pedalaman_p3_internet_elektrik`
- `SEKOLAH LOKASI PEDALAMAN WAWASAN ASLI`

**Construction**:
- Field names differ between infrastructure extracts. **Always inspect first**:
  - `SELECT table_name, column_name FROM information_schema.columns WHERE upper(column_name) LIKE '%ELEKTRIK%' OR upper(column_name) LIKE '%INTERNET%' OR upper(column_name) LIKE '%PEDALAMAN%' OR upper(column_name) LIKE '%LOKASI%'`
- Likely columns: `TSSEKOLAH.KODSUMBERELEKTRIK`, `TSSEKOLAH.KODSUMBERAIR`, `T_SekolahBeroperasi+PraIPG.Pedalaman` / `LOKASI`.
- Apply PraIPG exclusion and group by `NEGERI`, `PPD`, `KODSEKOLAH`.
