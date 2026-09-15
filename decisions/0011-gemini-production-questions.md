# ADR-0011: Gemini for production question generation

- Status: Accepted
- Date: 2026-09-15
- Supersedes: the question-generation selection in ADR-0006
- Approval: Maria's explicit production-deployment instruction, 2026-09-15

## Decision

Use Google's native Gemini API with `gemini-3.8-flash` for question generation,
thinking level `LOW` and service tier `standard`. The question-stage credential
is configured independently. ADR-0009 and ADR-0010 define the thinking and
service-tier contracts; deployment settings and credential storage belong in
the private `Deploy` repository.

All other model selections in ADR-0006 remain accepted. This decision does not
change prompts, the retrieval pipeline or the public API contract. Provider
errors remain explicit; there is no server-side switch to another provider.

## Consequences

The question route sends question context to Google. This approval records the
runtime model selection; it does not establish compliance with ADR-0002's
worldwide-availability requirement. Provider disclosure and consent remain
governed by `privacy/lampada-data-processing.md`.

## References

- [ADR-0006: AI model selection](0006-approved-ai-model-selection.md)
- [ADR-0009: Gemini thinking levels](0009-gemini-question-thinking.md)
- [ADR-0010: Gemini service tier](0010-gemini-question-service-tier.md)
- [AI data processing](../privacy/lampada-data-processing.md)
