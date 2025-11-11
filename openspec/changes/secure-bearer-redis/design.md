# Design: Bearer auth with Redis

Overview
- Introduce a minimal, fail-closed Bearer token check for both REST and WS.
- Tokens are UUID strings and must exist in Redis to be accepted.

Components
- Redis client: lazy-initialized singleton from `REDIS_URL`.
- Token store: Redis set `AUTH_REDIS_SET` (default `elysia:auth:tokens`) or key prefix `elysia:auth:token:<uuid>`; v1 uses set membership for O(1) checks.
- REST dependency: parses `Authorization: Bearer <uuid>`, validates UUID and Redis membership; raises HTTP 401 on failure.
- WS gate: extracts token from `Authorization` header (preferred) or `token` query param; validates before `accept()`; closes with code 1008 on failure.

Failure modes
- Missing/invalid/malformed token → 401 (REST) or 1008 (WS).
- Redis error/unavailable → 503 (REST) or 1013 (WS try again later), logged server-side.

Non-goals
- Token management endpoints; rotation; scopes; per-route roles.

Notes
- Keep the implementation small and optional via env: if `AUTH_REQUIRED=false`, bypass validation (default true in prod).
- Health checks can be exempted via config if required by deployment, otherwise protected.
