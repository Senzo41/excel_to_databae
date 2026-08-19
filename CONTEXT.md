# Domain Glossary: excel_to_database

This document defines the canonical domain terms for `excel_to_database`. All specifications, code, and tickets should adhere to these definitions.

---

## Source Concepts

### Workbook
A single spreadsheet file (`.xlsx`, `.xls`, `.csv`) containing one or more **Sheets**.

### Sheet
A 2D grid within a **Workbook**. May contain one or more independent or messy **Data Regions**, metadata comments, non-standard headers, and merged cells.

### Data Region
The contiguous rectangular block of actual data rows and columns within a **Sheet**, isolated from title blocks, summary footnotes, or decorative empty spaces.

### Header Candidate
A detected row (or multi-row range) within a **Data Region** representing column labels, which may be located deep in the sheet (e.g., row 30) or contain unnamed/ambiguous headers.

---

## Schema & Transformation Concepts

### Target Schema
The unified, normalized destination data model (tables, columns, types, and primary/foreign keys) proposed by the AI Agent.

### Filename Metadata Extraction
The process of parsing temporal or categorical dimensions (e.g., year, department, region, version) from the source file path/name into synthetic columns during ingestion.

### Column Mapping
The specific semantic correspondence between a messy source column (across one or more **Sheets** or filename metadata) and a canonical column in the **Target Schema**, including type casting and data cleaning rules.

### Transformation Operation
An atomic, parameter-driven transformation rule (e.g., `DIRECT`, `CAST`, `CLEAN_STRING`, `PARSE_DATE`, `REGEX_EXTRACT`, `MAP_VALUES`, `FALLBACK_VALUE`, `EXTRACT_FROM_FILENAME`) executed by the **Polars** engine.

### Conflict Strategy
The policy for resolving duplicate keys or record collisions during entity normalization and union operations (`KEEP_FIRST`, `KEEP_LAST`, `MERGE_NON_NULL`, `FAIL_ON_CONFLICT`).

### Mapping Plan
The comprehensive, structured transformation specification detailing how source **Data Regions** and metadata are extracted, cleaned, unified, and loaded into the **Target Schema**.

---

## Storage & Execution Concepts

### Target Sink
The physical destination format or database where the structured data is written (e.g., Parquet file tree, DuckDB, SQLite, PostgreSQL).

### Migration Job
The end-to-end execution of a **Mapping Plan** using **Polars** to transform raw data from source **Workbooks** and write it into the selected **Target Sink**.

### Review Session
The interactive review phase in the UI where the user inspects the AI's proposed **Target Schema** and **Mapping Plan**, makes manual adjustments, and approves the **Migration Job**.
