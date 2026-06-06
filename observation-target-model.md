# The Observation-Target Model: name a system by what it watches, not what it is

*Part of a series of architecture notes from systems I've built as independent R&D.*

## How monitoring surfaces multiply

You build a dashboard to watch one thing. It's useful. So you build another to watch a second thing, and a third for something else. Six months later you have five dashboards, each built fresh, each slightly different in layout, refresh model, and how it defines "healthy" — and adding a sixth means building a sixth bespoke thing from scratch.

The sprawl isn't a discipline failure. It's what happens when each surface is conceived as *its own system* rather than as *one system pointed somewhere new*.

## The reframe

Stop naming these systems by what they **are** ("the build dashboard", "the status page", "the audit viewer") and start naming them by what they **observe**.

Underneath, they are the *same shape*: ingest some state, evaluate it against expectations, surface what needs attention. What differs between them is only the **observation target** — the slice of the world each one watches.

```
                    ┌─────────────────────────────────┐
                    │   one engine (the shape)         │
   target A  ──────►│   ingest → evaluate → surface    │──────►  view of A
   target B  ──────►│                                  │──────►  view of B
   target C  ──────►│   (target is a parameter,        │──────►  view of C
                    │    not a rebuild)                │
                    └─────────────────────────────────┘
```

Once you see it this way, a new surface isn't a new system. It's a new *target* plugged into the engine you already have. The thing that changes is small and declarative — what to watch, what "healthy" means for it, how to render it. The engine doesn't change at all.

## Plug in, don't spawn

The discipline this gives you is a single rule: **when you need to watch something new, plug a new target into the canonical engine — don't spawn a new bespoke system.**

That one rule is what keeps the surfaces consistent. Because they share an engine, they share behaviour for free: the same refresh model, the same way of flagging stale data, the same notion of what an alert is. Users learn one surface and they've learned all of them. And the marginal cost of a sixth surface drops from "build a dashboard" to "describe a target."

The naming convention enforces the discipline. If a surface is *named for its target*, then "I need a new dashboard" naturally rephrases as "I need to observe a new target" — and the second phrasing points you at the engine, not at a blank file. The vocabulary does the steering.

## A concrete shape

A target is a small description, roughly:

- **what state to read** (where the data lives, how to pull it)
- **what "healthy" means** for this target (the expectations to evaluate against)
- **how to surface it** (what a human needs to see at a glance, what counts as needs-attention)

The engine takes any target matching that shape and produces a consistent surface from it. Watching something new = writing one more of these descriptions. No new framework, no new layout language, no new alerting logic — those live in the engine, once.

## Why this matters beyond consistency

1. **Marginal cost collapses.** The first surface is real work. The tenth is a config. That changes what's worth observing at all — when a new surface is cheap, you watch things you'd otherwise have left dark.
2. **Coverage gaps become visible.** When systems are named by target, "what aren't we observing?" is a question you can actually ask and answer, because the targets are an enumerable list rather than a pile of one-off dashboards.
3. **Improvements compound.** Fix the engine — better stale-data handling, a clearer alert model — and *every* surface gets it at once, instead of having to be back-ported into five separate builds.
4. **The mental model is small.** One engine plus a list of targets is far easier to hold in your head than five independent systems, and far easier to hand to someone else.

## What I'd caution

- **The engine has to actually be general.** If "plugging in a target" keeps requiring engine changes, you don't have an engine — you have a pile of special cases wearing a shared name. The test is whether a genuinely new target needs zero engine edits.
- **Don't force unlike things into one shape.** The model works when the surfaces really are the same shape (ingest → evaluate → surface). Something with a fundamentally different interaction model shouldn't be jammed into the engine for the sake of tidiness — that's how a clean abstraction rots.
- **A shared engine is a shared blast radius.** One engine behind every surface means a bug in the engine is a bug everywhere. That's the flip side of "improvements compound" — so the engine earns a higher test bar than any single surface would.
- **Resist the bespoke pull.** There's always a reason *this one* surface deserves to be special. Sometimes it's real; usually it's the start of the sprawl you were trying to avoid. Make "special" earn it.

---

*This note describes a personal system built as independent R&D. The architecture and methodology are my own; happy to discuss or demo.*
