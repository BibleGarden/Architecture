# ADR-0013: Pseudonymize client addresses in application statistics

- Status: Accepted
- Date: 2026-09-26
- Ticket: ClickUp 123pfqmzumj

## Context

Bible-API stored raw addresses for most requests in `cep_public.api_requests`.
Only AI endpoints stored HMAC pseudonyms. The dashboard displayed and searched
the raw address, although the statistics use it only to count distinct clients.

## Decision

Bible-API stores the first 40 hexadecimal characters of HMAC-SHA-256 of the
resolved client IP for every recorded API request. `CLIENT_HMAC_KEY` is required
at startup even with AI disabled. The previous `AI_CLIENT_HMAC_KEY` value is
copied unchanged to retain existing pseudonym identity; its old name is
rejected. AI requests continue to omit the user agent, while other requests
retain it. Request and response bodies are not part of the statistics log.

The dashboard shows the first eight pseudonym characters with the full value
available in a tooltip, and searches by full pseudonym or prefix. "Unique
clients" means distinct IP-based pseudonyms in the available period, not
people or devices. The existing `unique_ips` API field and `client_ip` storage
column keep their historical names; the recent-request API calls the exposed
value `client_pseudonym`.

After release, a one-off Bible-API command validates stored values with
`ipaddress` and converts only raw IPv4/IPv6 addresses in bounded batches. Its
dry run counts candidates without writing, and repeated runs skip converted
rows. The AI rate limiter still receives the resolved real IP in memory and
uses its existing HMAC client buckets. A separate nginx access-log change,
decided in parallel, will remove IP addresses from that log.

## Consequences

The raw request table retains its 14-day purge schedule. Historical daily
aggregates remain as recorded; counts across the change can differ because
previously one address had both raw and AI-pseudonym identities. Keeping the
key value stable preserves comparisons after conversion. Pseudonyms are stable
within a key's lifetime and remain sensitive operational data; access stays
limited to authenticated statistics.
