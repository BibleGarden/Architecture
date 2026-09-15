# ADR-0010: Explicit Gemini question service tier

- Status: Accepted
- Date: 2026-09-15

## Context

Google's GenerateContent API accepts a service tier separately from model and
thinking level. Priority requests can be downgraded by Google to Standard;
requesting Priority does not prove which tier served the response.

## Decision

Require question-only `AI_QUESTION_SERVICE_TIER=standard|priority` when AI is
enabled and the question provider is `gemini`. Reject the variable otherwise,
including blank or unsupported values. Send it as top-level `serviceTier` in
the Gemini question request. Other stages and providers do not receive it.

For a Priority request, require the documented `x-gemini-service-tier` response
header to report `priority`. Missing confirmation or a different tier is a
provider error; do not accept the answer as a Priority result. If the response
also contains `usageMetadata.serviceTier`, it must agree with the header.
Do not infer the header from the response body or retry on another tier.

Standard requests retain the existing answer behavior. Startup and diagnostics
identify requested and actual tiers without printing arbitrary header values
or credentials. Thinking configuration remains governed by ADR-0009.

## Consequences

Operators explicitly choose the requested service tier. The client cannot stop
Google from performing a downgrade, but it refuses to silently accept one for
Priority. Such a response may already have consumed provider resources before
being rejected locally. Credentials and active settings belong in `Deploy`;
production model selection and trial reporting remain separate.

## References

- [Google Priority inference](https://ai.google.dev/gemini-api/docs/priority-inference)
- [GenerateContent service tier](https://ai.google.dev/api/generate-content#ServiceTier)
- [ADR-0009: Gemini thinking levels](0009-gemini-question-thinking.md)
