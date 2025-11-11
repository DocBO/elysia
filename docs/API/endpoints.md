# API Endpoints

This page explains how the FastAPI app organizes routes, path patterns, and payloads. Endpoints are grouped by feature with consistent request/response shapes and lightweight error handling.

## Overview
- Base app: `elysia.api.app:app` (served via `elysia start ...`).
- Health: `GET /api/health` → `{ "status": "healthy" }`.
- Namespacing: Routers mount under stable prefixes:
  - `/init` (user/tree bootstrap)
  - `/ws` (WebSockets: pipeline, preprocessing)
  - `/collections` (Weaviate collections + metadata)
  - `/user/config` (user-level config & frontend prefs)
  - `/tree/config` (per-conversation config)
  - `/feedback` (user feedback storage)
  - `/tools` (list, add/remove tools/branches)
  - `/db` (save/load/delete trees)
  - `/util` (NER, titles, follow-ups, debug)

## REST Conventions
- Path params: most routes require `user_id`; some also need `conversation_id` or `collection_name`.
- Body: JSON models defined in `elysia.api.api_types` (e.g., `InitialiseTreeData`).
- Success: responses include `{ ..., "error": "" }`. On handled failures, `error` is a message; status is usually 200 or a specific code for timeouts.

## Key REST Endpoints (by router)
- `/init`
  - `POST /init/user/{user_id}`: create/get user; returns config + frontend_config.
  - `POST /init/tree/{user_id}/{conversation_id}`: create/get tree; body `{ "low_memory": bool }`.
- `/collections`
  - `GET /collections/{user_id}/list`: list collections with counts/vectorizers/processed flags.
  - `POST /collections/{user_id}/view/{collection_name}`: paginated search/sort/filter.
  - `GET /collections/{user_id}/get_object/{collection_name}/{uuid}`: fetch one object.
  - `GET|PATCH|DELETE /collections/{user_id}/metadata/{collection_name}`: read/update/delete preprocessed metadata; `DELETE /metadata/delete/all` for bulk.
- `/user/config`
  - `GET /user/config/{user_id}`: current in-memory config + frontend_config.
  - `PATCH /user/config/frontend_config/{user_id}`: update frontend save location/flags.
  - `POST /user/config/{user_id}/new`: reset user defaults. (Additional save/load/list helpers exist.)
- `/tree/config`
  - `GET /tree/config/{user_id}/{conversation_id}`: fetch active tree config.
  - `POST /tree/config/{user_id}/{conversation_id}`: patch tree settings/style/description/goals.
  - `POST /tree/config/{user_id}/{conversation_id}/new`: reset tree defaults. (Load from saved config also available.)
- `/tools`
  - `GET /tools/available`: discover custom Tool metadata.
  - `POST /tools/{user_id}/add|remove`: mutate tools in all trees for the user.
  - `POST /tools/{user_id}/add_branch|remove_branch`: manage branches.
- `/db`
  - `GET /db/{user_id}/saved_trees`: list stored conversations.
  - `GET /db/{user_id}/load_tree/{conversation_id}`; `POST /save_tree/...`; `DELETE /delete_tree/...`.
- `/feedback`
  - `POST /feedback/add|remove`; `GET /feedback/metadata/{user_id}`.
- `/util`
  - `POST /util/ner` (spaCy spans), `POST /util/title`, `POST /util/follow_up_suggestions`, `POST /util/debug`.

Example:
```bash
# List collections for a user
curl -s localhost:8000/collections/alice/list | jq .
```

## WebSockets
- Query pipeline: `WS /ws/query`
  - Send JSON: `{ "user_id": "alice", "conversation_id": "conv1", "query": "Find X", "query_id": "<uuid>", "route": "", "collection_names": ["my_collection"] }`
  - Receive stream of messages (types include `ner`, tool outputs, `title`, final `completed`).
- Preprocessing: `WS /ws/process_collection`
  - Send: `{ "user_id": "alice", "collection_name": "my_collection" }`
  - Receive progress payloads with `progress` and status.

Notes: Some utilities return `401/408` on timeouts; most endpoints encode errors as `{ "error": "..." }` for uniform client handling.
