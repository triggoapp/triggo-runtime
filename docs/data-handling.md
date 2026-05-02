# Data Handling

Runtime runs can contain connector inputs, outputs, errors, and journal events. Public runtime artifacts must treat those records as sensitive.

Rules:

- Redact secrets before storing logs or journal payloads.
- Keep telemetry off by default or explicitly opt-in.
- Do not include hosted SaaS databases, seed data, customer data, or source maps in public runtime images.
- Keep local examples no-secret by default.
