# ADR-0009: Explicit thinking levels for Gemini Flash questions

- Status: Accepted
- Date: 2026-09-15

## Context

The reviewed Gemini Flash question models use Google's `generateContent`
thinking-level control. Each model's supported levels must be validated
explicitly instead of inheriting a provider default.

## Decision

For question generation with `AI_QUESTION_PROVIDER=gemini`, require an explicit
`AI_QUESTION_REASONING_EFFORT` for these model IDs:

| Model | Allowed levels |
| --- | --- |
| `gemini-3.8-flash` | `low`, `medium`, `high` |
| `gemini-3.5-flash-lite` | `minimal`, `low`, `medium`, `high` |

The client maps the selected value to
`generationConfig.thinkingConfig.thinkingLevel` using Google's uppercase enum.
Missing, padded or unsupported values must stop startup. In particular,
`none` and `omit` are not supported, and `minimal` is valid only for the
Flash Lite profile, not for Gemini 3.8 or the generic OpenAI-compatible client.

The Gemini transport supplies the endpoint. `AI_QUESTION_ENDPOINT` and
`AI_QUESTION_OPENROUTER_PROVIDER_ENDPOINT` must be absent; the question-stage
key must be non-empty. Gemini receives no OpenAI/OpenRouter reasoning objects.

Other Gemini models and stages, OpenRouter, Together and generic
OpenAI-compatible profiles retain their existing contracts. Prompts, retries,
timeouts and output-token limits are unchanged.

## Consequences

Existing question configurations for either reviewed model must add an explicit
thinking level before startup. `minimal` means little to no thinking, not a
guarantee that thinking is disabled. Credentials and active configuration are
operational facts in `Deploy`. Production selection remains governed by
ADR-0006; trial measurements and comparisons belong in `AI-Evaluation`.

## References

- [Gemini thinking guide](https://ai.google.dev/gemini-api/docs/thinking)
- [GenerateContent ThinkingConfig](https://ai.google.dev/api/generate-content#ThinkingConfig)
- [ADR-0003: Explicit AI configuration contract](0003-explicit-ai-configuration-contract.md)
- [ADR-0006: Approved production model selection](0006-approved-ai-model-selection.md)
