# ADR-0004: SSE Event Reconnection Buffering and LLM Reasoning Streaming

## Context

Network interruptions or page reloads during asynchronous profiling, LLM schema synthesis, or Polars migration jobs can disconnect client event listeners. Additionally, LLM analysis takes 5–20 seconds, during which users need feedback on what the agent is reasoning about without forcing the client to assemble complex partial JSON ASTs.

## Decision

1. **Ring Buffer Event Replay**: The backend maintains an in-memory ring buffer of the latest 200 `StreamingEvent`s per `Session Workspace`. On reconnect with standard `Last-Event-ID`, missed events are replayed before streaming live events.
2. **Decoupled LLM Streaming**: The backend streams the LLM's chain-of-thought `reasoning_summary` tokens as `LLM_TOKEN` SSE events in real time. Once the LLM finishes, the backend validates the full structured JSON into a Pydantic `TargetSchemaAndMappingPlan` model and emits a single `PLAN_READY` event.

## Considered Options

1. **Streaming Partial JSON Deltas**: High frontend parsing complexity with risk of UI desynchronization on broken frames.
2. **Current State Snapshot Only on Reconnect**: Simple, but drops log history and stage transitions if reconnecting mid-migration.
3. **Ring Buffer Replay + Token Streaming (Chosen)**: Reliable fault recovery adhering to SSE standards with clear UX feedback.
