# Architecture decisions

Cross-repository decisions are recorded here when they affect more than one
BibleGarden repository or an external system contract.

| ADR | Status | Decision |
| --- | --- | --- |
| [0001](0001-lampada-ai-data-processing.md) | Accepted | Require separate explicit consent and paid processing for every Lampada AI purpose |
| [0002](0002-ai-model-provider-policy.md) | Accepted | Use AI models compatible with worldwide availability and keep runtime configuration operational |
| [0003](0003-explicit-ai-configuration-contract.md) | Accepted | Configure each AI stage explicitly without inherited providers or credentials |
| [0004](0004-explicit-openai-reasoning-effort.md) | Accepted | Require an explicit reasoning-effort enum for each OpenAI-compatible chat stage |
| [0005](0005-server-controlled-ai-prefetch.md) | Accepted | Let the server disable or limit speculative question and scripture requests |
