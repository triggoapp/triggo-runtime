# Triggo Runtime

**Developer Preview runtime artifacts for running approved workflows through REST and MCP.**

Triggo Runtime is the execution layer between an AI agent and the systems it touches. The agent gets a narrow tool surface. Operators get scoped API keys, dry-run mode, idempotency keys, approval decisions, replay lineage, redacted logs, and an audit journal.

This repository is the public artifact repo: contracts, docs, examples, workflow fixtures, Docker Compose, and issue templates. The runtime image is private-source and governed by [EULA.md](./EULA.md). The documentation, examples, and workflow files are MIT-licensed through [LICENSE](./LICENSE).

Telemetry is off by default. No CIS connector implementations are included in this Developer Preview artifact.

## Trust Model

- Public REST contract: [openapi/runtime-v1.yaml](./openapi/runtime-v1.yaml)
- Public MCP manifest: [mcp/manifest.json](./mcp/manifest.json)
- Public scopes and errors: [contracts/scopes.json](./contracts/scopes.json), [contracts/errors.json](./contracts/errors.json)
- Proprietary runtime image: `ghcr.io/triggoapp/triggo-runtime:0.1.0`
- No managed-service resale, white-labeling, or competing-runtime redistribution under the image EULA

Do not treat this artifact as production HA self-hosting. It is for local validation, MCP/API tool building, and safety-contract inspection.

## Quick Start

```bash
cp .env.example .env
docker compose up
```

REST listens on http://localhost:8090. MCP uses the same local runtime API key.

## Local Examples

No-secret workflow examples live in [examples/workflows](./examples/workflows):

- HTTP echo
- Approved workflow as MCP tool
- Dry-run before write
- Approval-gated write
- Replay failed run

MCP client examples:

- [Claude Desktop](./examples/claude-desktop.json)
- [Cursor](./examples/cursor-mcp.json)

## Safety Contracts

The Developer Preview surface is intentionally small:

- Scoped credentials: tools can only request credentials allowed by policy.
- Dry-run: write-capable calls can be evaluated without connector side effects.
- Idempotency: repeated keys with different request hashes are conflicts.
- Approval: gated writes pause before connector execution.
- Replay: replayed runs are linked to their source run.
- Audit journal: public run state is reconstructable from durable events.
- Egress: private networks and denied hosts are blocked by policy.
- Redaction: bearer tokens, API keys, emails, and sensitive fields are removed before persistence.

## Support

Developer Preview issues are handled best-effort. There is no support SLA.

Useful reports include the runtime image tag, MCP client, sanitized request/response payloads, and the no-secret workflow that reproduces the issue.
