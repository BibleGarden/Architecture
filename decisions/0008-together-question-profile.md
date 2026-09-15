# ADR-0008: Explicit Together question profile

- Status: Accepted
- Date: 2026-09-15

## Context

Together and OpenRouter use Chat Completions but have different request
policies. Selecting a provider must explicitly select its configuration and
wire contract, independently of the endpoint hostname.

## Decision

Retain the optional question-only `AI_QUESTION_PROVIDER=together` profile with
model `google/gemma-4-31B-it`, endpoint `https://api.together.ai/v1`, a non-empty
stage API key and `AI_QUESTION_REASONING_EFFORT=none`. Missing or different
values, use on another stage, and leftover OpenRouter routing configuration
must stop startup.

The profile maps `none` to `reasoning={"enabled":false}` and requests
`response_format={"type":"json_object"}`. It sends neither flat
`reasoning_effort` nor OpenRouter `provider` routing fields. Existing provider
profiles retain their independent contracts; no alternate provider is selected
on failure.

## Consequences

The integration is available for deliberate use; availability does not select
it as the working provider. Production selection remains governed by ADR-0006.
Credentials and operation belong in `Deploy`. Trial measurements and suitability
conclusions belong in `AI-Evaluation`.

## References

- [Bible-API request contract](https://github.com/BibleGarden/Bible-API/blob/main/architect/adr/0023-together-question-profile.md)
- [Together trial report](https://github.com/BibleGarden/AI-Evaluation/blob/main/evaluation/bench_data/together_gemma_2026-09-15/report.md)
- [ADR-0003: Explicit AI configuration contract](0003-explicit-ai-configuration-contract.md)
- [ADR-0006: Approved AI model selection](0006-approved-ai-model-selection.md)
- [Together chat completions API](https://docs.together.ai/reference/chat-completions)
