# Ramblings about AI

An ongoing collection of written findings about how large language models actually behave. Each document is self-contained: it states its claim, its evidence, its limits, and can be read on its own. Identifying details are removed from everything here, and counts are exact.

New findings get added as they happen, one document each, indexed below.

## Index

### The delivery experiment (first series, 2026)

A single controlled experiment on how the delivery of instructions affects model compliance, and what chasing one missing reply header revealed about how these models are built. The three documents form a set and read best in order.

1. [The Binding Weakness of System Prompts](system-prompt-weakness.md) — the experiment itself: a behavioral instruction set delivered four ways, adherence running 18/21, 14/21, 15/18, and 18/18. A system prompt can make information available but cannot make behavior binding.
2. [One Dynamic, Three Failures](root-dynamic-three-failures.md) — the root cause: the learned objective creates the gap, the architecture makes it stick, stochasticity makes it a lottery, and compliance failures, hallucination, and jailbreak susceptibility are one dynamic wearing three hats.
3. [Verification Without Identification](verification-without-identification.md) — the division of labor: the assistant verifies what it is pointed at; the identification act belongs to the human, and the operational rules that implies.

*Short version of the series:* the tokens were in the same context window in every arm. The only difference was whether they ever became the object of the model's own generation. Text that is never answered is text that is never weighted.
