# Scopes

Runtime scopes are capability boundaries for local API and MCP calls.

Current public scopes include:

- `actions:read`
- `actions:run`
- `runs:read`
- `connectors:read`
- `approvals:decide`

Credential scopes are separate from API scopes. A tool must not load credentials outside its declared credential scope, even when the caller has `actions:run`.
