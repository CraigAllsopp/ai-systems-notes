# Verification Discipline: why "done" isn't "verified"

*Part of a series of architecture notes from systems I've built as independent R&D.*

## The most expensive word in any system is "done"

Someone finishes a piece of work, marks it **done**, and everyone moves on. The trouble is that "done" is a claim made by the person who did the work, at the moment they stopped doing it — and that's exactly the moment they're least able to see what they missed. "Done" means *I performed the work.* It does not mean *the work is correct.* Those are different facts, and the gap between them is where bugs, regressions, and silent failures live.

I came to software from twenty years in M&E and decarbonisation infrastructure, where this distinction is not a nicety — it's the whole job. A contractor saying a system is *installed* is not the same as it being *commissioned*. Installed is the contractor's claim. Commissioned is an independent check that it actually does what it's supposed to, signed by someone whose name is on the line if it doesn't. You would never energise a building on "installed." So it always struck me as odd that software happily ships on "done."

## The reframe: split the claim from the confirmation

A two-state model — *not done* / *done* — bakes in the lie, because "done" is forced to mean both "finished" and "correct" at once. The fix is to refuse to let one word carry both. I run a **four-state lifecycle**:

```
   pending  →  in_progress  →  done  →  verified
              (work begins)   (doer's   (independent
                               claim)    confirmation)
```

- **done** is the *doer's* statement: I have stopped working on this and believe it complete.
- **verified** is a *separate* confirmation that it actually works — produced by something other than the doer's say-so.

The whole point is the seam between the third state and the fourth. "Done" is self-reported. "Verified" has to be earned against an independent check.

## Why the fourth state changes everything

Once "done" and "verified" are different columns rather than one flag, three things you couldn't do before become trivial:

1. **"Done but not verified" becomes a queryable backlog.** You can ask, at any moment, *what claims to be finished but hasn't actually been checked?* — and get a list. In a two-state world that list is invisible; the work is simply "done" and forgotten. Most silent failures are sitting in that gap, unseen, precisely because nothing was tracking it.

2. **The silent-failure class gets caught.** The dangerous failure isn't the one that errors loudly — it's the one that *reports success while having failed.* A process that exits "done" but actually crashed on a path the doer never hit. A deploy that "succeeded" but is serving the wrong thing. Verification is a second, independent assertion that catches exactly the case where the first one lied.

3. **Closure language has to be earned.** "Fixed," "shipped," "handled" — these are verification claims wearing completion clothes. Holding the fourth state separate forces honesty: you may say *done*, but you may not say *verified* until something independent has said so too.

There's a tool that implements the pattern as a plain-Markdown task parser — **[programme-parser](https://github.com/CraigAllsopp/programme-parser)** — where *done* and *verified* are genuinely distinct states, so "what's done-but-unverified?" is a query, not a hope.

## The hidden link: the verifier must be independent of the doer

Here's where this note meets the others in the series. A verification that the *doer* performs on their *own* work is barely a verification at all — it shares the doer's blind spots and the doer's incentive (which is to be finished). For the fourth state to mean anything, the check has to come from somewhere that doesn't share the first state's failure mode: a different person, a test the doer didn't write to pass, an automated assertion, a different kind of reader entirely. *Verification is independence applied to your own claims.* "Done" is what you believe; "verified" is what survives a look from somewhere you don't control.

## What I'd caution

- **Verification has a cost — scale it to the stakes.** Not every task earns a full commissioning. The discipline is to *decide* the verification bar deliberately, not to default to "done" because verifying felt like effort.
- **Self-verification is theatre.** If the same agent that did the work signs off the work, you have a rubber stamp, not a check. The independence is the active ingredient; remove it and the fourth state is decoration.
- **A check that can't fail isn't a check.** If "verified" gets stamped automatically, it has rotted back into "done" with extra steps. The test of a real verification is that it is *able to come back negative* — and sometimes does.
- **Verify the verifier, occasionally.** A stale or wrong check confers false confidence, which is worse than none. The thing that confirms your work also needs confirming, now and then.

---

*This note describes a personal system built as independent R&D. The architecture and methodology are my own; happy to discuss or demo. — Craig Allsopp*
