# Two Stores: why raw capture and curated knowledge should not be the same store

*Part of a series of architecture notes from systems I've built as independent R&D.*

## The temptation

When you build a system that remembers things, the obvious move is one clever store: everything goes in, and the store is smart enough to keep the good stuff and quietly handle the rest. One database, one source of truth, done.

It doesn't hold up. A single store is quietly being asked to do two jobs that pull in opposite directions, and it can only ever be good at one of them.

## The two jobs are in tension

**Capture** wants to be *permissive*. Record everything, exactly as it arrived, and never lose anything. The whole value of a capture log is that you can trust it to be complete — if it's been filtered, it's not raw any more, and you've lost the thing that made it worth keeping.

**Curation** wants to be *selective*. Only the things worth keeping, cleaned up, deduplicated, and structured so you can actually query them. The value of a curated store is precisely that it *has* been filtered — it's the signal, with the noise removed.

Ask one store to do both and you're forced to choose at write time:

- Gate at the door, and your "complete" capture log isn't complete — it's missing whatever the gate rejected, including the things you didn't yet know you'd want.
- Don't gate, and your "curated" knowledge is full of raw noise — and now every query has to re-filter on the fly, forever.

There's no setting that satisfies both. The conflict is structural, not a tuning problem.

## What I built instead

Two stores, with a one-way flow between them:

```
   raw input  ─►  [ VERBATIM STORE ]  ─► curation ─►  [ CURATED STORE ]  ─► queries / answers
                  append-only                          gated, structured
                  never gated                          deduplicated
                  the safety net                       the thing you read
```

- The **verbatim store** is append-only and never gated. Everything that comes in is written down as-is. It is not what you query for answers — it's the record you can always fall back to.
- The **curated store** is gated and structured. Things earn their way in through whatever quality bar you set. This is the store the rest of the system actually reads.
- Curation flows **one way**: verbatim → curated. The curated store is *derived*, never the origin.

The payoff is in that last property. Because curation is a derivation from a complete raw record, **a curation mistake is recoverable.** Set the bar too high and drop something useful? It's still in the verbatim store; re-run curation and recover it. Get the structure wrong? Re-derive. The raw record is the ground truth that makes every downstream decision reversible.

Try to recover from a mistake in a single permissive-or-selective store and you can't — the information you'd need to recover from is the information the store's one setting already threw away (or never cleaned).

## A concrete way to feel it

Imagine a system that captures a stream of events and surfaces the important ones.

- **One store, gated:** an event that looked unimportant at capture time gets dropped. Two weeks later you realise that class of event mattered after all. It's gone. You cannot go back and re-decide, because the raw events were never kept.
- **Two stores:** the same event was written verbatim regardless of whether it looked important. Your curation logic was wrong, but the data wasn't lost — you change the rule and re-derive. The mistake cost you a re-run, not the data.

The two-store version lets you **be wrong about curation safely.** That is the entire point. You will be wrong about what matters — at capture time you don't yet know — so the architecture should make that survivable.

## Why this matters beyond tidiness

1. **Curation becomes reversible.** Your filtering rules can evolve because the raw record outlives any single version of them.
2. **The gate can be strict without fear.** A high quality bar on the curated store is safe precisely because nothing rejected is lost — it's still in verbatim.
3. **Provenance is free.** Every curated item traces back to a verbatim origin, so "where did this come from?" always has an answer.
4. **Debugging is possible.** When the curated store says something surprising, you can diff it against the raw record and see exactly what curation did.

## What I'd caution

- **The verbatim store grows without bound.** That's by design, but plan for it — cheap append-only storage, time-based partitioning, and an explicit (long) retention policy. It's a log, not a working set.
- **Coverage has a start date.** A verbatim store only contains what it captured *after* it existed. Anything from before is gone for good — so stand the capture layer up early, before you think you need it. (I learned this the slightly painful way: the raw layer doesn't cover the period before I built it, and that gap is permanent.)
- **One-way flow is a rule, not a suggestion.** The moment the curated store can write back into verbatim, you've collapsed them back into one store and lost the guarantee. Keep the arrow pointing one way.
- **Don't query verbatim for answers.** It's the safety net, not the working surface. If you find application code reading the raw store directly, that's a smell — it means curation isn't surfacing something it should.

---

*This note describes a personal system built as independent R&D. The architecture and methodology are my own; happy to discuss or demo.*
