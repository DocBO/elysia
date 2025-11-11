# Design: MCP Adapter Tool

Approach
- Add a new Tool that bridges MCP servers to Elysia tools. Keep it optional and isolated.
- Transport: start with stdio JSON‑RPC to a locally spawned MCP client process or a provided command; later add WS if needed.

Components
- MCPClientManager: manages a long‑lived MCP session (connect/start, send request idempotently, handle responses, graceful close).
- MCPAdapterTool: a Tool that
  - can `is_tool_available()` based on MCP config presence,
  - exposes inputs: `server`, `tool_name`, `args` (dict JSON), optional `timeout`.
  - in `__call__`, ensures a session, runs the MCP tool, yields Text/Result payloads.

Mapping
- Inputs: Elysia `inputs` → MCP `runTool` params.
- Outputs: prefer structured payloads → Text for v1; later add table/document mapping when schemas available.
- Errors: surface as Elysia Error/Status with concise messages.

Security
- Store MCP credentials in Settings.API_KEYS‑style fields; reuse encryption util for persistence.
- Do not echo secrets in logs or responses.

Non‑Goals (v1)
- No automatic enumeration to UI unless requested; agent can still call MCP by name.
- No multi‑tenant secret broker; rely on existing config mechanisms.

Risks
- Transport/compat quirks across MCP servers. We mitigate by stubbing a narrow, minimal contract and documenting it.
