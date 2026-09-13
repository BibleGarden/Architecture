# ADR-0003: AI configuration is explicit per stage

- Status: Accepted
- Date: 2026-09-13

## Context

Bible-API previously allowed a shared provider configuration and key inheritance.
That made the effective provider and credential of a stage less visible, and
could send a credential to a provider selected elsewhere. The configuration
contract must be strict without changing ADR-0002's models, prompts, pipeline,
topology, or deployment.

## Decision

`AI_ENABLED` is the only explicit switch for the AI surface. When enabled,
question, scripture rewrite, scripture rerank, and transcription each state
their own provider and model; a remote stage also states its own endpoint and
key. An explicitly present empty remote key means that endpoint is
unauthenticated. A missing key is invalid.

Embeddings are an independent mandatory stage in every deployment. Its
provider and model identity, including the vector dimensions, identify the
index being read or built; its remote endpoint and key are likewise
stage-specific.

There are no legacy aliases, shared `AI_OPENAI_COMPAT_*` values, or
`GEMINI_API_KEY` switch or inheritance. Startup rejects incomplete,
contradictory, or legacy configuration.

Each stage repeats its complete configuration rather than using a backend
registry or alias indirection. Repetition keeps the effective provider,
model, endpoint, and credential boundary auditable in one place and makes
fail-fast validation deterministic. Per-stage credentials prevent accidental
cross-provider secret inheritance and limit a credential to the service it is
intended to authorize.

## Consequences

Existing deployments must migrate every enabled stage atomically to explicit
settings and remove legacy variables; partial migrations fail at startup.
Operators must keep embedding identity aligned with the stored index and
rebuild or select the matching index when that identity changes. Runtime
endpoints and secrets remain operational documentation in `Deploy`.

## References

- [ADR-0002: AI model provider policy](0002-ai-model-provider-policy.md)
