# Audit Journal

The runtime journal is append-only. Events must be sufficient to reconstruct run state without relying on transient streams or in-memory state.

Minimum public event claims:

- Run start and terminal state are recorded.
- Step success and failure are recorded.
- Approval pending/rejected states are recorded.
- Safety violations are recorded.
- Replay lineage is recorded.
- Stored payloads are redacted.
