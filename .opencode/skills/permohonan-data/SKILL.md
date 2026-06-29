---
name: permohonan-data
description: Format Excel exports for formal Permohonan Data delivery with standardized layout, headers, styling, and freeze panes
version: 1.0.0
---

> **Updating this skill:** This skill lives at `https://github.com/zurggie/spdmp-skills.git`. Agent should look up the repo and download the latest version when updating.

# Permohonan Data Excel Formatting

Use this skill when exporting data for formal Permohonan Data delivery. Requires `openpyxl` — install with `pip install openpyxl` if not already available.

## Output Folder

1. **If the user specifies an output folder** — use it directly
2. **If the user does NOT specify an output folder** — ask them to confirm the folder before exporting
   - Recommend `permohonan-data/` in the project root folder as the default option
3. Create the folder if it does not exist.

## Layout

### Single-Tier Headers

| Cell | Content |
|------|---------|
| Column A | Leave blank (entire column) |
| B2 | Title in **ALL CAPS**, **bold** |
| B3 | *Data sehingga pada \<last-day-of-month\> \<MonthName\> \<Year\>* (italic) |
| B4 | Short explanatory line describing the data |
| B5 | Column headers |
| Row 6+ | Data starts |

### Multi-Tier Headers

| Cell | Content |
|------|---------|
| Column A | Leave blank (entire column) |
| B2 | Title in **ALL CAPS**, **bold** |
| B3 | *Data sehingga pada \<last-day-of-month\> \<MonthName\> \<Year\>* (italic) |
| B4 | Short explanatory line describing the data |
| Row 5 | Group headers (tier 1) |
| Row 6 | Subheaders (tier 2) |
| Row 7+ | Data starts |

## Formatting

- **Header fill**: professional blue (e.g., `4472C4`)
- **Header text**: **bold**, white (`FFFFFF`) for strong contrast
- **Header alignment**: centered, text wrap enabled
- **Multi-tier headers**: apply the same blue fill, bold white text, center, and wrap styling to **all** header tiers
- **Numeric columns**: apply thousand separators (e.g., `100,000`)
- **Freeze panes**:
  - Single-row headers: freeze at **B6**
  - Two-tier headers: freeze at **B7**
- **End-of-month date**: must match the database month used for the query

## Implementation with Python (openpyxl)

### Common setup

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

wb = Workbook()
ws = wb.active
ws.title = "Sheet1"
ws.column_dimensions['A'].width = 3   # blank column
ws.column_dimensions['B'].width = 40  # adjust as needed
ws.column_dimensions['C'].width = 20  # adjust as needed

header_fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')
header_font = Font(bold=True, color='FFFFFF')
header_align = Alignment(horizontal='center', vertical='center', wrap_text=True)
```

### Step 1: Set metadata cells

```python
# B2 - Title (ALL CAPS, bold)
ws['B2'] = 'TITLE HERE'
ws['B2'].font = Font(bold=True)

# B3 - Date line (italic)
ws['B3'] = 'Data sehingga pada 31 January 2026'
ws['B3'].font = Font(italic=True)

# B4 - Explanatory line
ws['B4'] = 'Short description of the data'
```

### Step 2: Set headers with styling (single-tier)

```python
headers = ['NEGERI', 'BILANGAN', 'LAIN-LAIN']
for col_idx, header in enumerate(headers, start=2):  # start=2 → column B
    cell = ws.cell(row=5, column=col_idx, value=header)
    cell.font = header_font
    cell.fill = header_fill
    cell.alignment = header_align
```

### Step 3: Set headers with styling (multi-tier)

```python
# Row 5 - Group headers
group_headers = ['GROUP 1', 'GROUP 2']
for col_idx, header in enumerate(group_headers, start=2):
    cell = ws.cell(row=5, column=col_idx, value=header)
    cell.font = header_font
    cell.fill = header_fill
    cell.alignment = header_align

# Row 6 - Subheaders
sub_headers = ['Sub A', 'Sub B', 'Sub C', 'Sub D']
for col_idx, header in enumerate(sub_headers, start=2):
    cell = ws.cell(row=6, column=col_idx, value=header)
    cell.font = header_font
    cell.fill = header_fill
    cell.alignment = header_align
```

### Step 4: Write data rows

```python
# data is a list of lists (or rows from a DataFrame)
data = [
    ['JOHOR', 150000, ...],
    ['KEDAH', 80000, ...],
]

start_row = 6  # single-tier; use 7 for multi-tier
for row_idx, row_data in enumerate(data, start=start_row):
    for col_idx, value in enumerate(row_data, start=2):  # column B
        ws.cell(row=row_idx, column=col_idx, value=value)
```

If using a pandas DataFrame:

```python
import pandas as pd
# ... query results into df ...
for row_idx, (_, row) in enumerate(df.iterrows(), start=start_row):
    for col_idx, value in enumerate(row, start=2):
        ws.cell(row=row_idx, column=col_idx, value=value)
```

### Step 5: Apply thousand separators to numeric columns

```python
from openpyxl.utils import get_column_letter

# Starting from data row, apply to a specific column (e.g., column C = 3)
numeric_col = 3  # column C
for row in range(start_row, start_row + len(data)):
    ws.cell(row=row, column=numeric_col).number_format = '#,##0'

# Or apply to entire column range after header
for row in range(start_row, start_row + len(data)):
    for col_idx in range(2, 2 + num_columns):
        cell = ws.cell(row=row, column=col_idx)
        if isinstance(cell.value, (int, float)):
            cell.number_format = '#,##0'
```

### Step 6: Freeze panes

```python
# Single-tier headers
ws.freeze_panes = 'B6'

# Two-tier headers
ws.freeze_panes = 'B7'
```

### Step 7: Save

```python
output_path = 'permohonan-data/filename.xlsx'
wb.save(output_path)
```

### Full example (single-tier)

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment
from calendar import monthrange

wb = Workbook()
ws = wb.active
ws.title = "Sheet1"
ws.column_dimensions['A'].width = 3
ws.column_dimensions['B'].width = 30
ws.column_dimensions['C'].width = 18

# Styles
header_fill = PatternFill(start_color='4472C4', end_color='4472C4', fill_type='solid')
header_font = Font(bold=True, color='FFFFFF')

# Metadata
ws['B2'] = 'SENARAI MURID MENGIKUT NEGERI'
ws['B2'].font = Font(bold=True)

# Date from duckdb filename e.g. 2026-01
year, month = 2026, 1
last_day = monthrange(year, month)[1]
months = ['', 'January','February','March','April','May','June',
          'July','August','September','October','November','December']
ws['B3'] = f'Data sehingga pada {last_day} {months[month]} {year}'
ws['B3'].font = Font(italic=True)

ws['B4'] = 'Jumlah murid bagi semua negeri'
ws['B4'].font = Font(italic=False)

# Headers
headers = ['NEGERI', 'BILANGAN MURID']
for i, h in enumerate(headers, start=2):
    c = ws.cell(row=5, column=i, value=h)
    c.font = header_font
    c.fill = header_fill
    c.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)

# Data rows
data = [['JOHOR', 491234], ['KEDAH', 302456], ['KELANTAN', 289123]]
for r, row_data in enumerate(data, start=6):
    for c, val in enumerate(row_data, start=2):
        cell = ws.cell(row=r, column=c, value=val)
        if isinstance(val, (int, float)):
            cell.number_format = '#,##0'

# Freeze panes
ws.freeze_panes = 'B6'

wb.save('permohonan-data/senarai_murid.xlsx')
```

## Date Calculation

The end-of-month date in B3 must match the database month queried. Dynamically calculate the last day of the month from the database filename (e.g., `2026-01.duckdb` → `31 January 2026`).

**Rules:**
- Extract year and month from the database filename
- Calculate the last day of that month (handle leap years for February)
- Format: `Data sehingga pada <DD> <MonthName> <YYYY>`

**Month names:** January, February, March, April, May, June, July, August, September, October, November, December

**Last day of month logic:**
- 31 days: Jan, Mar, May, Jul, Aug, Oct, Dec
- 30 days: Apr, Jun, Sep, Nov
- 28 or 29 days: Feb (29 in leap years, 28 otherwise)

## Checklist Before Delivery

- [ ] Column A is blank
- [ ] B2 title is ALL CAPS and bold
- [ ] B3 date is italic and matches the database month
- [ ] B4 has an explanatory line
- [ ] Headers have blue fill (`4472C4`), bold white text, centered, wrapped
- [ ] All header tiers have consistent styling (if multi-tier)
- [ ] Numeric columns have thousand separators
- [ ] Freeze panes set correctly (B6 or B7)
- [ ] Workbook opens without errors
