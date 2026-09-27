# ADR-0016: Gemini Flash-Lite for scripture rewrite and rerank

- Status: Accepted
- Date: 2026-09-27
- Ticket: ClickUp 123pfqn0kzh
- Supersedes: the rewrite and rerank rows of ADR-0006
- Approval: Maria's production instruction, 2026-09-27

## Context

Scripture rewrite sent the prayer topic, and replies under answer-context
consent, to Cerebras. ADR-0006 approved Cerebras as a transport without any
review of its data terms, and the public Lampada policy did not disclose it.
On the same day the company-hosted Qwen chat service used for rerank was
unavailable.

## Decision

Use Google's native Gemini API with `gemini-3.5-flash-lite` for scripture
rewrite and rerank, with the same paid Google project as question generation
(ADR-0011). Google is the only external AI provider that receives prayer
content. Transcription and embeddings stay on the company-hosted services
selected in ADR-0006. No quality evaluation was run before the switch, by
owner decision.

## Consequences

- Prayer topics and allowed replies reach Google for questions and for
  scripture selection; `privacy/lampada-data-processing.md` and the public
  Lampada policy name Google for both purposes.
- Cerebras must not receive production content again without a recorded review
  under "Changing the provider or its terms" in that document.
- As with ADR-0011, this records the runtime selection; it does not establish
  compliance with ADR-0002's worldwide-availability requirement.

## References

- [ADR-0006: AI model selection](0006-approved-ai-model-selection.md)
- [ADR-0011: Gemini for production questions](0011-gemini-production-questions.md)
- [AI data processing](../privacy/lampada-data-processing.md)
