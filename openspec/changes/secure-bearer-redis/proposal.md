# Proposal: Secure all connections with Bearer + Redis

Goal
- Add a minimal Bearer token check for every REST and WebSocket connection. Tokens are UUIDs, validated against a local Redis instance.

Context
- Current API has no auth; CORS is wide-open; WS accepts all connections. See `elysia/api/app.py` and `elysia/api/utils/websocket.py`.

Scope (initial)
- Require `Authorization: Bearer <uuid>` on every REST request.
- Require a token on WebSocket (header or query param); verify before accepting the socket.
- Validate token format (UUID) and existence in Redis.
- Return 401 on missing/invalid tokens, with no sensitive details.

Assumptions
- A local Redis service is available (default `redis://localhost:6379/0`).
- Tokens are stored server-side in Redis (simple key existence or a set membership).
- No multi-tenant or role-based rules in v1; allow/deny only.

Out of scope (v1)
- Token issuance/rotation endpoints.
- Fine-grained RBAC or per-route scopes.
- Rate limiting, audit logs, or IP allowlists.

Open Questions
- Token transport for WS: header only, or also allow `?token=`? (Default both.)
- Redis schema: key-per-token (e.g., `elysia:auth:token:<uuid>`) vs a set (e.g., `elysia:auth:tokens`).
- Fail-closed behavior if Redis unavailable? (Default: fail closed with 503.)
