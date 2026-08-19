# ADR-0001: SSE and REST Architecture for Real-Time Streaming

## Context

The system requires real-time progress streaming across long-running stages: file extraction with Calamine, Polars statistical profiling, LLM schema synthesis, and Polars migration execution into target sinks. At the same time, users interactively inspect and adjust Mapping Plans during the Review Session and trigger or abort jobs.

## Decision

We adopt Server-Sent Events (SSE) for unidirectional telemetry and status streaming from FastAPI to the React client (`GET /api/sessions/{session_id}/events`), combined with standard REST endpoints for state mutations (file uploads, plan updates, migration triggering, cancellations).

## Considered Options

1. **Full Bidirectional WebSockets**: Rejected due to unnecessary stateful framing complexity, firewall/proxy traversal friction, and loss of standard REST/OpenAPI semantics for plan validation.
2. **REST Polling**: Rejected due to high latency, redundant request overhead, and lack of smooth real-time progress feedback.
3. **SSE + REST (Chosen)**: Lightweight, native HTTP/2 stream with automatic browser reconnection, preserving clean HTTP status codes and Pydantic validation for mutations.
