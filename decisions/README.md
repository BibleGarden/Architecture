# Architecture decisions

Cross-repository decisions are recorded here when they affect more than one
BibleGarden repository or an external system contract.

| ADR | Status | Decision |
| --- | --- | --- |
| [0001](0001-lampada-ai-data-processing.md) | Accepted | Require separate explicit consent and paid processing for every Lampada AI purpose |
| [0002](0002-ai-model-provider-policy.md) | Superseded for model selection | Preserve worldwide availability; ADR-0006 records the selected models |
| [0003](0003-explicit-ai-configuration-contract.md) | Accepted | Configure each AI stage explicitly without inherited providers or credentials |
| [0004](0004-explicit-openai-reasoning-effort.md) | Accepted | Require an explicit reasoning-effort enum for each OpenAI-compatible chat stage |
| [0005](0005-server-controlled-ai-prefetch.md) | Accepted | Let the server disable or limit speculative question and scripture requests |
| [0006](0006-approved-ai-model-selection.md) | Accepted | Select models for each AI stage while retaining ADR-0002's availability requirement |
| [0008](0008-together-question-profile.md) | Accepted | Isolate the optional Together question profile and explicitly disable reasoning |
| [0009](0009-gemini-question-thinking.md) | Accepted | Require an explicit thinking level for Gemini 3.8 Flash questions |
