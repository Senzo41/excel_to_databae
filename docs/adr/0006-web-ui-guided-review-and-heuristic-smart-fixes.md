# ADR-0006: Guided Review Session Flow with Dual-List Entity Mapping and Heuristic Smart-Fixes

## Context

The web application must allow non-technical business users to ingest heterogeneous, messy multi-sheet Excel files, review AI-synthesized target schemas, inspect and adjust column mappings, resolve data quality anomalies (e.g. casing variations, percentage scaling errors, European comma decimal formats), and monitor migration execution.

## Decision

We adopt:
1. **Guided 4-Stage Wizard Flow with Non-Linear Navigation**: `1. Ingest & Extract` → `2. AI Synthesis & Profiling` → `3. Review & Map` → `4. Target Sink & Migrate`. Users can freely jump back to review mappings without resetting workspace state.
2. **Dual-List Entity Mapping Layout**: Left column presents source sheets, detected data regions, and candidate fields; Right column presents target schema entities, data types, and inline transformation builder cards.
3. **1-Click Heuristic "Smart Fix" Actions**: Data quality anomalies detected during statistical profiling (e.g. casing/typos like `"Spanen"` vs `"spanen"`, integer percentages `78.0` vs `0.78`, European comma decimals `"12,50"`) surface as contextual warning badges with 1-click transformation fixes (`SCALE_NUMERIC`, `NORMALIZE_CATEGORICAL`, `PARSE_EU_DECIMAL`).
4. **Instant 3-Row Sample Value Diff Pills & Bottom Dry-Run Drawer**: Live before/after transformation feedback rendered directly inside transformation cards, backed by a persistent 50-row transformed target table drawer.
5. **Decoupled AI Reasoning Accordion**: Live typewriter streaming of LLM reasoning summary tokens during synthesis, collapsing into explanation badges attached to AI-suggested mappings.

## Considered Options

1. **Visual Node-and-Wire Graph Canvas (React Flow)**: High visual novelty but degrades into unusable spaghetti diagrams when mapping 10+ sheets and 50+ columns for non-technical users.
2. **Spreadsheet-Only Matrix Grid**: Familiar to Excel users but creates high visual noise when configuring complex cross-sheet unions, filename metadata extraction, and multi-step transformation rules.
3. **Guided Dual-List with Smart-Fix Cards (Chosen)**: Optimal information density, clear mental model for non-technical users, instant feedback on transformations, and high accessibility.
