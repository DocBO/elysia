# Proposal: Dataset Upload + Universal Embedding Manager

Goal
- Add a secure ingestion workflow to create/update Weaviate collections from user-provided files (CSV/JSONL v1) with streamed progress.
- Introduce a project-wide Embedding Manager to avoid spawning new embedders per request and provide a single, reusable embedding interface across tools and ingestion.

Motivation
- Current backend can query/aggregate/visualise but cannot upload/ingest new datasets. Closing this gap enables end-to-end use.
- Embedding usage appears in multiple places (future MCP tools, preprocessing, potential client-side chunking). A single manager avoids duplicated setup and improves latency.

Scope (initial)
- REST: `POST /collections/{user_id}/upload` (multipart/form-data). Accept CSV/JSONL, options for collection name, upsert, vectorizer settings, batch size.
- WS: `WS /ws/upload_progress` to stream server-side progress for a given `upload_id`.
- Optional: `POST /collections/{user_id}/finalize/{collection_name}?preprocess=true` to trigger preprocessing after ingest.
- Embedding Manager: singleton lifecycle, provider-agnostic, batch API, in-memory LRU cache, settings-driven provider/model.

Non-goals (v1)
- Parquet/PDF ingestion, auto-chunking on upload, complex transforms.
- Multi-tenant quota controls or background job queue; keep simple, synchronous ingestion with progress streaming.

Security
- Bearer + Redis required on REST and WS (integrates with the `secure-bearer-redis` change).

Open Questions
- Column mapping UI vs auto-infer only for v1?
- Should ingestion be resumable, or is idempotent upsert sufficient?
