# Replay

Replay creates a new run linked to the failed source run. Replay lineage must be visible in the audit journal.

Replay must not depend on transient streams. The durable event history and replay input must be enough to explain what happened.
