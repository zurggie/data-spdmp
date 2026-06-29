# Teachers & Staff (Guru / Staf / AKP)

Teacher and staff data covering position, qualifications, specialisation (opsyen), subjects taught, and non-teaching staff.

> **Always `DESCRIBE` first.** Column names drift between monthly snapshots — verify before writing any query.

---

## `"T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"`

**Purpose**: Primary teacher table. Lists all teachers filling positions (including Pra and IPG). Use for teacher counts, distribution by school, demographics, and position analysis.

**Key columns:**

| Column | Type | Description |
|--------|------|-------------|
| `KODSEKOLAH` | VARCHAR | School code — joins to `T_SekolahBeroperasi+PraIPG` |
| `KPUTAMA` | VARCHAR | Primary teacher ID — joins to `TGSTAF` |
| `Umur` | BIGINT | Teacher age |
| `JANTINA` | VARCHAR | Gender |
| `Kaum` | VARCHAR | Ethnicity |
| `TARAFJAWATAN` | VARCHAR | Position grade (Pensiun Cemerlang, DG44, etc.) |
| `SkimKhidmat` | VARCHAR | Service scheme |
| `KODGREDGAJI` | VARCHAR | Salary grade |
| `KEDUDUKAN` | VARCHAR | Position type (Guru, Guru Penolong, etc.) |
| `JulatUmur` | VARCHAR | Age range bucket |
| `TahunBersara` | BIGINT | Retirement year |
| `KODJAWATAN` | VARCHAR | Position code — joins to `LT_JAWATANGURU` |
| `KODJAWATANLUARNORMA` | VARCHAR | External-norm position code (e.g. PK roles, GPKI) |
| `TARIKHLAHIR` | TIMESTAMP | Date of birth |
| `KETUA` | VARCHAR | Head/leadership flag — `'PGB'` for Pengetua/Guru Besar |
| `GURUOPSYENPRA` | VARCHAR | Preschool opsyen flag |
| `JENIS_CACAT` | VARCHAR | Disability type |

**When to use**: Teacher counts per school, teacher demographics, position distribution, age/retirement analysis.

**Joins with:**
- `"T_SekolahBeroperasi+PraIPG"` ON `KODSEKOLAH`
- `"TGSTAF"` ON `KPUTAMA`
- `"TGIKHTISAS"` ON `KPUTAMA`
- `"TGMATAPELAJARAN"` ON `KPUTAMA`

---

## `"TGSTAF"`

**Purpose**: Staff master data. Detailed personal information for all staff including appointment dates, service history, salary, and contact info.

**Key columns (89 columns):** `KPUTAMA`, `KODSEKOLAH`, `JENISSTAF`, `NAMA`, `JANTINA`, `KODKAUM`, `KODAGAMA`, `KODWARGANEGARA`, `TARIKHLAHIR`, `TARIKHLANTIKANPERTAMA`, `TARIKHLANTIKANSEMASA`, `KODJAWATAN`, `KODGREDGAJI`, `GAJIHAKIKI`, `OPSYENDOMINAN`, `GURUDLP`, `KODLABELGURU`

**When to use**: When you need detailed personal data, appointment dates, salary info, or service history for teachers/staff.

**Joins with:**
- `"T_EPGO_SenaraiGuruIsiPjawatan+PraIPG"` ON `KPUTAMA`
- `"LTKAUM"` ON `KODKAUM`

---

## `"TGIKHTISAS"`

**Purpose**: Teacher specialisation/qualification (Ikhtisas / Opsyen). Lists each teacher's academic specialisations with up to 4 opsyen codes.

**Key columns:**

| Column | Type | Description |
|--------|------|-------------|
| `KPUTAMA` | VARCHAR | Teacher ID — joins to teacher tables |
| `KODIKHTISAS` | VARCHAR | Qualification code |
| `NAMAINSTITUSI` | VARCHAR | Institution name |
| `TAHUN` | VARCHAR | Year of qualification |
| `KODOPSYEN1` | VARCHAR | Primary specialisation — joins to `LT_OPSYEN` |
| `KODOPSYEN2` | VARCHAR | Secondary specialisation |
| `KODOPSYEN3` | VARCHAR | Tertiary specialisation |
| `KODOPSYEN4` | VARCHAR | Fourth specialisation |

**When to use**: When analysing teacher qualifications and subject-matter expertise. Find which teachers are qualified in specific fields (STEM, Bahasa Cina, etc.).

---

## `"TGMATAPELAJARAN"`

**Purpose**: Teacher-subject mapping. Lists the subjects each teacher actually teaches and weekly teaching hours.

**Key columns:** `KPUTAMA`, `KODSUBJEK`, `KODKELAS`, `BILWAKTUMENGAJAR`, `STATUSDLP`

**When to use**: To find which teachers teach which subjects, or to calculate subject-specific teacher distribution.

---

## `"T_EPGO_SenaraiAKP"`

**Purpose**: AKP (Anggota Khidmat Pelaksana) — non-teaching support staff such as clerks, assistants, and operators.

**Key columns:** `KODSEKOLAH`, `KPUTAMA`, `JANTINA`, `Kaum`, `TARAFJAWATAN`, `KODJAWATAN`, `JAWATAN`, `TARIKHLAHIR`

**When to use**: When analysing non-teaching staff distribution, separate from teaching staff counts.

---

## Position / Role Code Values

These codes may appear in `KODJAWATAN` or `KODJAWATANLUARNORMA`. Always check both columns with a `UNION` when filtering by role.

### Head / Leadership

| Role | Filter |
|---|---|
| **PGB** (Pengetua / Guru Besar) | `KETUA = 'PGB'` |
| **GPK** (Penolong Kanan) | `KODJAWATAN IN ('PK1','PK2','PK3','PKP')` — also check `KODJAWATANLUARNORMA` |
| **PK1 / PK Pentadbiran** | `kod = 'PK1'` |
| **PK2 / PK HEM** | `kod = 'PK2'` |
| **PK3 / PK Kokurikulum** | `kod = 'PK3'` |
| **PKP / PK Petang** | `kod = 'PKP'` |

### Prasekolah

| Filter | Notes |
|---|---|
| `KODJAWATAN = 'GPRA'` | Guru prasekolah explicit code |
| `COALESCE(GURUOPSYENPRA, '') <> ''` | Has preschool opsyen flag |

### Pendidikan Khas (Special Education)

Codes can appear in `KODJAWATAN` or `KODJAWATANLUARNORMA`:

| Code | Role |
|---|---|
| `GPKIB` | Guru Pendidikan Khas Integrasi (Bahasa) |
| `GPKID` | Guru Pendidikan Khas Integrasi (Dasar) |
| `GPKIL` | Guru Pendidikan Khas Integrasi (Lain) |
| `PKPK` | Penolong Kanan Pendidikan Khas |
| `GPKI` | Guru Pendidikan Khas Integrasi (generic) |

### Lantikan (Appointment Type)

Source: `TGSTAF.KODTARAFJAWATAN` or teacher base `TARAFJAWATAN`. Field name varies by snapshot — `DESCRIBE` to confirm.

| KODTARAFJAWATAN | Kategori |
|---|---|
| `TPC`, `TTP`, `CBA` | Tetap |
| `KOI`, `COS`, `KO4`, `KONS`, `MOU` | Kontrak |

### Position Grade (KODGREDGAJI examples)

| Code | Meaning |
|---|---|
| `GL17` | Guru Lepasan 17 |
| `DG41`, `DG44`, `DG48`, `DG54` | Guru DG grades |

### STEM (Menengah Atas)

| Filter | Codes |
|---|---|
| **Class codes (Menengah Atas)** | `1006`, `1007`, `1026`, `1027`, `1035` |
| **Subject codes (default STEM)** | `E101`, `E110`, `E202`, `A804`, `E204`, `E205`, `E207`, `E321`, `E322`, `E324`, `E605`, `E606`, `E607`, `E609` |

---

## Pointer: Guru Mengajar Mata Pelajaran Tertentu

Use when user asks:
- `BIL GURU MP MATEMATIK`
- `GURU MENGAJAR BAHASA MELAYU SEKOLAH RENDAH`
- `BIL GURU BAHASA ARAB SMKA SABK`
- `GURU SAINS BIOLOGI KIMIA FIZIK`

**Construction**:
- Join guru base → `TGMATAPELAJARAN` (on `KPUTAMA`) → `LT_MATAPELAJARAN` (on `KODSUBJEK = KODMATAPELAJARAN`).
- Filter `KODSUBJEK IN (...)` using codes from `LT_MATAPELAJARAN` — always `DESCRIBE` to confirm subject codes.
- `COUNT(DISTINCT KPUTAMA)`.
- Group by `NEGERI`, `PPD`, `JENISSEKOLAH`, subject.

---

## Pointer: Guru Opsyen vs Bukan Opsyen

Use when user asks:
- `BILANGAN GURU OPSYEN DAN BUKAN OPSYEN BAHASA MELAYU`
- `GURU KIMIA FIZIK OPSYEN`
- `BILANGAN GURU OPSYEN MATEMATIK MENGIKUT LOKASI`

**Construction**:
- Build two CTEs: one for teachers teaching the subject (via `TGMATAPELAJARAN`), one for teachers with the opsyen (via `TGIKHTISAS` checking `KODOPSYEN1..4`).
- `LEFT JOIN` them on `KPUTAMA`; classify as `Opsyen` / `Bukan Opsyen` based on the second CTE matching.

---

## Pointer: Bilangan Guru Mengikut Negeri / PPD / Jantina

Use when user asks:
- `Bilangan Guru Mengikut Negeri`
- `BILANGAN_GURU_MENENGAH_RENDAH_MENGIKUT_JANTINA_SEKOLAH_PPD_NEGERI`
- `BILANGAN GURU MENGIKUT JENIS SEKOLAH JAWATAN`

**Construction**:
- Group guru base by `NEGERI`, `PPD`, `JENISSEKOLAH`, `Peringkat`, `JANTINA` (after school join).
- `COUNT(DISTINCT KPUTAMA)`.

---

## Pointer: Senarai Guru (Teacher List)

Use when user asks for a list:
- `SENARAI GURU PPLD`
- `SENARAI GURU MATA PELAJARAN`
- `SENARAI GURU STEM`

**Construction**:
- Join guru base → `TGSTAF` (on `KPUTAMA`) for `NAMA` and personal details.
- Filter by `NEGERI` or other criteria; `ORDER BY NEGERI, PPD, KODSEKOLAH, NAMA`.
- Export if over 50 rows.

---

## Pointer: PGB / GPK

- **PGB**: filter `KETUA = 'PGB'` on the guru base table.
- **GPK**: filter `KODJAWATAN IN ('PK1','PK2','PK3','PKP')` — also union over `KODJAWATANLUARNORMA` since GPK roles may sit there.
- Group by role (`CASE` mapping) and school geography.

---

## Pointer: Guru Lantikan Tetap / Kontrak

Use when user asks:
- `BIL GURU TETAP KONTRAK MENGIKUT TAHAP DAN JANTINA`
- `guru lantikan tetap` / `guru kontrak`

**Construction**:
- Join guru base → `TGSTAF` for `KODTARAFJAWATAN` (field name varies — `DESCRIBE` first).
- Classify per the Lantikan table above.
- Also derive tahap: `KODJAWATAN='GPRA'` or non-empty `GURUOPSYENPRA` → `'Prasekolah'`; else use `Peringkat` from school join.
- `COUNT(DISTINCT KPUTAMA)` grouped by tahap, lantikan, jantina.

---

## Pointer: Guru Prasekolah

Filter: `KODJAWATAN = 'GPRA' OR COALESCE(GURUOPSYENPRA, '') <> ''`. Group by `NEGERI`, `PPD`.

---

## Pointer: Guru Pendidikan Khas

See Pendidikan Khas codes above. Use `UNION` over `KODJAWATAN` and `KODJAWATANLUARNORMA`. Group by `NEGERI`, `PPD`, school, jantina.

---

## Pointer: Guru STEM Menengah Atas

See STEM class/subject codes above. Build a CTE with the STEM subject list, join `TGMATAPELAJARAN` filtered by both class codes and subject codes, dedupe by `KPUTAMA`, count per school geography.
