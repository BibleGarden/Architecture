# ADR-0002: AI models must preserve worldwide availability

- Status: Superseded for model selection by ADR-0006; worldwide-availability principle retained
- Date: 2026-09-06

## Context

Bible Garden and related applications must be usable by people of every age and
country. Provider terms that exclude minors or countries conflict with this
product requirement. The project evaluated locally deployable models for the AI
stages and measured an alternative scripture-retrieval design.

Runtime provider endpoints, credentials, host topology and which model is live
at a particular moment are operational facts. They are deliberately absent from
this public ADR and are maintained in the private `Deploy` repository.

## Decision

1. The target AI contour uses self-hosted models or models whose licence and
   terms do not impose age or country restrictions on the application. An
   external Anthropic or OpenAI API is not a substitute for this requirement.
2. The retrieval pipeline remains rewrite → embeddings and BM25 → interleave
   → blacklist → diversity → rerank. ADR-0006 records the selected models for
   its stages.
3. The measured replacement of rewrite with a semantic-only index was rejected.
   It must not be reintroduced without new evaluation evidence.
4. ADR-0006 replaces the question-generation model selection.

## Consequences

- ADR-0001 records the 2026-09-03 Gemini consent contract. The provider-specific
  privacy notice must name the provider and terms actually in use.

## References

- [System architecture](../architecture.md)
- [ADR-0001: Lampada AI processing consent](0001-lampada-ai-data-processing.md)
- [ADR-0006: Approved AI model selection](0006-approved-ai-model-selection.md)
- [AI Evaluation](https://github.com/BibleGarden/AI-Evaluation)
