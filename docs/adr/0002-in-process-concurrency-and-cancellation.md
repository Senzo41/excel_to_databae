# ADR-0002: In-Process ThreadPool Concurrency with Cooperative Cancellation

## Context

The backend handles a mix of CPU-heavy operations (Rust-based Calamine spreadsheet parsing, Polars DataFrame transformations, and target sink writing) and I/O-heavy operations (async HTTP, LLM structured synthesis API calls, and SSE event streaming). The system must run efficiently in single-machine/embedded deployments without external infrastructure dependencies like Redis.

## Decision

We use FastAPI native asynchronous request handling combined with a bounded `ThreadPoolExecutor` (via `asyncio.to_thread`) for CPU-bound Polars and Calamine operations. Cancellation is implemented cooperatively using thread-safe `asyncio.Event` tokens checked at granular execution boundaries (between table and batch transformations).

## Considered Options

1. **Distributed Queue (Celery / ARQ + Redis)**: Rejected for initial architecture due to operational overhead and external dependencies for embedded/single-container use cases.
2. **Embedded DB Task Queue (SQLite worker loop)**: Rejected due to unnecessary polling latency and synchronization complexity compared to direct in-memory async primitives.
3. **In-Process ThreadPool + Async (Chosen)**: Zero external dependencies, clean async non-blocking event loop, with extensible runner interfaces for future distributed scale-out.
