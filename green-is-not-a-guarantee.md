# Green Is Not a Guarantee: a check that watches the start will report success forever

*Part of a series of architecture notes from systems I've built as independent R&D.*

## A job that ran perfectly for two months and did nothing

A scheduled task on one of my systems fired on the first of the month, every month. It reported no errors. It had a next-run date, a last-run timestamp, and a clean history. By every signal available, it was working.

It was calling a script that had never existed.

Nobody had written the target. The task had been registered ahead of the tool it was meant to run, the tool never got built, and the schedule went on firing into empty air — reporting success by producing nothing, on a monthly cadence, quite happily, forever.

That's not a story about a missing file. It's a story about what the monitoring was actually measuring. The scheduler knew how to answer *did this task start?* It had no opinion whatsoever on *did anything happen?* — and those two questions had silently become the same question in everyone's head, mine included.

## The reframe: liveness is not success

Most monitoring, left to grow on its own, converges on **liveness**: is the thing running, did it fire, is the process up, did the endpoint respond. Liveness is easy to measure, cheap to collect, and reassuring to look at.

It is also the wrong question, and it fails in a specific, nasty direction: **a liveness check reports green through a permanent failure.** Not intermittently. Not with a warning. Continuously, confidently, for as long as the failure lasts.

```
   liveness check          "did it START?"      →  green, forever
   ────────────────────────────────────────────────────────────────
   outcome check           "did it WORK?"       →  can actually go red
```

The distinction sounds obvious written down. It is remarkably hard to see in a system you built yourself, because you know what the job is *supposed* to do, and the green light agrees with you.

## Two shapes of the same failure

The phantom job wasn't alone. In the same morning I found a second monitor on the same system, healthy-looking and wrong in a completely different way:

**The frozen dashboard.** A status file, generated on a schedule, that I read at the start of every working session. The process generating it died — no crash, no traceback, the heartbeat simply stopped — and the file **froze**. It didn't blank, it didn't go stale-flagged, it didn't disappear. It sat there for a day looking exactly like a current answer, and I opened my day acting on incidents that had already resolved themselves hours earlier.

Two different mechanisms. One shape: **each answered a question adjacent to the one that mattered, and the adjacency was invisible from the outside.** *Did the task fire?* is adjacent to *did the work happen?* — and *when was this file written?* is adjacent to *is this information current?*

The frozen one is what I'd flag hardest to anyone building this stuff. A monitor that *crashes* is annoying but honest — you notice. A monitor that *freezes* keeps serving its last good answer, and stale output is indistinguishable from current output unless something is separately checking the generator's own liveness. A dead monitor is more dangerous than no monitor, because no monitor doesn't lie to you.

## What a check that can actually fail looks like

The other half of that morning was more encouraging, and it's the part that says what to build instead.

I was promoting two changes through a gate ladder I'd built earlier — a sequence of mechanical checks that a change must clear before it can reach live code. And the gates kept refusing me.

One gate requires that a change to existing code carries a **characterisation of the old behaviour**, captured *before* editing, so the new version can be proved to behave identically. I hadn't done it in the right order. Refused. I did it properly and tried again — and it refused a second time, because although the characterisation now existed, the test suite the gate runs didn't actually *read* it. I'd written a check that sat alongside the evidence rather than consuming it. The gate knew the difference.

A separate check caught a fake file path in a test fixture. I fixed it, and it then caught the same pattern **in the comment I'd written explaining why I'd avoided fake file paths**.

That's the standard. Not "did the process start" but a check that is **able to come back negative, and does, against a motivated and well-informed operator who is actively trying to pass it.** I wrote those gates, I knew what they tested, I wanted through, and they still stopped me three times in one morning — twice for something real.

The difference isn't cleverness. It's that a liveness check is looking at the *machinery*, and an outcome check is looking at the *product*. Machinery is easy to see and tells you almost nothing.

## What I'd caution

- **Absence of a detected failure is not evidence of safety.** It's evidence about your detectors. If nothing has gone red in a year, the honest first hypothesis is that your checks can't go red — not that nothing has gone wrong.
- **Check the generator, not just the artefact's age.** A timestamp on a generated file tells you when it was written, not whether the writer is alive. The frozen dashboard was well inside its freshness window because it had been regenerated right up until the moment its process died.
- **A check that has never failed is a check you haven't tested.** Deliberately break the thing and confirm the monitor notices. If you can't make it go red on demand, you don't know it can.
- **Watch for adjacent questions.** Every one of the four failures was a check answering a *nearby* question and being read as answering the real one. When you write a monitor, write down the exact question it answers — then write down the question you actually care about, and look hard at the gap.
- **The ones you build for yourself are the most dangerous.** You know what the job is meant to do, so a green light confirms what you already believe. Independent checks matter most where you're most confident.

I'll add the sting in the tail, because leaving it out would be the same failure this note describes: one of the two changes I promoted that morning shipped with a soak gate that only verifies the new code is *present* in the file, not that it *behaves*. It'll pass on its first run regardless. I flagged it as nominal in the change's own record rather than let a green tick imply more than it had earned — which is the least you can do when you've just spent a morning finding out what green is worth.

There's a small tool that implements the counter-pattern for the scheduled-job case — **[cron-logger](https://github.com/CraigAllsopp/cron-logger)** — a persistent fire-log that makes silent cron death visible, by recording what actually happened rather than trusting that a schedule implies an outcome.

---

*This note describes a personal system built as independent R&D. The architecture and methodology are my own; happy to discuss or demo. — Craig Allsopp*
