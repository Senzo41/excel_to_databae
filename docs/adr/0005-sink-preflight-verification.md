# ADR-0005: Dedicated Pre-Flight Target Sink Verification

## Context

Target sinks may be remote relational databases (PostgreSQL, MySQL) requiring network credentials and schema permissions, or local file directories (DuckDB, SQLite, Parquet). If credentials or disk permissions fail after the user triggers a long-running migration, system resources are wasted and the user experience suffers.

## Decision

We expose a dedicated endpoint `POST /api/sessions/{session_id}/test-sink` that tests connectivity, authentication, schema creation permissions, and table collision checks prior to initiating the `POST /api/sessions/{session_id}/migrate` call.

## Considered Options

1. **Verify at Migration Trigger Time Only**: Fails late after initiating worker allocation.
2. **Client-Side Regex / Syntax Validation Only**: Cannot detect bad credentials, network timeouts, or insufficient database grants.
3. **Dedicated Pre-Flight Endpoint (Chosen)**: Clear separation of configuration verification from execution triggering.
