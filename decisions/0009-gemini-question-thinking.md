# ADR-0009: Explicit thinking control for Gemini 3.8 Flash questions

- Status: Accepted
- Date: 2026-09-15

## Context

Google's Gemini 3.8 Flash supports internal thinking. Question generation needs
an explicit choice rather than inheriting the provider's thinking default.
The existing transport uses the `generateContent` API.

## Decision

For question generation with `AI_QUESTION_PROVIDER=gemini` and
`AI_QUESTION_MODEL=gemini-3.8-flash`, require
an explicit `AI_QUESTION_REASONING_EFFORT` of `low`, `medium` or `high`.
The client maps it to `generationConfig.thinkingConfig.thinkingLevel`
(`LOW`, `MEDIUM` or `HIGH`). Missing or unsupported values, including `none`
and `omit`, must stop startup for this exact provider/model pair.

The endpoint is supplied by the Gemini transport; `AI_QUESTION_ENDPOINT` and
`AI_QUESTION_OPENROUTER_PROVIDER_ENDPOINT` must be absent. The question-stage
key must be non-empty. Gemini uses neither flat OpenAI reasoning effort nor
OpenRouter/Together reasoning and routing objects.

Other Gemini models and stages retain their existing configuration contract,
as do OpenRouter, Together and the generic OpenAI-compatible profile. No
prompt, retry, timeout or output-token-limit change accompanies this mapping.

## Consequences

Thinking depth is explicit for the selected question model; `low` is the
lowest documented level, not a claim that thinking is disabled. Credential
sources and local runtime configuration are operational facts in `Deploy`.
Production model selection remains governed by ADR-0006. Trial measurements
and provider comparisons belong in `AI-Evaluation`.

## References

- [GenerateContent ThinkingConfig](https://ai.google.dev/api/generate-content#ThinkingConfig)
- [ADR-0003: Explicit AI configuration contract](0003-explicit-ai-configuration-contract.md)
- [ADR-0004: Explicit reasoning configuration](0004-explicit-openai-reasoning-effort.md)
- [ADR-0006: Approved production model selection](0006-approved-ai-model-selection.md)
