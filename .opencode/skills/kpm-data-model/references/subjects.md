# Subjects (Mata Pelajaran) & Academics

Subject enrolment, STEM categorisation, classroom-based assessment, and dropout data.

> **Always `DESCRIBE` first.** Column names drift between monthly snapshots — verify before writing any query.

---

## Subject Tables

### `"T_SubjekAmbilMurid"`

**Purpose**: Student-subject enrolment. Records which subjects each student is taking.

**Columns:**
| Column | Type | Description |
|--------|------|-------------|
| `ID_INDIVIDU` | VARCHAR | Student ID |
| `ID_MP` | DOUBLE | Subject ID — joins to `LT_MATAPELAJARAN` (via code mapping) or `LTI_SubjekAmbilMurid` |

**Joins with:**
- `"LT_MATAPELAJARAN"` ON `ID_MP` (via code mapping)
- Student tables via `ID_INDIVIDU = IDMURID`

---

### `"LT_MATAPELAJARAN"`

**Purpose**: Subject lookup table. Maps subject codes to subject names.

**Key columns:** `KODMATAPELAJARAN`, `MATAPELAJARAN`, `PERINGKAT`, `KOD_KUM_MATAPELAJARAN`, `MP_DLP`

**When to use**: Decode subject codes to human-readable names. Filter by education level (`PERINGKAT`).

---

### `"LTI_SubjekAmbilMurid"`

**Purpose**: Subject name lookup table for `T_SubjekAmbilMurid`. Maps `ID_MP` to human-readable subject names.

| Column | Type | Description |
|--------|------|-------------|
| `ID_MP` | — | Subject ID — joins to `T_SubjekAmbilMurid.ID_MP` |
| `MP` | VARCHAR | Subject name (Mata Pelajaran) |

**Joins with:**
- `"T_SubjekAmbilMurid"` ON `ID_MP`

> Use `LTI_SubjekAmbilMurid` (not `LT_MATAPELAJARAN`) when joining with `T_SubjekAmbilMurid`, because the join column is `ID_MP` not `KODMATAPELAJARAN`.

---

### `"TGMATAPELAJARAN"`

**Purpose**: Teacher-subject mapping. Records which teachers teach which subjects and their weekly teaching hours.

**Key columns:** `KPUTAMA`, `KODSUBJEK`, `BILWAKTUMENGAJAR`, `STATUSDLP`

See [teachers.md](teachers.md) for details.

---

## STEM Categorisation Pattern

A common query pattern categorises students into STEM streams based on their subject combinations using DuckDB's PIVOT.

**Construction outline**:
1. `LT_MATAPELAJARAN` carries a `KATEGORI_STEM` column with one of: `SAINS TULEN`, `MATEMATIK TAMBAHAN`, `SAINS GUNAAN DAN TEKNOLOGI`, `VOKASIONAL` (or NULL).
2. Join `T_SubjekAmbilMurid` → `LT_MATAPELAJARAN` on `ID_MP`, filter `KATEGORI_STEM IS NOT NULL`.
3. PIVOT on `KATEGORI_STEM` with `USING count(ID_INDIVIDU)`, grouped by `KODSEKOLAH`, `ID_INDIVIDU`, `KOD_THN_TING` (or `KODTINGKATANTAHUN`).
4. Map counts to categories:

| Category | Criteria |
|----------|----------|
| **STEM A** | SAINS TULEN (3) + MATEMATIK TAMBAHAN (1) |
| **STEM B** | SAINS TULEN (2) + MATEMATIK TAMBAHAN (1) + SAINS GUNAAN DAN TEKNOLOGI (1) |
| **STEM C1** | SAINS GUNAAN DAN TEKNOLOGI (2) |
| **STEM C2** | VOKASIONAL (1) |
| **ADD MATH** | MATEMATIK TAMBAHAN (1) only |

> Note: Column names in `LT_MATAPELAJARAN` may vary by dataset. Always `DESCRIBE` to verify (`KATEGORI_STEM` may be missing in older snapshots).

For teacher-side STEM queries, see [teachers.md → Guru STEM Menengah Atas](teachers.md#pointer-guru-stem-menengah-atas). Key subject codes: `E101`, `E110`, `E202`, `A804`, `E204`, `E205`, `E207`, `E321`, `E322`, `E324`, `E605`, `E606`, `E607`, `E609`. Key class codes for Menengah Atas: `1006`, `1007`, `1026`, `1027`, `1035`.

---

## Pointer: Murid Mengambil Mata Pelajaran Tertentu

Use when user asks:
- `BIL_MURID_MP_SABK_SMKA`
- `BILANGAN MURID BAGI MP BAHASA TAMIL`
- `MURID MEMPUNYAI MATA PELAJARAN BAHASA CINA KOMUNISASI`

**Construction**:
- Source: `T_SubjekAmbilMurid` (join `LTI_SubjekAmbilMurid` on `ID_MP` for subject names).
- Join murid base on `ID_INDIVIDU = IDMURID`.
- Filter by subject name (`MP IN (...)`) or subject code.
- `COUNT(DISTINCT ID_INDIVIDU)` grouped by `NEGERI`, `PPD`, `JENISSEKOLAH`, `KODTINGKATANTAHUN`, subject.
- Apply PraIPG exclusion on the school join.

---

## Pointer: DLP (Dual Language Programme) Detection

Use when user asks:
- `MURID DLP`
- `GURU DLP`
- `SUBJEK DLP`

**Construction**:
- DLP implementation varies. **Always inspect schema first**:
  - `SELECT table_name, column_name FROM information_schema.columns WHERE upper(table_name) LIKE '%DLP%' OR upper(column_name) LIKE '%DLP%'`
- Possible sources:
  - A dedicated DLP table or view
  - `LT_MATAPELAJARAN.MP_DLP` — DLP subject flags
  - `TGMATAPELAJARAN.STATUSDLP` — teacher DLP status
- Build the join against the confirmed source.
