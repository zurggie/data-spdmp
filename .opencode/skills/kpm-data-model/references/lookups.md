# Lookups & Classification Tables

All `LT_` (lookup/descriptive) and `TK_` (classification) tables used to decode codes into human-readable labels. Join these tables on `KOD...` columns to add descriptions to your queries.

> **Standard join pattern:** `LEFT JOIN "LT..." lk ON m.KOD... = lk.KOD...` — pick the lookup whose join column matches the code column on your fact table.

## Naming Convention

Lookup tables follow a consistent pattern:

| Prefix | Domain | Examples |
|--------|--------|----------|
| `LT` (no underscore) | **Student-related** | `LTKAUM`, `LTJANTINA`, `LTAGAMA`, `LTTINGKATANTAHUN` |
| `LT_` (with underscore) | **Teacher/staff-related** | `LT_KAUM`, `LT_JAWATANGURU`, `LT_MATAPELAJARAN`, `LT_OPSYEN` |

The same concept can have **two separate tables** for students vs teachers:

| Concept | Student table | Teacher table |
|---------|-------------|---------------|
| Ethnicity | `LTKAUM` | `LT_KAUM` |
| Religion | `LTAGAMA` | `LT_AGAMA` |
| Grade/Form level | `LTTINGKATANTAHUN` | `LT_TINGKATAN_TAHUN` |

Always join with the correct table for your data source! Joining `T_SenaraiMuridALL+PraIPG` with `LT_KAUM` (teacher ethnicity) would give no matches.

---

## Student Demographics

### `"LTKAUM"`
| Column | Description |
|--------|-------------|
| `KODKAUM` | Ethnicity code |
| `KETERANGANJENISKAUM` | Ethnicity name (e.g., Melayu, Cina, India) |
| `KODPENGELASAN` | **Pengelasan** / grouping code — this is what most queries group by |
| `STATUSBUMIPUTERA` | Bumiputera status |

**Pengelasan codes** (use `KODPENGELASAN` for ethnic grouping):

| Code | Meaning |
|------|---------|
| `M` | MELAYU |
| `C` | CINA |
| `I` | INDIA |
| `A` | ORANG ASAL |
| `X` | BUMIPUTERA SABAH |
| `Y` | BUMIPUTERA SARAWAK |
| `L` | LAIN-LAIN |
| NULL | TIADA MAKLUMAT (sometimes grouped under `L`) |

**Grouping pattern**: `CASE WHEN m.KODKAUM IS NULL THEN 'L' ELSE lk.KODPENGELASAN END` to coalesce NULLs into `L`.

**Used by:** Student tables (`KODKAUM`)
> For **teacher** ethnicity codes, use `"LT_KAUM"` instead.

### `"LTJANTINA"`
| Column | Description |
|--------|-------------|
| `KODJANTINA` | Gender code |
| `KETERANGAN` | Gender (e.g., Lelaki / Perempuan) |

**Used by:** Student and teacher tables (`KODJANTINA` / `JANTINA`)

### `"LTAGAMA"` (students)
| Column | Description |
|--------|-------------|
| `KODAGAMA` | Religion code |
| `KETERANGAN` | Religion name |

**Used by:** `VIEW_DATA_MURID_EPRD`
> For **teacher** religion codes, use `"LT_AGAMA"` instead. Used by `TGSTAF`.

### `"LT_AGAMA"` (teachers)
Same structure — separate table for teacher/staff religion codes.

### `"LT_WARGANEGARA"`
| Column | Description |
|--------|-------------|
| `KODWARGANEGARA` | Citizenship code |
| `KETERANGAN` | Citizenship status |

**Used by:** `TGSTAF`

### `"LTALIRAN"`
| Column | Description |
|--------|-------------|
| `KODALIRAN` | Stream code |
| `KETERANGANALIRAN` | Stream name (Sains, Sastera, Vokasional, etc.) |

**Used by:** `VIEW_DATA_KELAS_EPRD`, student tables

---

## Education Levels

### `"LTTINGKATANTAHUN"` (students)
| Column | Description |
|--------|-------------|
| `IDKODTINGKATANTAHUN` | Numeric level ID |
| `KODTINGKATANTAHUN` | Level code (e.g., T1, T2, T3, F1, F2, F3...) |
| `KETERANGANTINGKATANTAHUN` | Description (Tahun 1, Tahun 2, Tingkatan 1...) |
| `SUSUNAN` | Sort order |

**Used by:** Student tables (`KODTINGKATANTAHUN`), class tables (`IDKODTINGKATANTAHUN`)
> For **teacher** grade/form level lookup, use `"LT_TINGKATAN_TAHUN"` instead.
>
> Order by `SUSUNAN` to display grades in natural progression.

Example KODTINGKATANTAHUN → KETERANGANTINGKATANTAHUN mappings: `T1` → Tahun 1, `T2` → Tahun 2, `F1` → Tingkatan 1, `F5` → Tingkatan 5, `PRASEKOLAH` → Prasekolah.

### `"LT_MATAPELAJARAN"`
| Column | Description |
|--------|-------------|
| `KODMATAPELAJARAN` | Subject code |
| `MATAPELAJARAN` | Subject name (Bahasa Melayu, English, Matematik...) |
| `PERINGKAT` | Education level (Rendah / Menengah) |
| `KOD_KUM_MATAPELAJARAN` | Subject group code |
| `MP_DLP` | DLP (Dual Language Programme) status |

**Used by:** `TGMATAPELAJARAN`, `T_SubjekAmbilMurid` (via `ID_MP`)

### `"LT_OPSYEN"`
| Column | Description |
|--------|-------------|
| `KOD_OPSYEN` | Specialisation code |
| `OPSYEN` | Specialisation name (e.g., Pendidikan Seni, BM, Sejarah) |
| `OPSYEN_DOMINAN` | Dominant/Main opsyen flag |

**Used by:** `TGIKHTISAS`

---

## Teacher / Staff Classifications

### `"LT_JAWATANGURU"`
| Column | Description |
|--------|-------------|
| `KODJAWATAN` | Position code |
| `KET_JWT_1` | Position category (Guru Penolong, Guru Cemerlang, etc.) |
| `JAWATAN_TADBIR` | Administrative position flag |
| `PENGELASAN` | Classification |

| Example Code | Meaning |
|-------------|---------|
| GL17 | Guru Lepasan 17 |
| DG41 | Guru DG41 |
| DG48 | Guru DG48 |
| DG54 | Guru DG54 |

**Used by:** `T_EPGO_SenaraiGuruIsiPjawatan+PraIPG` (`KODJAWATAN`), `TGSTAF`

### `"LT_TUGASKHAS"`
| Column | Description |
|--------|-------------|
| `KODTUGASKHAS` | Special duty code |
| `KETERANGAN` | Duty description (Kaunselor, LIBRARIAN, SPBT, etc.) |

**Used by:** `TGTUGASKHAS`

### `"LT_AKADEMIKTINGGI"`
| Column | Description |
|--------|-------------|
| `KODAKADEMIKTINGGI` | Academic qualification code |
| `KETERANGAN` | Qualification name (PhD, Master, Bachelor, Diploma) |

**Used by:** `TGKELULUSANAKADEMIK`

### `"LT_GREDPERGERAKAN"`, `"LT_TARAFJWT"`, `"LT_PENGISIANJAWATAN"`

Additional teacher/staff classification tables for salary grade movement, position level, and position filling type.

---

## School Classifications (`TK_` prefix)

### `"TKJENISSEKOLAH"`

The most important school lookup — 17 columns with school type hierarchy.

| Column | Description |
|--------|-------------|
| `KODJENISSEKOLAH` | School type code |
| `JENISSEKOLAH` | School type name |
| `KODKUMPJENIS` | Group category |
| `KUMJENIS` | Group name (Kebangsaan, Jenis Kebangsaan, SM...) |
| `SM_SR` | Level flag (SEKOLAH RENDAH / SEKOLAH MENENGAH) |
| `QUICKFACTS` | Quickfacts mapping (SK, SJKC, SJKT, SMK, SBP, etc.) |

| Code | Type | Level | Quickfacts |
|------|------|-------|------------|
| 101 | SK | Rendah | SK |
| 103 | SJK(C) | Rendah | SJKC |
| 104 | SJK(T) | Rendah | SJKT |
| 201 | SMK | Menengah | SMK |
| 203 | SM Teknik | Menengah | SMT |
| 206 | SM Berasrama Penuh | Menengah | SBP |
| 211 | SBJK | Menengah+Rendah | SBJK |
| 408 | Program Pra di IPG | Program | — |

**Used by:** All school tables (`KODJENISSEKOLAH`)

### `"TKSTATUSSEKOLAH"`
| Code | Meaning |
|------|---------|
| — | School status (operating/tutup/sementara) |

**Used by:** `TSSEKOLAH` (`KODSTATUSSEKOLAH`)

### `"TKPERINGKATSEKOLAH"`
| Code | Meaning |
|------|---------|
| R | Rendah (Primary) |
| M | Menengah (Secondary) |

**Used by:** School tables

### `"TKLABELSEKOLAH"`
**Purpose**: School labels/accreditations (Sekolah Kluster, Sekolah Berprestasi Tinggi, etc.)

### `"TKJENISASRAMA"`
**Purpose**: Hostel/dormitory type classification. Join from `TSSEKOLAH.KODJENISASRAMA` for asrama queries — see [schools.md](schools.md#asrama-hostel-schema-inspection).

---

## Geographical Lookups

| Table | Column | Description |
|-------|--------|-------------|
| `"TKDAERAH"` | `KODDAERAH` | District code and name |
| `"TKNEGERI"` | `KODNEGERI` | State code and name |
| `"TKPARLIMEN"` | `KODPARLIMEN` | Parliament constituency |
| `"TKPPD"` | `KODPPD` | PPD (District Education Office) |
| `"TKDUN"` | `KODDUN` | State constituency |

**Used by:** `TSSEKOLAH` for geographical code joins. `T_SekolahBeroperasi+PraIPG` has denormalised names.

---

## Infrastructure Classification Tables

| Table | Description | Key Column |
|-------|-------------|------------|
| `"TKSTATUSBANGUNAN"` | Building condition/status | `KODSTATUSBANGUNAN` |
| `"TKSUMBERELEKTRIK"` | Electricity source type | `KODSUMBERELEKTRIK` |
| `"TKSTATUSBEKALANELEKTRIK"` | Electricity supply status | `KODSTATUSBEKALANELEKTRIK` |
| `"TKBEKALANAIR"` | Water supply type | `KODSUMBERAIR` |
| `"TKJENISRUANG"` | Room type (Bilik Darjah, Makmal, Perpustakaan...) | `KODJENISRUANG` |
| `"TKJENISHARTA"` | Asset type | `KODJENISHARTA` |
| `"TKKEMUDAHAN_OKU"` | OKU facility type | `KODKEMUDAHAN_OKU` |
| `"TKLOKASI"` | Location type (Bandar / Luar Bandar) | `KODLOKASI` |
| `"TKJENISBANTUAN"` | School aid type (Kerajaan / Bantuan) | `KODJENISBANTUAN` |
| `"TKJENISPEDALAMAN"` | Interior/remote classification | `KODJENISPEDALAMAN` |
| `"TKPEMBIAYA"` | Financing source | `KODPEMBIAYA` |

---

## Others

| Table | Description |
|-------|-------------|
| `"LTNEGARA"` | Country list |
| `"LTHUBPENJAGA"` | Guardian relationship types |
| `"LTSESI"` | School session types |
| `"LTSEBABPADAM"` | Deactivation reasons |
| `"LTSEBABTIDAKHADIR"` | Absence reasons |
| `"LTOKU"` | OKU disability types |
| `"LT_BERSARA"` | Retirement types |
| `"LT_JENISCACAT"` | Disability type classification |
| `"LT_STATUSKAHWIN"` / `"LT_TARAF_PERKAHWINAN"` | Marital status |
| `"LT_BANDAR"` | City/town lookup |
| `"LTI_SubjekAmbilMurid"` | Subject name lookup for `T_SubjekAmbilMurid` — join on `ID_MP` |
| `"LT_PJJ"` | Distance learning eligibility |
| `"LT_P_INSTITUSI"` | Institution type lookup |

---

## Quick Reference: Code Column Naming

Most lookup tables use `KOD...` for the code column and `KETERANGAN` or `NAMA` / `JENISSEKOLAH` / `MATAPELAJARAN` / `OPSYEN` for the description. Run `DESCRIBE` to verify the exact column name.
