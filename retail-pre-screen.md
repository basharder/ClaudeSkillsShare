# Excel Batch Output Format Spec

## When to Use
Load this file when generating an `.xlsx` batch output (4+ models). Replicate the exact structure from `assets/Complete_example___formatting_template.xlsx`. Do not approximate — match it precisely.

---

## Sheet Name
`Xref Tracker`

---

## Row Structure

| Row | Content |
|---|---|
| 1 | Title: **"Canarm Bath Fan Cross-Reference Tracker"** — merged A1:P1, dark navy fill (`#1F3864`), white bold 14pt font, row height 27.75 |
| 2 | Subtitle: `SOP: Bath Fan Cross-Reference Validation v1.0  |  Source: Canarm_Model_data.xlsx  |  Auto-updated per batch run` — merged A2:P2, medium blue fill (`#2E75B6`), white 9pt font, row height 15.75 |
| 3 | **Headers** — dark navy fill (`#1F3864`), white bold font, row height 30 |
| 4+ | Data rows — light blue fill (`#F2F7FF`) for all cells except Classification (col E) and Correction Cost Exposure (col O) |

---

## Column Spec (exact order and widths)

| Col | Header | Width |
|---|---|---|
| A | # | 4 |
| B | Brand | 12 |
| C | Competitor SKU | 22 |
| D | Canarm Match | 14 |
| E | Classification | 16 |
| F | Confidence | 10 |
| G | Constraining Factor | 42 |
| H | Depth Δ (in) | 10 |
| I | CFM Δ% | 8 |
| J | Sones Δ | 9 |
| K | Duct | 7 |
| L | Motor | 12 |
| M | Speed Class | 18 |
| N | Run Date | 12 |
| O | Correction Cost Exposure | 22 |
| P | Exposure Driver | 36 |

---

## Color Coding

### Classification (Column E)
| Classification | Fill Color (hex) |
|---|---|
| TRUE | `#C6EFCE` (green) |
| Best Option | `#FFEB9C` (yellow) |
| Best Option ⚠ | `#FFD966` (amber — borderline cases) |
| High Risk | `#FFC7CE` (red) |
| N/A — ACCESSORY | `#DDEBF7` (light blue) |
| NO MATCH | `#FFC7CE` (red, same as High Risk) |

### Correction Cost Exposure (Column O)
| Rating | Fill Color (hex) |
|---|---|
| LOW | `#C6EFCE` (green) |
| MEDIUM | `#FFEB9C` (yellow) |
| HIGH | `#FFC7CE` (red) |
| CRITICAL | `#FF0000` with white bold font |

---

## Additional Formatting Rules

- All data rows: height 21.75 (auto-wrapping allowed for long Constraining Factor text)
- Freeze panes at row 4 (A4)
- Font: Calibri 11pt throughout (or Arial as fallback)
- Text wrap enabled on columns G and P
- Use `—` (em dash) for N/A numeric delta fields, not blank or "N/A"
- Depth Δ: use `0.00` for exact matches, numeric value for mismatches
- CFM Δ%: numeric (e.g., `9.1`, `+10.0`) — no % symbol in cell
- Sones Δ: include sign (e.g., `+0.2`, `−0.7`, `0.0`)
- Run Date: ISO format `YYYY-MM-DD`

---

## Python Generation Pattern

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment
from openpyxl.utils import get_column_letter

wb = Workbook()
ws = wb.active
ws.title = "Xref Tracker"

# Title row (merged A1:P1)
ws.merge_cells('A1:P1')
ws['A1'] = 'Canarm Bath Fan Cross-Reference Tracker'
ws['A1'].font = Font(bold=True, size=14, color='FFFFFF')
ws['A1'].fill = PatternFill('solid', fgColor='1F3864')
ws['A1'].alignment = Alignment(horizontal='left', vertical='center')
ws.row_dimensions[1].height = 27.75

# Subtitle row (merged A2:P2)
ws.merge_cells('A2:P2')
ws['A2'] = 'SOP: Bath Fan Cross-Reference Validation v1.0  |  Source: Canarm_Model_data.xlsx  |  Auto-updated per batch run'
ws['A2'].font = Font(size=9, color='FFFFFF')
ws['A2'].fill = PatternFill('solid', fgColor='2E75B6')
ws['A2'].alignment = Alignment(horizontal='left', vertical='center')
ws.row_dimensions[2].height = 15.75

# Header row
header_fill = PatternFill('solid', fgColor='1F3864')
header_font = Font(bold=True, color='FFFFFF', size=11)
headers = ['#','Brand','Competitor SKU','Canarm Match','Classification','Confidence',
           'Constraining Factor','Depth Δ (in)','CFM Δ%','Sones Δ','Duct','Motor',
           'Speed Class','Run Date','Correction Cost Exposure','Exposure Driver']
for col_idx, header in enumerate(headers, 1):
    cell = ws.cell(row=3, column=col_idx, value=header)
    cell.font = header_font
    cell.fill = header_fill
    cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
ws.row_dimensions[3].height = 30

# Freeze panes
ws.freeze_panes = 'A4'

# Column widths
col_widths = [4,12,22,14,16,10,42,10,8,9,7,12,18,12,22,36]
for i, width in enumerate(col_widths, 1):
    ws.column_dimensions[get_column_letter(i)].width = width

# Classification color map (col E)
class_colors = {
    'TRUE': 'C6EFCE',
    'Best Option': 'FFEB9C',
    'Best Option ⚠': 'FFD966',
    'High Risk': 'FFC7CE',
    'N/A — ACCESSORY': 'DDEBF7',
    'NO MATCH': 'FFC7CE',
}

# Correction Cost Exposure color map (col O)
exposure_colors = {
    'LOW': 'C6EFCE',
    'MEDIUM': 'FFEB9C',
    'HIGH': 'FFC7CE',
    'CRITICAL': 'FF0000',
}

# Data row fill
data_fill = PatternFill('solid', fgColor='F2F7FF')
```

---

## File Naming

`Canarm_Xref_Batch_[BRAND]_[YYYY-MM-DD].xlsx`
e.g., `Canarm_Xref_Batch_Broan_2026-03-01.xlsx`

Multi-brand: `Canarm_Xref_Batch_Mixed_[YYYY-MM-DD].xlsx`
