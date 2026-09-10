# ai-systems-notes

Architecture notes from AI systems I've built as independent R&D. Concepts and methodology, written up plainly.

I came to AI from 20 years delivering M&E and decarbonisation infrastructure — and I build AI systems with the same instincts: gates before you trust a change, verification distinct from completion, provenance on everything, and a healthy respect for the cost of being confidently wrong.

## Notes

- **[Cross-Paradigm AI](cross-paradigm-board.md)** — why a multi-agent "board" needs genuinely different minds, not the same model N times. Disagreement as signal.
- **[Engineering Independence](engineering-independence.md)** — why a board of different LLMs still shares a blind spot (they're all next-token predictors over human text), and why the fix is a member that isn't an LLM. Independence, not neutrality.
- **[Two-Stores Architecture](two-stores-architecture.md)** — raw verbatim capture + curated knowledge store, and why separating them beats one clever store. Lets you be wrong about curation safely.
- **[The Observation-Target Model](observation-target-model.md)** — name a system by what it watches, not what it is. One engine, many targets — plug in, don't spawn.
- **[Verification Discipline](verification-discipline.md)** — the four-state lifecycle (pending → in_progress → done → verified) and why "done" isn't "verified." The verifier must be independent of the doer. Pairs with [programme-parser](https://github.com/CraigAllsopp/programme-parser).
- **[Model-Independent Discipline](model-independent-discipline.md)** — why your system shouldn't care which model is inside it. Swap the model and see what moves: if everything does, you don't have a system, you have a model with some wrapping.
- **[Green Is Not a Guarantee](green-is-not-a-guarantee.md)** — a check that watches the *start* will report success forever. Three shapes of false-green, and what a check that can actually fail looks like. Pairs with [cron-logger](https://github.com/CraigAllsopp/cron-logger).

## Papers

Longer, measured write-ups. Where a note argued a case, these count instances and price them — including the ones where the instrument turned out to carry the defect it was hunting.

- **[The gate ladder](papers/SUP-01-gate-ladder.md)** — what a machine-checkable "proven safe" looks like when you actually write one down, what each rung refuses, and the three ways a gate lies. Develops [Verification Discipline](verification-discipline.md).
- **[The consequence loop](papers/SUP-02-consequence-loop.md)** — feedback architecture for an agent that asserts before it checks, and three failed attempts to automate the hardest part. Develops [Model-Independent Discipline](model-independent-discipline.md) and [Engineering Independence](engineering-independence.md).
- **[The dry seam](papers/SUP-03-dry-seam.md)** — six producers writing to consumers that did not exist, what they cost, and the seventh seam that opened inside the fix for the other six. Develops [Green Is Not a Guarantee](green-is-not-a-guarantee.md).

The notes above them are kept as written. They are what I believed at the time, and where a paper now disagrees with one, that disagreement is the point rather than something to tidy away.

## Tools

These notes pair with small open-source tools that implement the patterns:
- **[programme-parser](https://github.com/CraigAllsopp/programme-parser)** — four-state Markdown task parser
- **[scan-history](https://github.com/CraigAllsopp/scan-history)** — cross-reference-before-flag primitive (a persistent seen-ledger)
- **[cron-logger](https://github.com/CraigAllsopp/cron-logger)** — persistent fire-log for scheduled jobs; spots silent cron death

---

*Independent R&D. The architecture and methodology are my own; happy to discuss or demo. — [Craig Allsopp](https://github.com/CraigAllsopp)*
