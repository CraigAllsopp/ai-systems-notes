# Engineering Independence: why an AI board still needs a member that isn't an AI

*Part of a series of architecture notes from systems I've built as independent R&D. Companion to [Cross-Paradigm AI](cross-paradigm-board.md).*

## The thing the first note didn't say

In [Cross-Paradigm AI](cross-paradigm-board.md) I argued that a multi-agent board needs genuinely different models — different minds, not the same mind wearing N hats — because models from the same family share blind spots and confirm each other's bias.

That's true. It's also incomplete. There's a layer of shared bias that *survives* even a board of genuinely different models, and it took me a while to see it because it hides under the word everyone reaches for: **unbiased**.

## Correlated minds

Every large language model — whoever trained it, whatever the lab, whatever the paradigm — learned from overlapping human text. Different corpora, yes; different fine-tuning, yes; but the substrate is the same kind of thing. They are all, at bottom, next-token predictors over human language.

That shared substrate means their errors are **correlated**. A board of several models from different training lineages reduces the *lab-specific* bias, the *RLHF-specific* bias, the *prompt-specific* bias — but not the bias they all inherit from being language models trained on human text. On a question where that substrate misleads, they can be confidently wrong **the same way**.

And here's the trap: when correlated voters agree, averaging them doesn't cancel the error — it *launders* it. You get a tighter distribution around the wrong answer and call it confidence. The classic result here is the **Condorcet jury theorem**: a majority of voters beats a single voter *only if their errors are independent*. Correlated voters break the theorem. A board of LLMs is exactly the case the theorem warns you about — voters who look independent and aren't.

## The fix is a member that fails differently

If the problem is a shared failure mode, the fix isn't a better LLM or more of them — it's a member whose errors **can't** correlate with theirs, because it doesn't share the substrate. A member that isn't a language model at all. Two kinds earn their seat:

- **A computation engine** for anything formalizable — arithmetic, units, dates, deterministic lookups. A language model can produce a fluent, wrong number; a computation engine produces the right one or refuses. It fails by *erroring*, not by *confabulating* — a completely different failure shape.
- **A deterministic, structural reader** over the knowledge store (see [Two-Stores Architecture](two-stores-architecture.md)). Instead of reading by language-plausibility, it reads by graph connectivity, frequency, and recency: *these two things are linked, this record is stale, this contradicts that.* Where it agrees with the language models, that agreement is real — two genuinely different lenses landed in the same place. Where it disagrees, that seam is exactly where to look.

## Independence, not neutrality

Here's the correction that changed how I describe the whole thing. The non-LLM member is **not** "the unbiased one." There's no unbiased observer — claiming one is the trap, not the escape. The structural reader has its own bias: it sees connectivity, not meaning. It will miss things a language model catches, and catch things a language model misses.

That's the entire point. Its value isn't the *absence* of bias — it's that its bias is **uncorrelated** with the language models'. It can only be wrong in ways they can't, and they can only be wrong in ways it can't. Independence is the property you're engineering. Neutrality is a story you tell yourself right before you trust a correlated panel.

So the goal was never a view from nowhere. It's *enough genuinely independent views that their errors don't line up* — and the cheapest, most reliable way to guarantee a view doesn't share the others' errors is to build it on a different substrate entirely.

## None of this is new — except where it's applied

The pieces are old. Ensemble diversity, query-by-committee, the wisdom of crowds needing independent members — all well-trodden. What's underused is applying them honestly to **LLM ensembles**, where it's tempting to assume that different models are independent voters. They're not. The moment "multi-agent" became fashionable, the independence assumption got smuggled in unexamined. The discipline is to *check* that your members fail differently — and if they don't, to add one that does.

## What I'd caution

- **A non-LLM member only covers what's formalizable or structural.** It triangulates the language models; it doesn't replace them. Most real questions still need the minds that understand meaning.
- **Independence has to be verified, not assumed.** Adding a "different" member proves nothing until you've seen it disagree on cases where the others agreed. If it never breaks the consensus, it isn't buying you independence — it's just latency.
- **The structural reader has real blind spots.** It mistakes well-connected for true, and frequent for important. It's a *different* bias, deployed on purpose — not an oracle.

---

*This note describes a personal system built as independent R&D. The architecture and methodology are my own; happy to discuss or demo. — Craig Allsopp*
