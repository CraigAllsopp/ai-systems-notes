# Model-Independent Discipline: why your system shouldn't care which model is inside it

*Part of a series of architecture notes from systems I've built as independent R&D.*

## The assumption hiding in "just use a better model"

The obvious way to make an AI system better is to put a better model in it. A new model launches, you swap it in, and everything you've built gets smarter for free. It's such a natural move that the model quietly becomes the thing you think is doing the work — and the scaffolding around it gets treated as plumbing.

That assumption is worth poking, because it has an uncomfortable corollary. If the model is what's doing the work, then your system is only ever as reliable as this week's model — and it inherits every quirk, regression, and behavioural shift that arrives with the next one. You don't have a system. You have a model with some wrapping.

## The test: swap the model and see what moves

There's a clean way to find out which one you've built. Change the model underneath and watch what happens to the *behaviour* — not the prose, the behaviour. Do the same gates fire? Do the same checks come back negative when they should? Does work still refuse to be called "done" until something independent confirms it? Or does the discipline quietly soften, because it was really living in the old model's habits all along?

If swapping the model changes how your system *behaves*, the model was doing the work. If it doesn't, the discipline is yours.

## What I saw

When the new model — Fable 5 — launched, I pointed the same setup at it that had been running on the previous one. Same gates, same separate verification states, same standing rules about provenance and closure. I wasn't testing the model's cleverness; I was watching whether my own scaffolding held when the thing inside it changed.

It held. The discipline behaved identically: a claim still couldn't reach "verified" on the new model's say-so alone, the checks still fired, and closure language was still earned rather than assumed. The parts that were a property of *the structure* didn't notice the swap, because they were never the model's to begin with.

Plenty *did* change — just not the discipline. The new model was faster, and its output was a little sharper. It also burned through context markedly quicker, which carried a real cost I'll come back to. But those are differences in the *engine* — its speed, its polish, its appetite — the kind of thing you'd expect to differ between any two models. The rails sitting around the engine didn't move. The model changed; the discipline didn't.

That's the whole note: the interesting fact isn't that a model changed — it's that it *could*, the things you'd expect to differ did, and the discipline you actually engineered carried straight across.

## Why this matters beyond a tidy result

1. **Your system outlives any one model.** Models launch, deprecate, get repriced, and shift behaviour between versions. If your discipline is external to the model, none of that forces a rebuild — you adopt the new model for its strengths and your rails come along unchanged.
2. **It's the test of whether you built a system or a prompt.** A clever system prompt travels *with* the model and its quirks; it isn't independent of it. Discipline that survives a model swap is, almost by definition, the part you actually engineered — gates, state, and checks that live outside the model.
3. **It de-risks the upgrade treadmill.** You can take a new model for what it's genuinely better at without betting your system's reliability on it behaving like the last one. The floor is held by the structure, so the model only has to raise the ceiling.
4. **It's the same principle as the rest of this series.** *Verification Discipline* says the check must be independent of the doer; *The Observation-Target Model* says the value lives in the engine, not the instance. This is that idea turned on the model itself: the doer is swappable, and the system is the part that isn't.

## What I'd caution

- **Model-independent doesn't mean model-identical.** The *discipline* held; that doesn't make two models interchangeable. Speed, polish, voice and cost all differ — so independence buys you the freedom to swap, not permission to stop choosing the right model for the task.
- **If your system is cache-heavy, do the sums before you switch.** A long, stable context you rely on being *cached* — read back cheaply each turn — is the workload a model swap quietly taxes. A newer model routing certain content to a more conservative one is expected, sensible behaviour; nothing to hold against it. But the *mechanics* of that routing carry a cost: each hand-off invalidates the prompt cache, so a context-heavy setup pays to rebuild it. A faster, sharper model can still cost you more, turn for turn, if your architecture leans on a cache the routing keeps resetting. Independence from a *model* isn't independence from its *mechanics*.
- **The discipline has to actually live outside the model.** If your "system" is mostly a system prompt, it isn't independent — it rides on the model's behaviour. The independence comes from machinery the model can't quietly opt out of: external state, gates that block, checks that can return negative.
- **"It ran" is not "it held."** A gate that silently no-ops on the new model hasn't survived the swap — it's *absent*, and you won't notice unless you watch it actually fire, and actually fail when it should. A check that can't come back negative has rotted into decoration, on any model.

---

*This note describes a personal system built as independent R&D. The architecture and methodology are my own; happy to discuss or demo. — Craig Allsopp*
