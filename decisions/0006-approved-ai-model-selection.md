# ADR-0006: Approved AI model selection

- Status: Accepted for non-question stages; question selection superseded by ADR-0011
- Date: 2026-09-14
- Supersedes: the model-selection parts of ADR-0002
- Approval: Maria's production-deployment instruction, 2026-09-14

## Context

The production configuration must name one model for each AI stage. The former
selection in ADR-0002 no longer matches the approved configuration. The
retrieval pipeline itself remains accepted: rewrite → embeddings and BM25 →
interleave → blacklist → diversity → rerank.

Provider endpoints, credentials and deployment host topology are private
operational facts and remain in `Deploy`.

## Decision

Use the following model selection. The question row records the 2026-09-14
selection; [ADR-0011](0011-gemini-production-questions.md) replaces it with
Gemini for production questions from 2026-09-15.

| Stage | Model | Transport decision |
| --- | --- | --- |
| Question generation | Cerebras `qwen-3.8-27b` | OpenAI-compatible; reasoning effort `low` |
| Scripture rewrite | Cerebras `qwen-3.8-27b` | OpenAI-compatible; reasoning effort `none` |
| Scripture rerank | `qwen3-30b-a3b-instruct-2507` | Company-hosted OpenAI-compatible service; reasoning effort omitted explicitly |
| Transcription | `large-v3-turbo` | Company-hosted OpenAI-compatible service |
| Embeddings | `bge-m3` at 1024 dimensions | Company-hosted OpenAI-compatible service |

Cerebras is an approved private runtime transport for the question and rewrite
stages. The explicit per-stage configuration, including its fail-fast rules,
remains governed by ADR-0003; the explicit reasoning values remain governed by
ADR-0004. This decision does not change the retrieval pipeline, prompts or
public API contract.

## Consequences

The selected question and rewrite model replaces Gemma 4 31B IT and the former
Qwen3-30B rewrite selection in ADR-0002. Operators must configure every stage
explicitly and preserve the embedding model identity with the index it reads.

ADR-0002's worldwide-availability requirement remains binding. This ADR does
not establish that Cerebras' terms, availability, data processing, retention,
human review or jurisdiction satisfy that requirement: no such legal or
provider evidence is recorded here. Provider-specific notices and consent are
governed by `privacy/lampada-data-processing.md`.

## References

- [ADR-0002: AI model provider policy](0002-ai-model-provider-policy.md)
- [ADR-0003: Explicit AI configuration contract](0003-explicit-ai-configuration-contract.md)
- [ADR-0004: Explicit OpenAI reasoning effort](0004-explicit-openai-reasoning-effort.md)
- [Lampada data processing rules](../privacy/lampada-data-processing.md)
