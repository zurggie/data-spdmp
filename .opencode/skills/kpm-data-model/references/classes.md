# Classes (Kelas)

Class-level data — class lists, grade levels, streams, and class sizes.

> **Always `DESCRIBE` first.** Column names drift between monthly snapshots — verify before writing any query.

---

## `"VIEW_DATA_KELAS_EPRD"`

**Purpose**: EPRD class view. Primary source for class-level data — class names, grade/level, stream, and student count per class.

**Key columns:**

| Column | Type | Description |
|--------|------|-------------|
| `IDKELAS` | DOUBLE | Unique class ID — joins to student tables |
| `NAMAKELAS` | VARCHAR | Class name (e.g., "1 ARIF", "5 SAINS 1") |
| `IDKODTINGKATANTAHUN` | DOUBLE | Grade level ID — joins to `LTTINGKATANTAHUN` |
| `SESIKELAS` | VARCHAR | Class session (Pagi / Petang) |
| `KODSEKOLAH` | VARCHAR | School code |
| `INTALIRAN` | DOUBLE | Stream code — Science/Arts/Vocational |
| `INTBIDANG` | DOUBLE | Field of study code |
| `IDKODNEGERI` | VARCHAR | State code |
| `BILMURID` | DOUBLE | Number of students in the class |

**When to use**: Class-level aggregations — number of classes per school, average class size, class distribution by stream or grade.

**Joins with:**
- `"T_SenaraiMuridALL+PraIPG"` ON `IDKELAS`
- `"VIEW_DATA_MURID_EPRD"` ON `IDKELAS`
- `"LTTINGKATANTAHUN"` ON `IDKODTINGKATANTAHUN`

**Alternative source**: Classes can also be derived from `T_SenaraiMuridALL+PraIPG` using `IDKELAS` and `NAMAKELAS` directly — useful when the class view isn't loaded for a snapshot.

---

## Grade Level Lookups

The grade/form level code is stored across multiple tables:

| Table | Key Column | Description Column |
|-------|-----------|-------------------|
| `"LTTINGKATANTAHUN"` | `IDKODTINGKATANTAHUN` (DOUBLE), `KODTINGKATANTAHUN` (VARCHAR) | `KETERANGANTINGKATANTAHUN` |
| `"LT_TINGKATAN_TAHUN"` | alternate variant | similar structure |

Also available in some tables as `KODTINGKATANTAHUN` (VARCHAR) directly. Order by `LTTINGKATANTAHUN.SUSUNAN` to display grades in the natural progression (T1, T2, ..., F1, F2, ...).

---

## Pointer: Bilangan Kelas Mengikut Sekolah Dan Tingkatan

Use when user asks:
- `BILANGAN_KELAS_MENGIKUT_SEKOLAH_TINGKATAN`
- `BILANGAN KELAS SMKA SABK RENDAH DAN MENENGAH`
- `BILANGAN KELAS PRA SEKOLAH`

**Construction**:
- Source: `T_SenaraiMuridALL+PraIPG` (via `IDKELAS` + `NAMAKELAS`) or `VIEW_DATA_KELAS_EPRD` (via `IDKELAS` + `BILMURID`).
- `COUNT(DISTINCT IDKELAS)` for class count; `COUNT(DISTINCT IDMURID)` for student count.
- Group by `NEGERI`, `PPD`, `KODSEKOLAH`, `NAMASEKOLAH`, `KODTINGKATANTAHUN` (and `NAMAKELAS` for per-class detail).
- Apply PraIPG exclusion if scoped to operating schools.

---

## Pointer: Kelas Melebihi Kapasiti

Use when user asks:
- `SENARAI KELAS KAPASITI LEBIH 30 MURID`
- `ANALISIS NEGERI KELAS LEBIH 40 MURID`

**Construction**:
- Build a CTE with `KODSEKOLAH`, `IDKELAS`, `NAMAKELAS`, `KODTINGKATANTAHUN`, plus a `COUNT(DISTINCT IDMURID)` (or use `BILMURID` from `VIEW_DATA_KELAS_EPRD`).
- Filter `class_size > {threshold}`.
- Join to school table for context.
- `ORDER BY NEGERI, PPD, KODSEKOLAH, KODTINGKATANTAHUN, class_size DESC`.
