# ADR-0004: Make OpenAI-compatible reasoning effort explicit per chat stage

- Status: Accepted
- Date: 2026-09-13

## Context

Question generation, scripture rewrite, and scripture rerank can use
OpenAI-compatible chat endpoints. Reasoning effort affects response cost,
latency, and quality, so an omitted configuration must not silently inherit a
provider or model default that can later change. Provider and model inference
would also make a trial difficult to reproduce.

Runtime endpoints and secrets remain operational facts in `Deploy`.

## Decision

For each of the question, rewrite, and rerank stages configured as
`openai_compat`, `reasoning_effort` is required and is exactly one of `omit`,
`none`, `low`, `medium`, or `high`. `omit` is an explicit instruction to send
no reasoning-effort field to an endpoint that does not support it; it is not a
missing value. A missing or invalid value rejects startup. A `gemini` stage
does not consume this OpenAI-compatible field. The reviewed Gemini question
models instead use the model-specific thinking contract in ADR-0009.

One explicit enum per stage is preferred to provider- or model-based inference:
the effective cost/quality choice is auditable and repeatable with the stage
configuration, even when provider defaults change.

Maria decided on 2026-09-13 to run a local trial of all three stages with
Cerebras `qwen-3.8-27b` and `reasoning_effort=none`, based on the artifacts of
task `86cbh1apk`. This trial is not a production model-policy or topology
change, does not self-label the four ungraded top-1 results, and leaves
production untouched.

### Amendment note — 2026-09-14

Question generation has a separate operational output ceiling:
`AI_QUESTION_MAX_TOKENS`, with an explicit default of `4096`. It includes any
provider reasoning tokens. Blank, non-integer or non-positive values must
reject startup. The ceiling does not change prompts, retry policy or the
selected model, and an empty provider response remains an error.

Implementation: [Bible-API ADR-0020](https://github.com/BibleGarden/Bible-API/blob/main/architect/adr/0020-explicit-chat-reasoning-effort.md).
This preserves the contract introduced on 2026-09-14; active model selection
is governed by ADR-0006 and ADR-0011, and runtime settings belong in `Deploy`.

## Consequences

Every OpenAI-compatible chat-stage configuration now makes its reasoning
behaviour explicit and fails fast when incomplete. Operators can compare
cost/quality results using a reproducible per-stage setting. Endpoints and
credentials continue to be documented only in `Deploy`.

## References

- [ADR-0002: AI model provider policy](0002-ai-model-provider-policy.md)
- [ADR-0003: Explicit AI configuration contract](0003-explicit-ai-configuration-contract.md)
- [ADR-0009: Gemini question thinking](0009-gemini-question-thinking.md)
- [ClickUp: trial artifacts](https://app.clickup.com/t/86cbh1apk)
