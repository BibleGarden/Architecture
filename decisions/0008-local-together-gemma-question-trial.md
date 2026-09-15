# ADR-0008: Local Together Gemma question trial

- Status: Integration retained; interactive trial rejected
- Date: 2026-09-14
- Amended: 2026-09-15
- Approval: Maria's instruction to retain the implementation after rejecting interactive use on latency

## Context

Maria requested moving local question generation from OpenRouter to Together
while retaining Gemma. Production model selection remains governed by ADR-0006.

## Decision

Retain the optional question-only `AI_QUESTION_PROVIDER=together` profile with
model `google/gemma-4-31B-it`, base endpoint `https://api.together.ai/v1`, a
non-empty stage API key and `AI_QUESTION_REASONING_EFFORT=none`. Startup must
reject any other model, endpoint or reasoning value for this profile, a missing
or empty key, and use of `together` for another stage.

The client maps `none` to `reasoning={"enabled":false}` and requests
`response_format={"type":"json_object"}`. It must never send flat
`reasoning_effort` or OpenRouter `provider` routing fields. Remove the
OpenRouter-specific routing variable. No alternative provider or model is
configured on failure.

Together is not selected for practical interactive question generation: Maria
rejected the trial on 2026-09-15 because response times were unstable. Retaining
the integration permits deliberate future trials; it is not a production
recommendation or a decision to select a replacement provider.

Retain the operational ceilings of 4,096 output tokens and 20 seconds per
question call. Scripture rewrite, rerank, transcription and embeddings remain
unchanged. Credentials and container operation are documented in `Deploy`.

## Verification and consequences

On 2026-09-14, Together's `GET /v1/models` catalogue listed the exact model ID
above as Gemma 4 31B-it FP8. This retains the model family but changes serving
precision from the previous Venice BF16 route; equivalent answer quality has
not been established.

Direct diagnostics from the local container on 2026-09-14 showed reasoning
content when the control was omitted and none with explicit disabling. Active
API requests and matched provider logs on 2026-09-15 verified the dedicated
request contract and valid question responses without reasoning fields.

The same active profile nevertheless returned HTTP 502 after timeouts of
17.164 and 17.015 seconds with reasoning disabled; the identical next payload
returned HTTP 200 in 1.511 seconds. This observed latency instability is the
basis for rejecting interactive use, recorded in
[incident 86cbhbakd](https://app.clickup.com/t/86cbhbakd). The other stages and
credentials were preserved. At trial close the local runtime still used
Together; no replacement had been selected or applied.

This local trial sends question content to Together. It does not approve
Together for production or change production configuration, consent, prompts
or the retrieval pipeline. Production provider changes remain subject to the
existing privacy rules and a separate deployment decision.

## References

- [ADR-0003: Explicit AI configuration contract](0003-explicit-ai-configuration-contract.md)
- [ADR-0004: Explicit OpenAI reasoning effort](0004-explicit-openai-reasoning-effort.md)
- [ADR-0006: Approved AI model selection](0006-approved-ai-model-selection.md)
- [Lampada data processing rules](../privacy/lampada-data-processing.md)
- [Together reasoning controls](https://docs.together.ai/docs/inference/chat/reasoning)
- [Together chat completions API](https://docs.together.ai/reference/chat-completions)
