## ADDED Requirements

Requirement: Orchestrator MCP capability discovery

#### Scenario: List capabilities
- Client calls `list_capabilities()`
- Server returns `{ capabilities: [string], server_version: string, mcp_version: string }`

Requirement: Dataset resource operations

#### Scenario: Register dataset
- Client calls `register_dataset({ name, uri, tags?, labels?, metadata? })`
- Server returns `Dataset` with `state` (free-form, suggested: "registered" or "staged")

#### Scenario: Update/delete/list/get datasets
- Server supports `update_dataset`, `delete_dataset`, `list_datasets`, `get_dataset` with paging; unknown fields preserved

Requirement: Policy attachment (open-ended)

#### Scenario: Set/remove policy
- Client calls `set_policy({ dataset_id, policy: { name, enabled, params } })`
- Server records policy; `remove_policy` by name removes it

Requirement: S3 operations via presigned URLs

#### Scenario: Create presigned upload
- Client calls `create_presigned_upload({ dataset_id, object_key, content_type?, size? })`
- Server returns `{ url, headers?, expires_at }` and never exposes raw credentials

Requirement: Processing and ingestion jobs

#### Scenario: Start ingest job
- Client calls `start_ingest({ dataset_id, target, mapping?, options?, idempotency_key? })`
- Server creates `Job { id, kind: "ingest", state: "queued"|"running", progress: 0, metadata? }`
- `job_status({ job_id })` returns progress and counters; `cancel_job` cancels if supported

Requirement: Open state model

#### Scenario: Unknown states tolerated
- The `state` field for Dataset/Job is free-form; clients must not assume closed enums; server may add custom states

Requirement: Versioning and forward compatibility

#### Scenario: Capabilities expose versions
- `list_capabilities` includes `server_version`, `mcp_version`, and feature flags (e.g., "ingest.weaviate")
- Clients ignore unknown fields and tolerate missing optional fields

Requirement: Auth & transport separation

#### Scenario: No secrets in tool payloads
- Orchestrator MCP server never returns raw store credentials; clients use presigned URLs for object access

## MODIFIED Requirements

Requirement: Event subscription (optional)

#### Scenario: Subscribe to updates
- If supported, `subscribe({ dataset_id?, kinds? })` yields `job.updated`/`dataset.updated` events with the same schemas as resource reads

## REMOVED Requirements

- None.
