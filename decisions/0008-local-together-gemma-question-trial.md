# ADR-0008: Local Together Gemma question trial

- Status: Accepted
- Date: 2026-09-14
- Amended: 2026-09-15
- Approval: Maria's instruction to retain Gemma on Together and explicitly disable reasoning

## Context

Maria requested moving local question generation from OpenRouter to Together
while retaining Gemma. Production model selection remains governed by ADR-0006.

## Decision

Use the dedicated question-only `AI_QUESTION_PROVIDER=together` profile with
model `google/gemma-4-31B-it`, base endpoint `https://api.together.ai/v1`, a
non-empty stage API key and `AI_QUESTION_REASONING_EFFORT=none`. Startup must
reject any other model, endpoint or reasoning value for this profile, a missing
or empty key, and use of `together` for another stage.

The client maps `none` to `reasoning={"enabled":false}` and requests
`response_format={"type":"json_object"}`. It must never send flat
`reasoning_effort` or OpenRouter `provider` routing fields. Remove the
OpenRouter-specific routing variable. No alternative provider or model is
configured on failure.

Retain the operational ceilings of 4,096 output tokens and 20 seconds per
question call. Scripture rewrite, rerank, transcription and embeddings remain
unchanged. Credentials and container operation are documented in `Deploy`.

## Verification and consequences

On 2026-09-14, Together's `GET /v1/models` catalogue listed the exact model ID
above as Gemma 4 31B-it FP8. This retains the model family but changes serving
precision from the previous Venice BF16 route; equivalent answer quality has
not been established.

Direct diagnostics from the local container on 2026-09-14 used synthetic
Russian question prompts. Paired requests showed reasoning content when the
control was omitted and none with `reasoning={"enabled":false}`. Requests
with reasoning disabled returned valid question JSON, but timings varied and
did not establish a latency improvement. The former omitted-reasoning API
profile also produced a provider timeout and HTTP 502. These observations
motivate explicit disabling; they do not establish latency or quality
acceptance.

On 2026-09-15, the active local API passed its health check and one synthetic
Russian first-question request returned HTTP 200 in 4.007 seconds. Matched
provider logs confirmed the disabled-reasoning object and JSON response format,
with no flat reasoning effort or provider routing fields. The response had no
reasoning fields. Other stages and credentials were preserved. This verifies
one successful request; Maria's quality review remains pending.

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
