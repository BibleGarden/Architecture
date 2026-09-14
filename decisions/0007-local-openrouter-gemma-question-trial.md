# ADR-0007: Local OpenRouter Gemma question trial

- Status: Accepted
- Date: 2026-09-14
- Approval: Maria's direct local/test instruction, 2026-09-14

## Context

The local Cerebras Qwen question trial recorded in ADR-0004 produced questions
Maria judged awkward. Gemma was evaluated as a no-thinking question model, but
ADR-0006 is the current approved production model selection and keeps Cerebras
Qwen with `low` reasoning for production questions.

This decision records a separately configured local/test trial. It does not
replace the history in ADR-0004 or the production selection in ADR-0006.

## Decision

For the local/test question stage only, use the paid OpenRouter provider
(`openrouter`) with model `google/gemma-4-31b-it`. Its base endpoint is
`https://openrouter.ai/api/v1`; question calls use
`https://openrouter.ai/api/v1/chat/completions`.

The configuration requires `AI_QUESTION_REASONING_EFFORT=none`. The request
maps that value to `reasoning: {"enabled": false}` and must not send the flat
`reasoning_effort` field. It uses `AI_QUESTION_MAX_TOKENS=4096` and a 20-second
timeout.

Every request includes the strict OpenRouter provider policy
`provider.only=["venice/bf16"]`, `provider.allow_fallbacks=false` and
`provider.data_collection=deny`. The full endpoint slug, rather than the base
`venice` slug, is required: no other Venice endpoint or provider is eligible,
and a failed Venice request fails rather than automatically moving elsewhere.

Maria selected `venice/bf16` for the trial's answer quality. The endpoint
catalogue identifies it as BF16; OpenRouter documents that lower-precision
quantized variants can degrade performance for some prompts. This is a
quality-oriented selection, not a claim that BF16 is a measured quality
benchmark.

Observed endpoint data is point-in-time only. On 2026-09-14, a direct read of
OpenRouter's `GET /api/v1/models/google/gemma-4-31b-it/endpoints` catalogue
reported the `venice/bf16` endpoint as `status=0`, with 99.84410185345574%
uptime for the preceding five minutes (99.87859669247682% for 30 minutes). It
listed a 256,000-token context, an 8,192-token completion maximum, and prices
of $0.12 per million input tokens and $0.36 per million output tokens. The
catalogue observation is not an SLA or a promise of future availability,
routing, quality or price.

## Consequences

The local/test question calls become paid calls to OpenRouter. The trial does
not change the production provider policy, production topology, production
consent, or the production model selection in ADR-0006. It does not change
scripture rewrite or rerank.

At the time of this decision, the inspected question runtime sends the existing
fallback and data-collection fields but does not yet send
`provider.only=["venice/bf16"]`. It must implement this request contract
before a local/test call may be described as pinned to Venice BF16.

The normative privacy document continues to name Google Gemini as the current
production provider. Before OpenRouter can receive production prayer content,
the provider-change requirements in that document apply; this local/test trial
does not satisfy them.

## References

- [ADR-0004: Explicit OpenAI reasoning effort](0004-explicit-openai-reasoning-effort.md)
- [ADR-0006: Approved AI model selection](0006-approved-ai-model-selection.md)
- [Lampada data processing rules](../privacy/lampada-data-processing.md)
- [OpenRouter provider routing](https://openrouter.ai/docs/guides/routing/provider-selection)
- [OpenRouter Gemma endpoint catalogue](https://openrouter.ai/api/v1/models/google/gemma-4-31b-it/endpoints)
