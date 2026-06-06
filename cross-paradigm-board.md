# Cross-Paradigm AI: why a multi-agent "board" needs different minds, not the same mind N times

*Part of a series of architecture notes from systems I've built as independent R&D.*

## The problem with most "multi-agent" systems

Most multi-agent setups run the **same model** with different system prompts and call it a team. That isn't multi-agent — it's multi-*view* of a single mind. The agents share training data, share blind spots, and converge on the same answer through slightly different doors. You get the *appearance* of deliberation with none of the protection.

If three agents share a bias, three agents will confirm it three times. Consensus among same-paradigm models is often shared-bias noise dressed up as agreement.

## What I built instead

A **board** of agents drawn from genuinely different model families / training paradigms, run in one of four modes:

| Mode | Shape | Use |
|---|---|---|
| **Pipeline** | each agent builds on the last | progressive refinement |
| **Compare** | all answer blind, side by side | surface the spread of views |
| **Adversarial** | one writes, one attacks, one resolves | stress-test a claim before committing |
| **Consensus** | all respond, a synthesiser merges | when you genuinely need one answer |

The design principle that matters most: **disagreement is signal, not noise.** Where the board splits, that's where the interesting question lives — so the system preserves the split rather than averaging it away.

## A real example

In adversarial mode reviewing a technical design, the **writer** model proposed a list of "prior art" the design had missed. The **attacker** model caught that the writer had mis-classified the entire category — the cited prior art solved a *different* problem class than the one under review. The **resolver** confirmed the attacker was right and corrected the record.

A single model — or three copies of the same model — would very likely have accepted the original mis-classification, because the error was *plausible*. It took a genuinely different mind in the attacker seat to catch it. That's the whole argument for cross-paradigm in one example.

## Why this matters beyond novelty

1. **Bias resistance by construction** — not "we tried to be unbiased" but "the architecture makes single-source bias structurally harder to propagate."
2. **Every disagreement is a research question** — a split board output isn't a failure to decide; it's a map of where the evidence is thin.
3. **Cheap insurance** — running a draft past a different paradigm before you commit costs a few API calls and catches the plausible-but-wrong.

## What I'd caution

- More agents ≠ more paradigms. Eleven models from three labs might be three paradigms wearing eleven hats. Diversity has to be real to count.
- A synthesiser that just averages defeats the point. The value is in *preserving* structured disagreement for a human (or a downstream adversarial pass) to adjudicate.
- Cross-paradigm costs latency and money. Use it where being wrong is expensive, not for every call.

---

*This note describes a personal system built as independent R&D. The architecture and methodology are my own; happy to discuss or demo.*
