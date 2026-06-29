# Infrastructure (Infrastruktur)

Physical assets, classrooms, sanitation, buildings, land, and utilities data.

> **Always `DESCRIBE` first.** Column names drift between monthly snapshots — verify before writing any query.

---

## Classroom Counts

### `"BILBD"`

**Purpose**: Classroom (Bilik Darjah) count per school, broken down by classroom type.

**Columns:**

| Column | Type | Description |
|--------|------|-------------|
| `KODSEKOLAH` | VARCHAR | School code |
| `JBilik` | VARCHAR | Classroom type (e.g., 'Perdana', 'Prasekolah', 'Khas', 'P.KHAS') |
| `Bil` | BIGINT | Number of classrooms |

**Filter values**:
- `JBilik = 'P.KHAS'` (use `upper(JBilik) = 'P.KHAS'` to be case-safe) for Pendidikan Khas classrooms.
- Other common values: `'Perdana'`, `'Prasekolah'`, `'Khas'`.

---

## Rooms, Buildings, Assets, Land

### `"TSRUANG"`

**Purpose**: Room/space data within school buildings — now the primary source for all room types including classrooms, toilets, labs, and other spaces. Replaces standalone tables like `TSTANDAS`.

**Key columns:** `KODSEKOLAH`, `KODJENISRUANG` (joins to `TKJENISRUANG`)

### `"TSBLOK"`

**Purpose**: Building block data. Records each building block within a school compound.

**Key columns:** `KODSEKOLAH`, `KODBLOK`, `LUASLANTAI`, `KODSTATUSBLOK`

### `"TSHARTA"`

**Purpose**: School assets/inventory. Records movable and fixed assets.

**Key columns:** `KODSEKOLAH`, `KODJENISHARTA`, `KODPEMBIAYA`, `KUANTITI`, `KEADAAN`

### `"TSTANAH"`

**Purpose**: School land data. Records land parcels owned by the school.

**Key columns:** `KODSEKOLAH`, `LUASTANAH`, `KODMILIK`

---

## Utility Lookup Tables

| Table | Description | Join Column |
|-------|-------------|-------------|
| `"TKSUMBERELEKTRIK"` | Electricity source types | `KODSUMBERELEKTRIK` |
| `"TKBEKALANAIR"` | Water supply types | `KODSUMBERAIR` |
| `"TKSTATUSBANGUNAN"` | Building condition/status | `KODSTATUSBANGUNAN` |
| `"TKSTATUSBEKALANELEKTRIK"` | Electricity supply status | `KODSTATUSBEKALANELEKTRIK` |
| `"TKJENISRUANG"` | Room type classification | `KODJENISRUANG` |
| `"TKJENISHARTA"` | Asset type classification | `KODJENISHARTA` |
| `"TKKEMUDAHAN_OKU"` | OKU facility types | `KODKEMUDAHAN_OKU` |
| `"TSKEMUDAHAN_OKU"` | School OKU facilities data | `KODSEKOLAH` |
| `"TSLABELSEKOLAH"` | School label/certification data | `KODSEKOLAH` (joins to `TKLABELSEKOLAH`) |

---

## Pointer: Combined Aggregate (Murid + Guru + Sekolah + BD)

For "Bilangan Murid + Guru + Sekolah + BD" in one statement, build four separate `COUNT(DISTINCT ...)` / `SUM(...)` sub-selects from the four base tables (`T_SenaraiMuridALL+PraIPG`, `T_EPGO_SenaraiGuruIsiPjawatan+PraIPG`, `T_SekolahBeroperasi+PraIPG`, `BILBD`) — or one row per metric — joined to the school base for the geography filter. See [SKILL.md → How to Construct a Query](../SKILL.md#how-to-construct-a-query) step 1-6.

---

## Pointer: Bilik Darjah Pendidikan Khas

Use when user asks:
- `BILIK DARJAH PPKI`
- `LUAS BILIK DARJAH PPKI`
- `DATA MAKLUMAT FIZIKAL PROGRAM PENDIDIKAN KHAS INTEGRASI`

**Construction**:
- Filter `BILBD` with `upper(JBilik) = 'P.KHAS'`.
- `SUM(Bil)` grouped by `NEGERI`, `PPD`, `KODSEKOLAH`, `NAMASEKOLAH` (after school join).
- For the "luas" variant, join `TSRUANG` instead and filter by the Pendidikan Khas ruang type.

---

## Pointer: Bilik Darjah / Jenis Ruang (Grouped by JBilik)

Use when user asks:
- `INVENTORI_BILIK_JENIS_RUANG`
- `BILANGAN KELAS DAN BILIK DARJAH`
- `BILIK MATEMATIK`

**Construction**:
- Source: `BILBD`.
- Always join with `T_SekolahBeroperasi+PraIPG` to get school context.
- `SUM(Bil)` grouped by `NEGERI`, `PPD`, `KODSEKOLAH`, `NAMASEKOLAH`, `JBilik`.
- Apply PraIPG exclusion on the school join.

---

## Pointer: Asrama / Sumber Elektrik / Pedalaman

Schema inspection and asrama/elektrik/pedalaman queries live in [schools.md](schools.md#pointer-asrama-hostel-schema-inspection) — see "Asrama (Hostel) Schema Inspection" and "Sumber Elektrik / Internet / Pedalaman" sections.
