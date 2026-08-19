# Define the Declarative Mapping Plan Schema Specification

Type: grilling
Status: resolved
Blocked by: 02

## Question

What is the exact data structure and validation model of the Declarative Mapping Plan (specifying source workbooks/sheets, data region boundaries, column mappings, type coercions, default values, null handling, joins/unions, and target sink definitions) that the AI agent produces and the Polars engine consumes?

## Answer

### 1. Specification Architecture
The Declarative Mapping Plan uses a **Target-Centric Model** where the root entity defines normalized destination tables (`TargetTableSpec`), and each table contains extraction mappings (`SourceExtractionMapping`) from one or more messy workbooks/sheets.

### 2. Transformation Operations Enum & Parameters
All transformations are strictly parameter-driven:
- `DIRECT`: Exact passthrough without modification.
- `CAST`: Safe type coercion to target SQL type (`INTEGER`, `FLOAT`, `BOOLEAN`, `DATE`, `TIMESTAMP`, `VARCHAR`).
- `CLEAN_STRING`: Strip whitespace, remove non-printable characters, apply case normalization (`LOWER`, `UPPER`, `TITLE`).
- `PARSE_DATE`: Parse date strings using specific format (`%Y-%m-%d`, `%d.%m.%Y`, or ISO autodetection).
- `REGEX_EXTRACT`: Extract regex capture groups (e.g. `^([A-Z]+)-(\d+)$` -> group 1).
- `MAP_VALUES`: Categorical value replacement dictionary (e.g. `{"Y": "TRUE", "N": "FALSE", "Ja": "TRUE"}`).
- `FALLBACK_VALUE` / `COALESCE`: Default value to inject if source value is null/empty.
- `EXTRACT_FROM_FILENAME`: Regex pattern to extract metadata embedded in the filename/path (e.g. `.*_(20\d{2})_Q([1-4]).xlsx` -> Year, Quarter) and inject as a synthetic column.

### 3. Conflict & Deduplication Strategies
Each target table defines:
- `primary_key`: List of column names forming the unique constraint.
- `deduplication_keys`: Columns checked for record uniqueness.
- `conflict_strategy`: `KEEP_FIRST` (default), `KEEP_LAST`, `MERGE_NON_NULL` (coalesces non-null values across duplicate records), or `FAIL_ON_CONFLICT`.

### 4. Decoupled Target Sink Configuration
Target sinks are configured independently of the logical mapping plan:
- `ParquetSinkConfig`: Target directory path, partition columns, compression (`zstd`, `snappy`).
- `DuckDBSinkConfig`: Database file path, table name, if_exists (`replace`, `append`).
- `SQLiteSinkConfig`: Database file path, table name, if_exists.
- `PostgresSinkConfig` / `MySQLSinkConfig`: Connection URL, schema, table name, batch size.

### 5. Concrete Pydantic Schema Model
```python
from pydantic import BaseModel, Field
from typing import List, Optional, Dict, Literal, Any

class ColumnTransformationSpec(BaseModel):
    operation: Literal[
        "DIRECT", "CAST", "CLEAN_STRING", "PARSE_DATE",
        "REGEX_EXTRACT", "MAP_VALUES", "FALLBACK_VALUE", "EXTRACT_FROM_FILENAME"
    ]
    target_type: Optional[str] = None
    date_format: Optional[str] = None
    string_case: Optional[Literal["LOWER", "UPPER", "TITLE"]] = None
    strip_whitespace: bool = True
    regex_pattern: Optional[str] = None
    regex_group: int = 1
    value_map: Optional[Dict[str, str]] = None
    default_on_null: Optional[Any] = None

class SourceColumnMapping(BaseModel):
    source_column: Optional[str] = None # None if extracted from filename
    target_column: str
    transformation: ColumnTransformationSpec

class SourceExtractionSpec(BaseModel):
    source_workbook_pattern: str # Exact filename or glob (e.g. "sales_*.xlsx")
    source_sheet_name: Optional[str] = None # Sheet name or None for all
    header_row_index: int = 0
    data_start_row_index: int = 1
    data_end_row_index: Optional[int] = None
    column_mappings: List[SourceColumnMapping]

class TargetColumnSpec(BaseModel):
    name: str
    data_type: Literal["INTEGER", "BIGINT", "FLOAT", "VARCHAR", "TEXT", "BOOLEAN", "DATE", "TIMESTAMP", "DECIMAL(12,2)"]
    is_primary_key: bool = False
    is_nullable: bool = True
    foreign_key_target: Optional[str] = None # e.g. "customers.customer_id"
    description: str

class TargetTableSpec(BaseModel):
    table_name: str
    description: str
    target_columns: List[TargetColumnSpec]
    primary_key: List[str]
    foreign_keys: List[Dict[str, str]] = Field(default_factory=list)
    sources: List[SourceExtractionSpec]
    deduplication_keys: Optional[List[str]] = None
    conflict_strategy: Literal["KEEP_FIRST", "KEEP_LAST", "MERGE_NON_NULL", "FAIL_ON_CONFLICT"] = "KEEP_FIRST"

class DeclarativeMappingPlan(BaseModel):
    plan_id: str
    plan_name: str
    reasoning_summary: str
    target_tables: List[TargetTableSpec]
```
