# Idempotency

Idempotency keys are scoped by workspace and request hash.

Expected behavior:

- Same key and same request returns the same accepted run/result.
- Same key with a different slug, input, dry-run flag, or workspace is a conflict or a distinct request according to the public contract.
- Completion writes must happen through host callbacks or persistence ports so retries cannot create duplicate side effects silently.
