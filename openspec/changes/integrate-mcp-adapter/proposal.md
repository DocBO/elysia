# Proposal: MCP support for the agent

Goal
- Enable Elysia’s decision tree to call Model Context Protocol (MCP) tools via a minimal adapter Tool, without refactoring existing tool orchestration or API contracts.

Context
- Current tools are Python-native (`elysia.objects.Tool`) and orchestrated by the decision tree. No MCP client exists in the repo.
- Weaviate integration, API payloads, and UI contracts rely on existing Return types (Text/Retrieval/Result).

Scope (initial)
- Add an optional MCP Adapter Tool that:
  - establishes a connection to an MCP server,
  - lists/inspects remote tools,
  - invokes a selected tool with typed args,
  - maps streamed/final MCP outputs to existing Return types.
- Do not change existing REST/WS contracts; expose MCP capability via the standard tool mechanism.

Assumptions
- MCP client library available to import, or we implement a minimal JSON‑RPC wrapper (stdio/websocket) in-process.
- Secrets for MCP auth are handled like other API keys (never exposed to clients; stored via existing encryption util when persisted).

Open Questions
- Preferred transport (stdio vs websocket) and server discovery flow?
- Do we need a router to surface MCP tool metadata under `/tools/available`, or keep metadata discovery inside the agent only for v1?
- Any provider‑specific auth headers or token rotation rules we must follow?
