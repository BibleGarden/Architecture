# ADR-0005: Server-controlled AI prefetch

- Status: Accepted
- Date: 2026-09-14

## Context

Lampada Mobile generates questions and selects scripture before the user asks
for them. This reduces waiting but spends provider capacity on unused results.
Operators need to control this work without releasing another mobile build.

## Decision

`POST /api/ai/question` and `POST /api/ai/scripture` accept an optional strict
JSON boolean `prefetch`, defaulting to `false`. Mobile marks every speculative
call with `true`; requests made to fulfil a user action use `false` or omit it.

Before provider calls, corpus work, or the endpoint's normal quota reservation,
the server applies separate question and scripture prefetch policies: an enable
switch and global/per-client quotas in a rolling 60-second window. Both switches
default to disabled; each endpoint defaults to two requests globally and one per
client per window. Explicit operational defaults are allowed; malformed or blank
settings and nonpositive quotas reject startup.
Enabling either prefetch policy requires `AI_ENABLED=true` and its normal valid
provider and client-HMAC configuration; enabling prefetch while AI is disabled
rejects startup.

Policy denial returns HTTP 429 with `{"detail":"prefetch_disabled"}` and no
`Retry-After`, or `{"detail":"prefetch_limit_exceeded"}` with `Retry-After`
in seconds. Denied prefetch consumes no normal quota; admitted prefetch remains
subject to the normal endpoint quota and processing rules.

Mobile remembers a denial for that context, without an automatic retry, visible
error, or replacement speculative request. A subsequent user action can make a
normal request. Consent requirements remain unchanged.

The limiter uses HMAC-pseudonymized client IPs and process-local counters,
matching the single-worker API design. Counters reset on restart. Enabling
multiple workers would require a shared limiter to retain aggregate limits.
Operators set switches and quotas manually; there is no spending-budget or
load-based automation. Models and generation pipelines remain governed by
[ADR-0002](0002-ai-model-provider-policy.md).

## Consequences

Deploy the API contract before the mobile build. Older clients omit the marker,
so their speculative requests remain indistinguishable from normal requests.
The policy controls cooperating clients; it is not an abuse-prevention boundary.
Disabled prefetch trades background API cost for waiting after a user action.
Configuration and rollout procedures live in `Deploy`.
