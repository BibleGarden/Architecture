# ADR-0017: Google Gemini as an alternative transcription route

- Status: Accepted
- Date: 2026-09-28
- Amends: the transcription row of ADR-0006 and the transcription statement of
  ADR-0016
- Approval: Maria's instruction, 2026-09-28

## Context

Lampada transcribes a recording the person chooses through Bible API. ADR-0006
selected Whisper `large-v3-turbo` on a company-hosted service, and ADR-0016
stated that transcription stays company-hosted. The owner wants the option to
move transcription to Google for speed without asking every person for consent
again.

## Decision

A Lampada recording may be transcribed by either of two processors:

- Whisper `large-v3-turbo` on the company-hosted service (ADR-0006);
- Google Gemini through Google's paid API, in the same paid Google project as
  question generation (ADR-0011) and scripture rewrite and rerank (ADR-0016).

A deployment uses one transcription route at a time and never sends the same
recording to both. The Gemini transcription model is recorded in this directory
before that route receives production content.

The transcription consent notice names both processors, so switching between
them does not invalidate stored decisions. Lampada Mobile records this in its
own ADR-0035 with the provider contract
`google-gemini-paid-whisper-self-hosted-2026-09`. Embeddings (bge-m3) remain
company-hosted and do not reach Google.

## Consequences

- `privacy/lampada-data-processing.md` and the public Lampada policy name both
  transcription processors and apply Google's paid-API terms to recordings.
- Release checks cover billing and disabled Gemini logging for the transcription
  model whenever transcription is routed to Gemini.
- Bible API and its logs still do not persist recordings or transcripts on
  either route.

## References

- [ADR-0006: AI model selection](0006-approved-ai-model-selection.md)
- [ADR-0016: Gemini for scripture rewrite and rerank](0016-gemini-scripture-rewrite-rerank.md)
- [AI data processing](../privacy/lampada-data-processing.md)
