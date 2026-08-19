# Research Pre-Extraction Heuristics for Messy Excel Layouts

Type: research
Status: resolved
Blocked by: none

## Question

What are the most effective open-source Python libraries (`openpyxl`, `calamine`, `fastexcel`, `polars`) and heuristic algorithms for detecting data regions, header rows (e.g., headers at row 30), multi-level headers, and merged cell ranges without loading full sheets into LLM context?

## Answer

### Key Findings & Architecture
1. **Parser Selection:** Use **`python-calamine`** (Rust core) for sub-10ms raw 2D grid extraction across `.xlsx`, `.xls` (BIFF8), `.xlsb`, and `.ods`. It is 10x–50x faster than `openpyxl` and uses minimal memory.
2. **Fast Merged Cell Extraction:** For `.xlsx`, use streaming XML `iterparse` on `xl/worksheets/sheetN.xml` targeting `<mergeCell ref="..."/>` (<5ms without DOM overhead). For `.xls`, use `xlrd`.
3. **Multi-Feature Header Scoring Algorithm:** Score candidate rows using a composite weighted metric:
   - **Type transition** ($w_1=0.25$, string ratio)
   - **Unique string ratio** ($w_2=0.20$)
   - **Horizontal row completeness** ($w_3=0.15$)
   - **Lookahead column type homogeneity** ($w_4=0.30$, strongest discriminator inspecting 5–10 rows below)
   - **Lexical bonus/penalty** ($w_5=0.10$, keywords like `id`, `date`, `amount`)
4. **Multi-Level Headers & Merged Cells:** Horizontally forward-fill merged spans on header rows, flatten hierarchy into sanitized snake_case names (`2023_actuals_q1`), vertically forward-fill merged dimension cells, and truncate at summary rows (`Total`, `Grand Total`).
5. **Execution Integration:** Pass detected `header_row` and bounding box directly into `polars.read_excel(engine="calamine", read_options={"header_row": r})` for zero-overhead migration.
