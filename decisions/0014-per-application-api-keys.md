# ADR-0014: Identify applications by separate Bible-API keys

- Status: Accepted
- Date: 2026-09-26
- Ticket: ClickUp 123pfqn0jzc

## Context

The released Bible Garden iOS app and unreleased Lampada app shared one
Bible-API key. Request statistics therefore could not distinguish them. The
released iOS key value cannot change without breaking installed clients.

## Decision

Bible-API requires three explicit keys at every startup. The value formerly
named `API_KEY` is renamed byte-for-byte to `BIBLE_GARDEN_API_KEY` and identifies
`bible-garden`. New distinct `LAMPADA_API_KEY` and `OPS_API_KEY` values identify
`lampada` and `ops`. The latter is used by monitoring, operator checks and
live evaluations. `API_KEY` is rejected at startup. Missing, blank, padded or
duplicate values are errors; the two new keys require at least 32 characters.
The request key and each configured key are hashed with SHA-256. Every
fixed-length digest is checked with `hmac.compare_digest`; matching the first
candidate does not skip the others.

Header authentication and audio query authentication both set the application
on request state. A non-empty audio query key keeps precedence over the header:
an invalid query key is denied even when the header is valid. API routes without
a resolved application are not recorded: authentication failures, redirects
issued before authentication (such as trailing-slash 307), unattributed 4xx
validation responses, audio OPTIONS and health checks. A successful response
without application is logged as an error and not recorded; the client still
receives its response.

`cep_public.api_requests` records the application; daily endpoint aggregates
are keyed by date, endpoint and application. Overall `all` rows exist for each
endpoint and for `_total_`, alongside per-application rows. Their distinct-client
counts span applications without double counting shared pseudonyms.
Rows recorded before the change and requests written by the old Bible-API
between schema migration and service replacement are `unknown`. Both columns
retain `DEFAULT 'unknown'` so the old writer can keep inserting during that
interval. Older rows are never retroactively attributed to Bible Garden. The
new writer always supplies an explicit application. The dashboard shows
request counts by application and filters
retained raw requests by application.

## Consequences

Dashboard-API's schema migration must run before the new Bible-API writer.
Bible-API must accept all three identities before a Lampada build containing
its new key is distributed. Existing Bible Garden installations continue to
use the unchanged value. Client keys are embedded in released applications, so
they identify clients for statistics but are not confidential user credentials.
The current AI rate limits remain keyed by client IP pseudonym, not application.
