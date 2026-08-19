# ADR-0003: Polars Sample Dry-Run Validation for Mapping Plan Mutations

## Context

During the Review Session, users modify Column Mappings, data types, transformation operations, and regex patterns. Validating only Pydantic schema structure allows runtime transformation bugs (e.g. invalid date formats, malformed regex capture groups, or impossible type casts) to go unnoticed until the full Migration Job runs on hundreds of thousands of rows.

## Decision

The `PUT /api/sessions/{session_id}/plan` endpoint performs a dual-layer validation:
1. Pydantic structural validation against the `TargetSchemaAndMappingPlan` specification.
2. A deterministic Polars dry-run executing the requested transformation expressions against a cached 50-row sample slice from each source Sheet.

If any transformation fails on the sample, the endpoint returns a `422 Unprocessable Entity` with exact field, column, and sample value failure diagnostics.

## Considered Options

1. **Pydantic Schema Validation Only**: Fast, but defers transformation logic errors to the full migration execution phase.
2. **Full Dataset Dry-Run**: Guarantees complete safety, but introduces unacceptable interactive UI latency on large Excel files.
3. **50-Row Sample Dry-Run (Chosen)**: Sub-50ms execution latency providing immediate feedback and sample previews in the UI.
