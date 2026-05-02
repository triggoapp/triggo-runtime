# Security

Runtime safety claims must be backed by tests before publication:

- Scoped credentials: tools can only request credentials allowed by policy.
- Dry-run: write-capable calls do not execute connector side effects.
- Idempotency: repeated keys with different request hashes are conflicts.
- Approval: gated writes pause before connector execution.
- Replay: replayed runs are linked to their source run.
- Audit journal: events are append-only and reconstructable.
- Egress: private networks and denied hosts are blocked by policy.
- Secrets: bearer tokens, API keys, emails, and configured sensitive fields are redacted before persistence.

Telemetry must be opt-in for public runtime artifacts. If a later build adds telemetry, docs and config must say exactly what is collected and how to disable it.
