## ADDED Requirements

Requirement: Enforce Bearer token on all REST endpoints

#### Scenario: Missing token
- Given a REST request without an `Authorization` header
- When it reaches the API
- Then the server responds `401 Unauthorized` with a generic message

#### Scenario: Malformed token
- Given a REST request with `Authorization` that is not `Bearer <uuid>`
- Then the server responds `401 Unauthorized`

#### Scenario: Unknown token
- Given a REST request with a valid UUID token not present in Redis
- Then the server responds `401 Unauthorized`

#### Scenario: Accepted token
- Given a REST request with a UUID token present in Redis
- Then the server processes the request normally (2xx on success)

Requirement: Enforce Bearer token on WebSocket connections

#### Scenario: Missing or invalid token
- Given a WS connection attempt without a valid Bearer token (header or `?token=`)
- Then the server refuses the connection with close code 1008 (policy violation)

#### Scenario: Accepted token
- Given a WS connection attempt with a UUID token present in Redis
- Then the server accepts the socket and processes messages as usual

Requirement: Redis-backed token validation

#### Scenario: Redis membership check
- Given a token `t`
- When validating, the server checks membership of `t` in the configured Redis set (default `elysia:auth:tokens`)
- And returns authorized only on membership

Requirement: Configurability and failure behavior

#### Scenario: Config
- The server reads `REDIS_URL` (default `redis://localhost:6379/0`) and `AUTH_REDIS_SET` (default `elysia:auth:tokens`)
- The server may read `AUTH_REQUIRED` (default `true`); when `false`, validation is bypassed

#### Scenario: Redis unavailable
- If Redis is unavailable while `AUTH_REQUIRED=true`, REST returns `503 Service Unavailable` and WS closes with 1013 (try again later)

## MODIFIED Requirements

Requirement: Documentation for auth

#### Scenario: API docs updated
- The project docs include a concise section explaining Bearer token usage for REST and WS, env vars, and expected error codes

## REMOVED Requirements

- None.
