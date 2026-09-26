# ADR-0015: Use UTC for new production timestamps and daily statistics

- Status: Accepted
- Date: 2026-09-26
- Ticket: ClickUp 123pfqn0k4y

## Context

The 2026-03-06 operational decision set the production host, MySQL and cron
to Europe/Moscow. Application containers run UTC. MySQL `DATETIME` values
have no zone, while dashboard code has interpreted some as UTC. Umami
PostgreSQL was changed independently to UTC on 2026-09-25.

## Decision

The production host, MySQL sessions and logs use UTC. New production MySQL
`DATETIME` values are UTC-naive; API timestamps exposed to clients carry an
explicit UTC `Z`, and people see instants in their browser's local zone.
Production daily statistics use UTC calendar days. The 00:01 production
aggregation runs against that calendar. Human-facing Telegram observation
times may remain in Moscow time when labelled as such.

No existing database values or daily aggregates are rewritten. Values in
`cep_public.api_requests.created_at` and `ai_content_reports.created_at`
before the exact cut-over instant in the production deployment protocol
remain MSK-naive. Older `api_request_daily_stats.date` rows represent
Moscow days. RAG `created_at` is unused metadata copied without conversion
from the local MSK database, including after the cut-over. The local
development host and MySQL remain in Europe/Moscow; Dashboard-API's explicit
local database zone allows its HTTP timestamps to carry UTC instants.

## Consequences

The old raw request rows expire after 14 days, but old content reports do
not. Until a separate policy is chosen, production Dashboard-API treats
all naive MySQL report/request datetimes as UTC when serializing them. Thus
pre-cut-over rows can display three hours late in the dashboard and report
notifications, and old content reports can remain so indefinitely. Daily
series spanning the cut-over combine Moscow
and UTC day boundaries. The dashboard and runbook must state that limit.
Rollback after any new UTC writes records another cut-over; changing the
timezone back cannot make mixed historical values uniform.

This decision supersedes the 2026-03-06 Europe/Moscow production-time
decision recorded in Deploy/operations.md. It does not change Umami's
already-UTC PostgreSQL configuration.
