## ADDED Requirements

Requirement: Provide an MCP Adapter Tool to invoke MCP tools from the decision tree without changing existing API payload contracts.

#### Scenario: Agent invokes an MCP tool by name
- Given MCP connectivity is configured (server endpoint and auth set in settings/config)
- When the agent selects the MCP Adapter Tool with inputs `{ server, tool_name, args }`
- Then the tool connects (or reuses a session) and calls `runTool(tool_name, args)` on the MCP server
- And the final output is returned as an Elysia Text result payload (frontend_type `text`), with `error` empty on success
- And any transport or tool error is returned as an Elysia Error payload with a concise message

Requirement: Expose minimal tool metadata for planner use

#### Scenario: Listing remote MCP tools (optional)
- Given the adapter is configured
- When the agent (or an internal planner) requests MCP tool metadata
- Then the adapter can list tool names and basic schemas
- And the data is only used internally by the planner in v1 (no public REST exposure required)

Requirement: Secure handling of MCP credentials

#### Scenario: Persisting MCP credentials
- Given users save configs via existing endpoints
- When MCP credentials are present in settings
- Then they are encrypted using the existing Fernet mechanism and never included in API responses

Requirement: Backwards compatibility

#### Scenario: No MCP config present
- Given no MCP settings are configured
- When the system runs standard tool flows
- Then the MCP Adapter Tool is unavailable and normal Weaviate flows remain unaffected

## MODIFIED Requirements

Requirement: Tool discovery endpoint can include adapter-provided tools (future)

#### Scenario: Including MCP tools in /tools/available (optional later)
- This repo may add MCP tools to `/tools/available` once stability is proven, but this is out of v1 scope.

## REMOVED Requirements

- None.
