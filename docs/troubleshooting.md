# Troubleshooting

## Runtime API key rejected

Check RUNTIME_API_KEYS in .env and pass Authorization: Bearer <key>.

## MCP tool missing

Confirm the key has actions:run and connectors:read scopes, then list tools
through REST and MCP to compare schema hashes.

## Image scan failed

The public image scanner blocks hosted SaaS code, private connector
implementations, source maps, secret-like files, and root DB schema leaks.

## No support SLA

Developer Preview issues are handled best-effort.
