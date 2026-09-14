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
does not consume this OpenAI-compatible field and rejects it when present.

One explicit enum per stage is preferred to provider- or model-based inference:
the effective cost/quality choice is auditable and repeatable with the stage
configuration, even when provider defaults change.

Maria decided on 2026-09-13 to run a local trial of all three stages with
Cerebras `qwen-3.8-27b` and `reasoning_effort=none`, based on the artifacts of
task `86cbh1apk`. This trial is not a production model-policy or topology
change, does not self-label the four ungraded top-1 results, and leaves
production untouched.

### Amendment note — 2026-09-14

This replaces the local trial settings stated above. Maria's manual product
check found questions on `none` unacceptable and questions on `low` good, so
the local Cerebras `qwen-3.8-27b` question stage uses
`reasoning_effort=low`. The verified local configuration is deliberately
stage-specific: scripture rewrite remains `qwen-3.8-27b` with `none`, while
scripture rerank is configured as `qwen3-30b-a3b-instruct-2507` with `omit`.
This amendment makes no rewrite or rerank change.

Provider-body diagnostics also showed that a question call can spend all
`1024` completion tokens on reasoning, return no content, and become a `502`.
The accepted remedy is the separate operational question ceiling
`AI_QUESTION_MAX_TOKENS=4096`; it becomes live with the coordinated runtime
change. It is a guardrail, not a model, reasoning-mode, or production-policy
change.

Verification: Maria's manual product check on 2026-09-14; the local
stage-specific configuration and the diagnostic provider bodies were inspected
on the same date.

## Consequences

Every OpenAI-compatible chat-stage configuration now makes its reasoning
behaviour explicit and fails fast when incomplete. Operators can compare
cost/quality results using a reproducible per-stage setting. Endpoints and
credentials continue to be documented only in `Deploy`.

## References

- [ADR-0002: AI model provider policy](0002-ai-model-provider-policy.md)
- [ADR-0003: Explicit AI configuration contract](0003-explicit-ai-configuration-contract.md)
- [ClickUp: trial artifacts](https://app.clickup.com/t/86cbh1apk)
