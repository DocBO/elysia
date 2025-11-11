# Design: Ingestion workflow + Embedding Manager

Ingestion API
- REST `POST /collections/{user_id}/upload` (multipart)
  - Fields: `file`, `collection_name`, `upsert` (bool), `batch_size` (int, default 512), optional vectorizer config (`vectorizer`, `model`, `named_vectors`), `id_property`.
  - Response: `{ upload_id, accepted, error }`. Parsing/insert runs inline; progress is streamed via WS.
- WS `WS /ws/upload_progress`
  - Client sends `{ user_id, upload_id }`. Server emits `{ progress:[0..1], inserted, updated, errors:[...], status }`.
- Optional finalize: `POST /collections/{user_id}/finalize/{collection_name}?preprocess=true`.

Flow
1) Validate Bearer token (REST + WS).
2) Parse CSV/JSONL in a streaming fashion; infer schema (types, nullable, arrays).
3) Create schema if missing; otherwise respect existing config; support upsert by `id_property`.
4) Batch insert with backpressure; emit progress at intervals; collect row-level errors.
5) Optionally trigger preprocess.

Embedding Manager
- Singleton with lazy init; configured via Settings (provider/model/api_base/keys).
- Interface: `embed(texts: list[str], model: Optional[str]=None, batch: Optional[int]=None) -> list[list[float]]`
- Caching: in-memory LRU keyed by (model, text hash); configurable max size.
- Reuse: shared across tools (e.g., visualisation or MCP tools), preprocessing, and ingestion; thread/async-safe.
- Provider support: wrap LiteLLM or provider SDKs; prefer batch endpoints when available.
- Fallback: if Weaviate handles vectorization server-side, Embedding Manager is unused for that collection.

Failure Modes
- Invalid file/format → 400 with reason; WS emits error and stops.
- Weaviate unavailable → 503; resume later by re-sending same file and `upsert=true`.
- Embedding provider unavailable → skip local embedding, log/emit warning; rely on server-side vectorizer if configured.

Security
- Requires Bearer + Redis auth.
- Enforce max upload size and supported MIME types.
