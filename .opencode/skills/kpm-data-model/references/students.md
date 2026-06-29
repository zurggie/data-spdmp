# Students (Murid)

Student/pupil data covering enrolment, demographics, citizenship, OKU status, and guardian information.

> **Always `DESCRIBE` first.** Column names drift between monthly snapshots — verify before writing any query.

---

## `"T_SenaraiMuridALL+PraIPG"`

**Purpose**: Primary student table for 2022+ datasets. Contains all students including Prasekolah (preschool) and IPG students. This is the main table for student counts, demographics, and enrolment analysis.

**Key columns:**

| Column | Type | Description |
|--------|------|-------------|
| `KODSEKOLAH` | VARCHAR | School code — joins to `T_SekolahBeroperasi+PraIPG` |
| `IDMURID` | VARCHAR | Unique student ID |
| `KODJANTINA` | VARCHAR | Gender code — joins to `LTJANTINA` |
| `KODKAUM` | VARCHAR | Ethnicity code — joins to `LTKAUM` |
| `KETERANGANSTATUSWARGANEGARA` | VARCHAR | Citizenship status (Warganegara / Bukan Warganegara) |
| `KETERANGANNEGARA` | VARCHAR | Country of citizenship |
| `KODTINGKATANTAHUN` | VARCHAR | Grade/form level — joins to `LTTINGKATANTAHUN` |
| `KETERANGANALIRAN` | VARCHAR | Stream (Sains/Sastera/Vokasional/etc.) |
| `KETERANGANBIDANG` | VARCHAR | Field of study |
| `IDKELAS` | DOUBLE | Class ID — joins to `VIEW_DATA_KELAS_EPRD` |
| `NAMAKELAS` | VARCHAR | Class name |
| `STATUSOKU` | INTEGER | OKU flag (1 = OKU, 0 = not) |
| `JENISOKU` | VARCHAR | OKU disability type |

**When to use**: Most student queries — enrolment counts, demographic breakdowns, per-school student lists.

**Joins with:**
- `"T_SekolahBeroperasi+PraIPG"` ON `KODSEKOLAH`
- `"LTKAUM"` ON `KODKAUM`
- `"LTJANTINA"` ON `KODJANTINA`
- `"LTTINGKATANTAHUN"` ON `KODTINGKATANTAHUN`
- `"VIEW_DATA_KELAS_EPRD"` ON `IDKELAS`

**Historical note**: For datasets before 2022, the table `"T_SenaraiMuridALL"` (without `+PraIPG`) was used and excluded preschool and IPG students. If querying older datasets (2018–2021), use this variant instead.

---

## `"VIEW_DATA_MURID_EPRD"`

**Purpose**: EPRD student view with richer personal data including guardian information, OKU registration, hostel status, and income details. Contains data for recent datasets.

**Key columns:** `IDMURID`, `NOKP`, `TKHLAHIR`, `IDKELAS`, `KODJANTINA`, `KODKAUM`, `KODSTATUSKEWARGANEGARAAN`, `KODAGAMA`, `PROGRAM`, `YATIM`, `ASRAMA`, `PENDAPATAN_PENJAGA1`

**When to use**: When you need guardian income, orphan status, hostel residency, or detailed OKU info. Join to `VIEW_DATA_KELAS_EPRD` for class context.

**Joins with:**
- `"VIEW_DATA_KELAS_EPRD"` ON `IDKELAS`
- Student base tables ON `IDMURID`

---

## `"T_DataSemuaMurid"`

**Purpose**: Comprehensive student view joining student personal data with school name, class info, guardian details, and SES (socio-economic status) information. The widest student table with 49 columns.

**Key columns:** `IDMURID`, `NAMA`, `NOKP`, `KAUM`, `AGAMA`, `KODJANTINA`, `STATUSWARGANEGARA`, `JENIS_OKU`, `IDKELAS`, `KODTINGKATANTAHUN`, `NAMAKELAS`, `KODSEKOLAH`, `NAMASEKOLAH`, `MURIDYATIM`, `PENDAPATANSEBULANPENJAGA1`, `SES`, `SESDETAIL`

**When to use**: When you need student names, guardian details, SES class, or a wide join of student + school + guardian data in one table.

---

## Demographic Code Values

### Ethnicity / Kaum grouping (Pengelasan)

Most ethnicity queries use `KODPENGELASAN` from `"LTKAUM"` for grouped analysis, not individual `KODKAUM` codes:

| KODPENGELASAN | Meaning | Bumiputera? |
|:---:|---|:---:|
| `M` | MELAYU | Yes |
| `C` | CINA | No |
| `I` | INDIA | No |
| `A` | ORANG ASAL | Yes |
| `X` | BUMIPUTERA SABAH | Yes |
| `Y` | BUMIPUTERA SARAWAK | Yes |
| `L` | LAIN-LAIN | No |
| NULL | TIADA MAKLUMAT (group as `L`) | No |

**Bumiputera test**: `LOWER(KODPENGELASAN) IN ('a', 'm', 'x', 'y')`.

**India filter**: `KODPENGELASAN = 'I'` (not a specific `KODKAUM`).

### Citizenship

- Status field varies by snapshot — try `KETERANGANSTATUSWARGANEGARA` on the murid base table or `WARGANEGARA` on `VIEW_DATA_MURID_EPRD`. Always `DESCRIBE` to confirm.
- "Bukan warganegara" filter: `upper(WARGANEGARA) NOT IN ('MALAYSIA', 'WARGANEGARA')` (or equivalent on the snapshot's status column).

### SES / B40 / M40 / T20

SES is based on total guardian income from `VIEW_DATA_MURID_EPRD`:

| Total income | Category |
|---|---|
| 0 (or NULL) | Tiada Maklumat |
| < 5250 | B40 |
| 5250 ≤ x < 11820 | M40 |
| ≥ 11820 | T20 |

Use `COALESCE(..., 0)` to treat NULL income as `Tiada Maklumat`.

---

## Grade / Form Level Codes (`KODTINGKATANTAHUN`)

Code lists for filtering by student level. Values vary by snapshot — `DESCRIBE "LTTINGKATANTAHUN"` to verify.

| Filter | Codes |
|---|---|
| **Prasekolah** | `PRA BIASA`, `PRA INTEGRASI` |
| **Tingkatan 6 / Kolej T6** | `T6S1`, `T6S2`, `T6S3`, `IBS2`, `IBS4`, `PRAU1`, `PRAU3` |
| **Kolej Vokasional / SVM / DVM** | `S1SVM`, `S2DVM`, `S3SVM`, `S4SVM`, `S3DVM`, `S5DVM` (also `JENISSEKOLAH = 'KV'`) |
| **Pendidikan Khas** | `KHAS`, `KHAM` |

---

## Pointer: Warganegara Queries

Use when user asks:
- `BILANGAN MURID WARGANEGARA AUSTRALIA`
- `ENROLMEN MURID WARGANEGARA INDIA`
- `BILANGAN MURID WARGANEGARA INDONESIA, LAOS, MYANMAR`
- `ENROLMEN_MURID_BUKAN_WARGANEGARA`
- `MURID BUKAN WARGANEGARA MENGIKUT NEGERI`

**Construction**:
- Filter field varies — check the snapshot's citizenship column (likely `VIEW_DATA_MURID_EPRD.WARGANEGARA` or base table's `KETERANGANSTATUSWARGANEGARA`).
- `COUNT(DISTINCT IDMURID)` after joining to base table.
- Group by `NEGERI`, `PPD`, `JENISSEKOLAH`, `KODTINGKATANTAHUN` as needed.
- Apply PraIPG exclusion.

---

## Pointer: Kaum / India Queries

Use when user asks:
- `MURID INDIA MENGIKUT PERINGKAT`
- `MURID KODPENGELASAN I`
- `ENROLMEN MURID KAUM INDIA`
- Any kaum / ethnic breakdown

**Construction**:
- Filter on `LTKAUM.KODPENGELASAN` (not a specific `KODKAUM`).
- For "Bumiputera vs Bukan" — use the Bumiputera test above.
- For full breakdown — `CASE` over `KODPENGELASAN` to map M/C/I/A/X/Y/L to display labels.

---

## Pointer: SES / B40 Queries

Use when user asks:
- `BIL_MURID_SES_NEGERI`
- `SEKOLAH_MENENGAH_SES`
- `B40_SEKOLAH_MENGIKUT_NEGERI`
- `BILANGAN MURID B40 MENGIKUT NEGERI DAN SEKOLAH`

**Construction**:
- Source income from `VIEW_DATA_MURID_EPRD.PENDAPATAN_PENJAGA1` (and `PENDAPATAN_PENJAGA2` if available).
- Classify per the threshold table above; `COALESCE` to 0 for NULLs.
- `COUNT(DISTINCT IDMURID)` grouped by SES class.
- Join to `T_SekolahBeroperasi+PraIPG` for school context.
- Alternative: `T_DataSemuaMurid` already has `SES` and `SESDETAIL` columns pre-computed.

---

## Pointer: Murid by Age

Use when user asks:
- `ENROLMEN_BELIA_15_30_KPM`
- `ENROLMEN MURID MENGIKUT UMUR`
- `PROJECTION WORKSHOP ENROLMENT PUPILS BY AGE`

**Construction**:
- `TKHLAHIR` is in `VIEW_DATA_MURID_EPRD`.
- Compute age as `YEAR(as_of_date) - YEAR(TKHLAHIR)` (or `date_diff('year', TKHLAHIR, as_of_date)`).
- Filter `TKHLAHIR IS NOT NULL`.
- Join to base student table on `IDMURID` for school context.

---

## Pointer: Prasekolah, T6, KV, Pendidikan Khas

Use the code lists in the "Grade / Form Level Codes" section above. Filter `T_SenaraiMuridALL+PraIPG.KODTINGKATANTAHUN IN (...)` and join to `T_SekolahBeroperasi+PraIPG` for school context.

For KV, the alternative filter is `T_SekolahBeroperasi+PraIPG.JENISSEKOLAH = 'KV'`.

---

## Pointer: DLP (Dual Language Programme) Students

Use when user asks:
- `MURID DLP`
- `SENARAI MURID DLP TINGKATAN 5`
- `DLP SARAWAK GURU MURID SEKOLAH`

**Construction rules**:
- DLP implementation varies by snapshot. **Always `DESCRIBE` first** — search `information_schema.columns` for `LIKE '%DLP%'` to find the source table and column.
- Possible sources: a dedicated DLP table, a flag column on the student table, or `LT_MATAPELAJARAN.MP_DLP` for subject-level DLP.
- After confirming the source, build a `WITH dlp_murid AS (SELECT DISTINCT IDMURID, KODSEKOLAH, KODTINGKATANTAHUN FROM ... WHERE <DLP filter>)` CTE, then `COUNT(DISTINCT IDMURID)` grouped by `NEGERI`, `PPD`, `KODSEKOLAH`, `NAMASEKOLAH`, `KODTINGKATANTAHUN`.
- Also check `TGMATAPELAJARAN.STATUSDLP` for teacher-side DLP.

---

## Pointer: Senarai Murid (Student List, Not Count)

Use when user asks for a list of pupils, not only counts:
- `SENARAI NAMA MURID KAUM INDIA B40 SEKOLAH KPM`
- `SENARAI MURID DLP TINGKATAN 5`
- `SENARAI MURID D1 LAHIR JANUARI 2020`

**Construction rules**:
- Check total rows first (`SELECT COUNT(*)`).
- Export if over 50 rows.
- Include only fields the user asked for — don't dump the whole row.
- Use `T_SenaraiMuridALL+PraIPG` for ID/code columns; join `T_DataSemuaMurid` (or `VIEW_DATA_MURID_EPRD`) for `NAMA`, `TKHLAHIR`, `NOKP`, etc.
- `ORDER BY NEGERI, PPD, KODSEKOLAH, KODTINGKATANTAHUN, IDMURID` for consistent output.
