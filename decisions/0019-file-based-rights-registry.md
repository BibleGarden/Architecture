# ADR-0019: Versioned rights registry in Bible-API

- Status: Accepted
- Date: 2026-10-18
- Approval: Maria, 2026-10-18
- Tickets: ClickUp 123pfqn3c69, 123pfqn3c6g, 123pfqn3c6p

## Context
Rights notices change rarely, when permissions arrive, and need reviewable evidence.
Text permissions do not establish rights to a separate audio recording.
## Decision
Bible-API owns one versioned YAML rights registry, validates it before startup,
and serves it at `/api/rights` with ordinary public-catalog API-key authentication.
Site commits a derived API response and generates EN/RU/UK rights pages offline.
New permissions change one registry entry through a PR; git history is the audit trail.
Rights metadata is not Bible content; the local-admin content rule does not apply.
## Alternatives and consequences
Reject a DB table plus admin editor: migrations and UI exceed this small catalog's needs.
Reject a DB table plus admin API: database publication adds complexity without useful editing benefits.
No migrations, export or Dashboard changes; Site snapshots must be refreshed after registry changes.
