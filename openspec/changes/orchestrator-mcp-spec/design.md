# Design: Lake Orchestrator MCP (open, extensible)

Principles
- Capability-based surface with versioning.
- Open state/policy models (strings + opaque params) with stable required fields.
- Idempotent, retry-safe mutations (optional idempotency_key).

Core Resources
- Dataset: { id, name, version?, uri, bytes?, created_at, updated_at, state, tags[], labels{}, policies[], metadata{} }
- Job: { id, kind, dataset_id, state, progress, counters{inserted,updated,errors}, error?, started_at?, finished_at?, metadata{} }

States (suggested; open-ended)
- Dataset: registered, staged, validating, validated, ingesting, ingested, waiting, running, failed, canceled, …
- Job: queued, running, waiting, succeeded, failed, canceled, …

Policies (open-ended)
- Policy: { name, enabled, params{} } e.g., lazy, busy-beaver, hot-index.

MCP Tools (JSON-RPC)
- Discovery
  - list_capabilities() → { capabilities[], server_version, mcp_version }
- Dataset lifecycle
  - list_datasets(filter?, page?) → { items: Dataset[], next_page? }
  - get_dataset(dataset_id) → Dataset
  - register_dataset({ name, uri, tags?, labels?, metadata? }) → Dataset
  - update_dataset({ dataset_id, name?, tags?, labels?, metadata? }) → Dataset
  - delete_dataset({ dataset_id, force? }) → { deleted: bool }
  - commit_dataset_version({ dataset_id, version, metadata? }) → Dataset
- Policies
  - list_policies(dataset_id) → { policies: Policy[] }
  - set_policy({ dataset_id, policy: Policy }) → { ok: true }
  - remove_policy({ dataset_id, policy_name }) → { ok: true }
- Lake storage (S3)
  - create_presigned_upload({ dataset_id, object_key, content_type?, size? }) → { url, headers?, expires_at }
  - list_objects({ dataset_id, prefix?, page? }) → { items: [{ key, size, etag, last_modified, metadata? }], next_page? }
  - head_object({ dataset_id, object_key }) → { size, etag, last_modified, metadata? }
- Processing/ingestion
  - start_ingest({ dataset_id, target, mapping?, options?, idempotency_key? }) → Job
  - reindex({ dataset_id, target, idempotency_key? }) → Job
  - validate({ dataset_id, validators?, idempotency_key? }) → Job
- Jobs
  - list_jobs({ dataset_id?, kind?, state?, page? }) → { items: Job[], next_page? }
  - job_status({ job_id }) → Job
  - cancel_job({ job_id, reason? }) → { canceled: bool }

Events (optional)
- subscribe({ dataset_id?, kinds? }) → server events { type: "job.updated"|"dataset.updated", data: Job|Dataset }

Versioning & Compatibility
- list_capabilities returns server_version, mcp_version and feature flags (e.g., "ingest.weaviate").
- Unknown fields ignored by clients; new states/policies allowed.

Auth & Transport
- MCP transport auth (TLS, proxy) handled outside tool payloads.
- Orchestrator holds all store credentials; clients never receive raw secrets (use presigned URLs).
