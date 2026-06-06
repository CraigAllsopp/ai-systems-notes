# ai-systems-notes

Architecture notes from AI systems I've built as independent R&D. Concepts and methodology, written up plainly.

I came to AI from 20 years delivering M&E and decarbonisation infrastructure — and I build AI systems with the same instincts: gates before you trust a change, verification distinct from completion, provenance on everything, and a healthy respect for the cost of being confidently wrong.

## Notes

- **[Cross-Paradigm AI](cross-paradigm-board.md)** — why a multi-agent "board" needs genuinely different minds, not the same model N times. Disagreement as signal.
- **[Engineering Independence](engineering-independence.md)** — why a board of different LLMs still shares a blind spot (they're all next-token predictors over human text), and why the fix is a member that isn't an LLM. Independence, not neutrality.
- **[Two-Stores Architecture](two-stores-architecture.md)** — raw verbatim capture + curated knowledge store, and why separating them beats one clever store. Lets you be wrong about curation safely.
- **[The Observation-Target Model](observation-target-model.md)** — name a system by what it watches, not what it is. One engine, many targets — plug in, don't spawn.
- **[Verification Discipline](verification-discipline.md)** — the four-state lifecycle (pending → in_progress → done → verified) and why "done" isn't "verified." The verifier must be independent of the doer. Pairs with [programme-parser](https://github.com/CraigAllsopp/programme-parser).

## Tools

These notes pair with small open-source tools that implement the patterns:
- **[programme-parser](https://github.com/CraigAllsopp/programme-parser)** — four-state Markdown task parser
- **[scan-history](https://github.com/CraigAllsopp/scan-history)** — cross-reference-before-flag primitive (a persistent seen-ledger)
- **[cron-logger](https://github.com/CraigAllsopp/cron-logger)** — persistent fire-log for scheduled jobs; spots silent cron death

---

*Independent R&D. The architecture and methodology are my own; happy to discuss or demo. — [Craig Allsopp](https://github.com/CraigAllsopp)*
