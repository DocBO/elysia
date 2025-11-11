# Proposal: Orchestrator MCP Server Spec (flexible, versionable)

Goal
- Capture a flexible MCP-facing specification for a Lake Orchestrator service (S3-compatible data-lake on Backblaze) that Elysia can call via MCP.
- Keep states and policies open/extensible; define capabilities and resource shapes without forcing closed enums.

Context
- Elysia will stay an agent/orchestrator for retrieval and presentation. Heavy data ops (staging, ingest, policy scheduling, cross-store sync) belong in a separate Orchestrator.
- MCP is the integration surface so Elysia (and others) can invoke high-level operations.

Scope (initial)
- Define MCP tools for: capabilities discovery, datasets CRUD, policy attach/remove, S3 object ops (presigned upload/list/head), ingestion/validation jobs, and job progress.
- Define resource shapes (Dataset, Job) with core fields + open metadata.
- Define open state and policy models; clients must tolerate unknown states/policies.

Assumptions
- Transport can be stdio or websocket JSON-RPC; auth handled outside the tool payloads.
- IDs default to UUIDv4; timestamps ISO-8601.

Open Questions
- Event streaming (subscribe) timeline? (Marked optional.)
- Policy catalog exposure (capabilities or a dedicated endpoint)?
- Idempotency keys: standard name and retention window?
