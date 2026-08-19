## Destination

A complete technical architecture specification, domain model (CONTEXT.md), and executable implementation roadmap for an enterprise web application (React + Vite / FastAPI + Polars) that allows non-technical teams to upload batches of complex Excel files, automatically infers and proposes unified target schemas and mapping plans via LLM synthesis, and migrates data into embedded formats (Parquet, DuckDB, SQLite) and SQL databases.

## Notes

- Tech stack: React + Vite (Frontend), FastAPI + Polars (Backend / Execution Engine).
- Primary Target Sinks: Parquet (directory), DuckDB, SQLite, PostgreSQL, MySQL.
- Domain glossary: `CONTEXT.md`.
- Skills to consult: `/grilling`, `/domain-modeling`, `/research`, `/prototype`.

## Decisions so far

<!-- the index — one line per closed ticket: enough to judge relevance, then zoom the link for the detail the ticket holds -->

- [Research Pre-Extraction Heuristics for Messy Excel Layouts](file:///Users/olefeineisen/spasssprojekte/excel_to_database/.scratch/excel-to-database/issues/01-research-pre-extraction-heuristics.md) — Extract 2D grid with `python-calamine` and stream XML merged cells (<5ms); score header candidate rows using type transition, uniqueness, and lookahead homogeneity ($w_4=0.30$); flatten multi-level headers and feed into Polars.
- [Research Schema Profiling & Anonymized Sampling Representation for LLM Prompts](file:///Users/olefeineisen/spasssprojekte/excel_to_database/.scratch/excel-to-database/issues/02-research-schema-profiling-and-sampling.md) — Pre-compute column distributions, regex skeletons, and Jaccard inclusion hints in Polars; serialize as Compact Markdown-KV (72% token savings) and enforce Pydantic structured output for mapping plans.
- [Define the Declarative Mapping Plan Schema Specification](file:///Users/olefeineisen/spasssprojekte/excel_to_database/.scratch/excel-to-database/issues/03-define-declarative-mapping-plan-schema.md) — Target-centric Pydantic model with structured atomic transformation operations (`DIRECT`, `CAST`, `CLEAN_STRING`, `PARSE_DATE`, `REGEX_EXTRACT`, `MAP_VALUES`, `FALLBACK_VALUE`, `EXTRACT_FROM_FILENAME`), conflict strategies, and decoupled target sink configs.

## Not yet specified

- **Multi-tenant / Team Sharing & Access Control**: How schemas, mapping configs, and uploaded raw files are shared or isolated across multiple user accounts if deployed centrally.
- **Incremental / Scheduled Syncs**: Whether and how recurring Excel drops (e.g. monthly updated spreadsheets) can re-run existing mapping plans with change detection.
- **Enterprise SSO & Compliance Logging**: RBAC integration and compliance logging for data migrations in enterprise environments.

## Out of scope

- Direct SaaS integrations (Salesforce, HubSpot, SAP API connectors) — the scope is strictly file-based spreadsheet migration to database/parquet.
- Full BI/Dashboard visualization tool — the system outputs clean structured storage (Parquet/SQLite/DB), AI tools and BI connect to the resulting sinks.
