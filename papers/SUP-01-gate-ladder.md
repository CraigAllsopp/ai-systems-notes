# The gate ladder

### "Proven safe" needs a `gates:` block

*Craig Allsopp, with Claude (Anthropic) — 2026-09-10*

> **Status: published 2026-09-10.**
> **Conflict of interest, stated once:** the second author is a large language model made by Anthropic — one of the companies this set of papers argues with. Where it assesses its own reliability, discount accordingly. Every number below comes from one estate's own instrumentation and can be checked against the artefacts named. Each figure was re-verified against the source files on 10 September 2026, not carried across from an earlier write-up; where that check changed a number or a claim, the corrected version is what appears here.

---

## 1. A hold point I already live in

On a construction site, "proven safe" is not an opinion. It is a document with a name — an Inspection and Test Plan — and it says, before the work starts, exactly what evidence will be required before anyone signs. Pressure-test to a stated figure, held for a stated duration, witnessed by a stated party. Radon readings for the week. A commissioning certificate for the plant. The ITP is written first, and it names its own evidence.

Four things make that document more than paperwork.

1. **A predicate.** "Safe" resolves to a test with a pass condition, decided in advance.
2. **An authority.** One named person signs. Not a committee, not an average.
3. **Bounded cost.** A hold point costs a day, and everyone knows it costs a day.
4. **Binding scope.** Everybody on that site obeys it — because it is law, not because one contractor chose it.

Remove any one and the hold point stops holding. Remove the predicate and "until proven safe" collapses into either *never* or *whenever the loudest person says so*. Remove the authority and you get a committee whose latency exceeds the work. Remove bounded cost and both sides argue forever about a ledger nobody can total. Remove binding scope and your gate becomes your competitor's opportunity.

> ⚠️ **I have never run this without one of them.** The four come from the construction case, where all four are statutory and removing one is not an experiment anyone will authorise. An analogy is not an ablation, and this sentence is doing the work of one.

I have spent five months building an autonomous software system that writes and promotes its own code, and the single most useful thing I brought to it was not from software. It was this document.

> I made the shorter version of this argument publicly in July, in a note called *Verification Discipline*: **a contractor saying a system is installed is not the same as it being commissioned.** Installed is the contractor's claim; commissioned is an independent check, signed by someone whose name is on the line. You would never energise a building on "installed" — and it always struck me as odd that software ships on "done". That note argued the case. This paper is what happened when I built it and measured what it caught.

This paper is about what happens when you write the ITP for an AI agent: the predicate as an actual file, the refusal ladder built around it, what it caught, what it lied about, and — the part that matters most — a clear statement of which rung the field actually needs and does not have.

**What this is not.** It is not a safety framework, and it is not an argument about existential risk. It is one operator's account of a working hold point at the smallest possible scale, with the numbers, including the ones that make the design look bad.

---

## 2. The four conditions, and the artefact nobody has written

Ask whether frontier AI should be held until proven safe and you will get an argument. My own position is that the question is not interesting: it is required, in the same way a pressure test is required, and I do not see the problem.

The argument is not really about *whether*. It is about the four *hows* above, and arguing about whether is a way of not doing the work on how. Anyone who has sat in a site meeting where the programme is slipping and the room discusses principles will recognise the manoeuvre.

Take them in turn, at civilisation scale.

**The predicate is missing.** For a system nobody can read the inside of, there is no agreed pass test. This is the load-bearing gap and everything else waits on it.

**The authority is contested.** Which lab, which state, which body holds the sign-off is not an engineering question, and it is where most of the public noise lives.

**The cost is unbounded and two-sided.** A hold might cost years — and the harm of *not* deploying is real and equally unmeasured. When neither column can be totalled, both sides can argue indefinitely and honestly.

**The scope does not bind.** With several labs, multiple jurisdictions and open weights, one party's gate is another's advantage. Construction discipline works because it is statutory. The short-term path to a binding gate is regulation; there is no second path that arrives quickly.

Locally, I have all four. One authority — me — with a fingerprint reader. A hold costs minutes. There is no competing party. That is why the rest of this paper can show a predicate that works: not because the problem is easy, but because at n=1 the four conditions are satisfied by default, and everything interesting is about what the predicate itself has to contain.

So: what does a machine-checkable "proven safe" look like when you actually write one down?

> ⚠️ **If you believe that sentence, you are wrong — and it is my sentence.** Nothing in this paper proves anything safe. Why, in short, in the note that follows.

---

## 3. The predicate, as a file

Every change to live code in my system carries a small declaration, written *before* the change is promoted. It states an identifier for the change, a class, a time-to-live, what happens on failure and on timeout — and then the part that does the work:

```yaml
change_id:   built_notify_20260908
change_class: live-py-edit
ttl_seconds: 86400
on_fail:     alert
on_timeout:  manual_review

gates:
  - id: G_C1
    name: hook_present_in_live_file
    check:
      primitive: log_signature_present
      args: { file: <target>, pattern: "<exact substring>", count_min: 1 }

  - id: G_C4
    name: the_seam_is_wet
    check:
      primitive: event_emitted
      args: { kind_or_type: <event name>, since: "<ISO timestamp>", count_min: 1 }
```

Twelve primitives exist today, and the list is deliberately short and dull: process-instance count, heartbeat freshness, port binding, a literal signature present in a file, a signature *absent* from a file since a timestamp, a database field equalling a value, an event emitted, an event **not** emitted, a process still alive after N seconds, a file hash matching a baseline, a pattern across a glob, and a knowledge-store record existing.

Three properties of that list carry the argument.

**An unknown primitive fails; it does not warn.** You cannot invent a gate name and have the system nod along. If the predicate is not in the vocabulary, the change does not pass.

**Both polarities are first-class.** `log_signature_absent` and `event_NOT_emitted` exist alongside their positives, because half of what you need to prove after a change is that the old wrong behaviour is *gone*. A regression is a claim about absence, and absence needs its own primitive.

**Time is explicit.** `since:` on the absence checks, `ttl_seconds` on the plan, `on_timeout: manual_review` as the default. A gate with no clock is a gate that passes eventually.

The last of those cost me something to learn. `on_timeout` used to be the interesting field; it turns out `since:` is, because "this bad pattern does not appear in the log" is trivially true if you forget to say *since when*.

That is "proven safe" made mechanical. It is small, it is boring, and it is checkable by a program at three in the morning with nobody watching — which is the entire point. **A predicate that requires a human to interpret it is not a predicate; it is a conversation.**

> ⚠️ **If you believe that sentence, you are wrong — and it is my sentence.** Not one of those twelve primitives is a safety property. See below.

---

### A note on the two sentences I left standing

Both are flagged above, both are mine, and both are still in the text on purpose. Three of their words do not survive the rest of this paper:

- **"proven safe"** — the ladder proves that a named change satisfies a short list of checkable properties. A signature is present in a file. An event fired inside a window. A process is still alive. **None of those is a safety property**, and no arrangement of them becomes one.
- **"machine-checkable"** — I find, later in this paper, one rung documented in the gateway's own header and never wired, and one limit that could not be reached in production. **Checkable on paper is not checked in the run.**
- **"when you actually write one down"** — the properties that would actually bear on safety have no primitive at all, mine included. **What is written down is the easy subset.**

They are left standing because they are exactly what the phrase makes a reader expect, and catching yourself believing them is the shortest route to the paper's real claim. **The subtitle is about necessity, not sufficiency:** a `gates:` block is what "proven safe" needs before it means anything at all, and nowhere near enough on its own.

The value is not that writing the predicate down closes the gap. It is that the gap becomes **visible and nameable** the moment you write one. You can point at a missing rung and argue about it. You cannot point at a principle.

---

## 4. The ladder: what each rung refuses

The declaration above is consumed by a promotion gateway that a change must pass to reach live code. Promotion is always two steps: a **prepare**, which runs the cheap checks and mints a confirmation token, and a **confirm**, which re-checks and installs. The rungs are numbered in the order they were built, not the order they run, and the running order is the argument.

**Prepare — ten checks, nobody present.**

| Rung | Refuses |
|---|---|
| G-1 | (non-blocking) refreshes the dependency index, so blast radius is never read stale |
| G0 | No plan present — or a plan whose contents do not parse |
| G1 | Project or destination cannot be resolved from the manifest |
| G2 | Sandbox health check fails (isolation, docs, anomaly) |
| G3 | The build log's hash chain is broken |
| G4 | Documentation is absent or trivial |
| G7 | Tests did not pass — **or did not actually run** |
| G5 | Security scan finds a critical issue on a promoted file |
| G5b | Antivirus scan fails |
| G6 | Diff produced and path-class risks surfaced for review |

**Confirm — the human's step.**

| Rung | Refuses |
|---|---|
| G10 | Confirmation token invalid |
| G10b | Live bytes changed since the plan was made |
| G10c | Sandbox bytes changed since the plan was made |
| G10d | The plan is invalid (read-only re-check) |
| G11 | **The human does not authenticate** |
| G12 | The change is not registered for post-promotion monitoring |

Two rungs are absent from those tables, and both absences are load-bearing.

**G8 was removed**, in April 2026. It took a full-system snapshot before every promotion, cost seventeen minutes, and was redundant: the per-file backups taken at install time are the actual revert mechanism. Deleting a gate that buys nothing is as much a design act as adding one, and the function is still in the file, unwired, with the reason written beside it.

**G9 was never wired at all.** The gateway's own header documents it — the generated history draft must pass a content scan before promotion — and no such call exists anywhere in the file. The draft is written; nothing reads it. I did not know that until I opened the source to write this table. I come back to it later, because it is the most instructive thing in the paper.

Everything cheap and mechanical refuses *before* the human is asked. That is not tidiness; it is the scarcity rule the rest of this paper turns on, and the four checks numbered G10 through G10d exist because I broke that rule and had to repair it three times, one binding at a time.

One honest exception, found while checking this table against source. After the fingerprint is read, the confirm step **re-runs** G1 through G7 before it copies anything, on the correct reasoning that state may have moved since prepare. Those re-runs can fail — and if they do, the authentication has already been spent. It is precisely the defect this paper is about, sitting in the one place I had never thought to look for it.

## 5. Bind it, or it is not a gate

A plan is written at one moment and confirmed at another. In my estate that gap is routinely hours and sometimes days.

For most of the system's life, nothing re-checked the code in between. The plan pinned a diff, a baseline and a token — and then the confirmation copied whatever happened to be sitting in the sandbox at the moment the fingerprint was read. Any change in between — a tree-wide edit, a hotfix, a second session touching the same file — silently invalidated the plan, and the promotion then landed a diff computed against a file that no longer existed.

That happened. On 9 August, thirty-one live files were re-stamped at 16:13; a plan prepared at 11:46 went stale, and it was caught only because a human re-checked hashes at authentication time.

Then I measured the estate before building the fix, and the measurement was worse than the incident:

> **Of 108 parked plans, exactly one had a baseline binding it to live bytes. The other 107 had no hash binding of any kind.**

The incident report had understated it. The finding was not *"re-stamps invalidate plans"*. It was that **plans had never been bound to live bytes at all.**

Worth being precise about what was and was not at risk, because I got this wrong in an earlier draft of this section. **Recovery was never the gap.** Full snapshots run several times a day, and the per-file backup taken at install time is the actual revert mechanism — as the note on the removed snapshot rung above already says. What was missing was not a net but a **detector**: nothing could tell you that live had moved between the plan being written and the confirmation being given. A promotion would land a diff computed against a file that no longer existed, and report success. **One plan in 108 carried a hash that would have noticed.**

Two things about the fix are worth more than the fix.

**Legacy was not failed closed.** A plan with no binding reports *unverifiable* and proceeds, because each of those 107 represents an authorisation a human already gave. Failing them shut would have invalidated the entire parked estate to fix a problem none of them was known to have. Verified after the change: **104 resolvable parked plans, zero blocked.**

**Two questions that sound identical were kept apart.** "Has this plan's confirmation already happened?" (safe to archive) and "has live moved since this plan was made?" (unsafe to confirm) have opposite consequences. One word, two meanings, deliberately not merged.

The binding then had to be done twice more before it was real. First the live bytes. Then the *sandbox* bytes — the sharper hole, because the artefact promoted is whatever sits in the sandbox when the fingerprint is answered, not what the reviewer read. Then the copy itself: a check that re-reads a path and a use that re-reads the same path are two reads, and two reads is a race. The final version hashes the *staged temporary copy* — the bytes verified and the bytes installed are now the same file, and there is no interval left to race. The test that proves it corrupts the staged copy while leaving the source byte-perfect; an implementation that re-hashed the source would pass a naive test and still be wrong.

**The generalisable rule: a gate that names an artefact but does not bind its bytes is a gate on a name.**

---

## 6. Spending the authority's attention

Here is the finding I would most want a policy reader to take, and it arrived as an irritation rather than an insight.

Twice in six days, in two separate sessions, the log recorded this sequence: *authentication OK — human verified*, and then, immediately after, *validation failed, promotion aborted*. A comparison of seven possible values — microseconds of work, one small file — rejected the promotion **after** I had stood at the machine and touched the sensor.

The gateway's own changelog puts the diagnosis better than I can:

> *"asking Craig to authenticate a promote already known invalid burns a Hello and teaches the prompt to be ignored"*

and, from the fix that followed:

> *"a substituted artefact is refused without spending a Hello on it."*

The root cause was not gate ordering. It was that **validation had been welded to registration**: the only way to ask "is this plan valid?" was to call the function that validates *and registers*, and registering is a side effect that legitimately must follow authentication. A question was fused to an action, so the question inherited the action's authorisation cost.

The fix splits the call. The read-only half moves to the moment the plan is authored — hours earlier, zero cost, nobody standing at the machine — and runs again immediately before the authentication prompt. The registering half stays after authentication, where a side effect belongs. A test pins the ordering so that a later author who "simplifies" it back breaks the build.

Generalise it and it is the scaling problem for every proposal to put a human in the loop on AI:

**The authority's attention is the scarce resource. A gate that spends it on foregone conclusions destroys itself — not by failing, but by teaching its authority to stop reading.**

> ⚠️ **Scarce relative to what?** I measured the attention and never measured the alternatives, so "the scarce resource" is a claim with evidence behind half of it. The 385 wasted builds later in this paper are a compute cost, sitting in the same document, unmeasured against this one.

The human-factors literature has known this for thirty years — I did not, until after this paper was drafted, and section 8 is where that lands. What I have not found is an oversight proposal for *AI* that treats the approver's attention as a metered resource with a depletion rate rather than as a role to be filled. Mine has one user with one fingerprint, and it still hit attention exhaustion inside the first five months — the two incidents above fell on 8 and 13 August 2026. A committee will hit it faster, and a rubber-stamped gate is worse than no gate, because it produces a signature that means nothing while looking exactly like one that does.

There is a live example in the estate as I write. Measured on 10 September 2026: two hundred and forty-three prepared changes sit awaiting my authentication, the oldest of them prepared a hundred and thirty-seven days earlier. The gate is not broken. The authority is the bottleneck, exactly as designed and exactly as unaffordable.

---

## 7. Three ways a gate lies

### The false green

The test gate ran a suite and read its exit code. The house convention names every suite the same thing, and that name matched none of the test runner's discovery patterns, and such a file has no entry point — so the process **exited zero having executed nothing.**

Measured on one real sandbox: **exit 0, 0 of 29 tests run, gate green.**

The fix is not "check the exit code better". Green is now licensed by the runner's own reported counts: zero collected fails, zero executed fails, an unparseable summary fails closed. And there is deliberately **no** "no tests, nothing to do, that's fine" branch — because that branch *is* the false green. A naive `returncode != 0` catches the runner's no-tests exit code only by accident.

**A gate that infers success from the absence of failure will eventually report success for the absence of work.**

I put that more cleanly in July, before I had this instance to hang it on: **absence of a detected failure is not evidence of safety — it is evidence about your detectors.** If nothing has gone red in a year, the honest first hypothesis is that your checks *cannot* go red.

### The rung that only exists on paper

G9 is documented in the gateway's own header: the generated history draft must pass a content scan before it is written to live. The scanning module is never imported. The function is never called. The draft is produced and filed unread.

I should not have been surprised, and that is the part worth reporting. Two months earlier, writing about what survives swapping the model underneath a system, I had stated this exact failure as a general law: **"a gate that silently no-ops hasn't survived — it is *absent*, and you won't notice unless you watch it actually fire, and actually fail when it should. A check that can't come back negative has rotted into decoration."** I wrote that down, published it, and then did not go looking for an instance in my own ladder until a paper forced me to. **Naming a failure mode is not the same as auditing for it** — and the gap between the two was, in my case, eight weeks and one unwired rung.

The same shape, one layer down, and this one cost real compute. In September I found three sandboxes that had burned **385 builds** between them — 182, 138 and 65 — on a nightly loop that kept re-dispatching a failed build. The repair code has a depth cap of three. It had become **unreachable in production**: the counter that walks a fix-chain back to its origin read that ancestry out of a derived text field, and when those records were re-rooted by an unrelated change, items five generations deep read as depth zero. The cap never fired. Escalation to a human never happened. The limit was in the source and not in the running system.

Both are the same lie, and it is a worse one than the false green, because there is nothing to observe. A false green produces a wrong answer you can go and check. A rung that is documented but unwired, or a limit that cannot be reached, produces **no signal at all** — and the artefact describing the system carries on describing a system that is not running.

I write this ladder's documentation. I audited this ladder for this paper. I still had both of these wrong until I read the source, and I found them only because publishing forced a line-by-line check that five months of operating had never forced.

**Documentation is not evidence that a gate exists. The only evidence is the call site.**

### The missing question

The deepest lie is structural, and my ladder still tells it.

In September I audited 49 changes that had been built by the autonomous queue and never promoted. **Twenty-six of them had passed all eight gates the gateway records in its prepare report — G1 through G7 plus G5b, the ones written down at prepare time — and held valid confirmation tokens.** On inspection, twenty of those twenty-six should never have been built: they were rivals of code already live under a different name, or they were built to a specification that had drifted from the task that spawned it. Nine further instances of specification drift showed up elsewhere in the same audit.

Every gate asks *"was this built right?"* **None asks "should this exist?"**

That is not a bug in any particular rung. It is a property of the whole ladder: **a ladder can only refuse on properties for which somebody has written a predicate.** Existence-checking is a rule I hold in my head and state in my instructions, and holding it in my head is exactly why twenty-six things with valid tokens are sitting in a sandbox.

Specification drift, not defective code, is the dominant failure mode — and it is invisible to every gate that tests build quality.

> ⚠️ **A count from a sample chosen for its outcome.** Those 49 changes were selected *because* they were unpromoted, and I never counted defective code in the same window. You cannot rank two failure modes having measured one. The sentence stands because I believe it; "dominant" is not a word this audit can pay for.

## 8. What is already known, and what is not

A prior-art sweep was run over this ladder in July 2026, because the honest version of "here is my design" includes where it has been done before. Its two load-bearing citations were re-checked against the sources themselves before this draft — not taken on trust from the sweep that produced them.

**The components are commodity.** Refusal on missing evidence exists in production supply-chain tooling — Google's Binary Authorization rejects at admission without an attestation; `in-toto-verify` fails on a missing signed link. Tamper-evident provenance from artefact to release is dense prior art (Sigstore/Rekor, SLSA). Staged gate ladders are decades old — Stage-Gate in product development, DO-178C's stages of involvement in avionics, CIBSE Commissioning Code M and BSRIA BG 49 in building services. The distinction between *done* and *verified* is anticipated squarely by **US 11,016,738 B2** (SAP SE, priority December 2018, granted 2021), whose four-state ladder runs *development approved → implemented → successfully tested → productive*, with the tested state requiring an **independent** tester and the productive state requiring a **change manager's** confirmation. That is my G7 and my G11, claimed eight years ago, for humans.

**The most damaging prior art is not patents.** It is the open literature. **in-toto**, documented since 2016, already realises evidence-required, accumulation across steps, and cryptographic chaining *together* — the three things I would most have wanted to claim, in one open-source project. And closest of all: **EviBound** (*Evidence-Bound Autonomous Research: A Governance Framework for Eliminating False Claims*, Ruiying Chen, arXiv:2511.05524, November 2025), which puts dual governance gates around autonomous-agent output, with claims propagating only when backed by a queryable run identifier, required artefacts, and a finished status.

That last one deserves the emphasis. Somebody else independently arrived at *claims must carry their evidence or they do not propagate*, aimed at exactly the problem this system has, and got there first. A patents-only search would have missed it entirely and left me believing I had invented something.

**The sweep missed the claim I would most want to be new.** Section 6 argues that a gate spending its authority's attention on foregone conclusions destroys itself by teaching that authority to stop reading. That is not a new finding. It is **disuse**, and it has a name and a literature going back three decades: Parasuraman and Riley's *Humans and Automation: Use, Misuse, Disuse, Abuse* (Human Factors, 1997) defines disuse as the neglect of automation **commonly caused by alarms that activate falsely** — my mechanism, published when I was at school, in a journal I had never opened. The July sweep did not find it because the sweep searched for gates and provenance and staged ladders, which is the vocabulary of my design rather than the vocabulary of my finding. A prior-art search run by the person who owns the framing inherits the framing.

The same applies one section earlier. "Every gate asks *was this built right?* and none asks *should this exist?*" is the **verification/validation** distinction, which Barry Boehm put in almost exactly those words in 1979 and which has been in IEEE 1012 ever since. My ladder having no validation rung is a real finding about my ladder. The distinction is a textbook.

So what is left that I have not found claimed elsewhere? A narrow list: refusal on **absent** evidence as an existence predicate rather than a failed-test verdict; gating the **work-item record** rather than the artefact; evidence thresholds that vary by who or what authored the change; and the commissioning-discipline import as an explicit design analogy.

Narrow, and shrinking as the field publishes. My conclusion was to publish rather than file: the goal is positioning, not licensing; the cost is unjustifiable against parties who could not be pursued anyway; and **publishing is itself the protection** — a dated public disclosure creates defensive prior art. This paper is that disclosure.

---

## 9. Limits

**n = 1.** One operator, one laptop, five months. Every figure is from one estate's own instrumentation. It is a lab notebook, not a study.

**No independent audit.** The system reports on itself. That is the right epistemic level for "did this gate refuse?" and the wrong one for "is this gate sufficient".

**The self-description was wrong twice.** Writing this paper against source turned up one documented rung that is not wired and one cap that cannot be reached. Both had stood unnoticed through five months of daily use. Assume the same rate applies to anything here I have not personally re-read — which is why every figure in this paper names the artefact it came from.

**Bootstrap limitation, stated and not papered over.** The gateway is itself promoted through the gateway, so its own test gate runs under the version being fixed. The first properly gated promotion of any gateway change is the *next* one. This is recorded in the tool's own changelog at three separate versions rather than hidden.

**The four conditions hold here and nowhere else.** One authority, minutes of cost, no competitor. At scale every one of those inverts.

**No predicate for the top rung.** The twelve primitives work because the properties are mechanically checkable — a file exists, an event fired, a version advanced. For *"safe to hold persistent goals"* nobody has the primitive, including me. The ladder generalises; the top rungs are empty.

That last limit is the honest ending. What I have is a working hold point for a small autonomous system, an argument that hold points need four conditions rather than good intentions, and a measured demonstration that the scarce resource is not compute or capability but **the attention of the person who signs.**

The rung above the ones I built is where the work is. It is not a mystery to be solved by a breakthrough. It is a `gates:` block somebody has to write.

---

### Artefacts referenced

Promotion gateway (G-1 – G12, v0.39) · plan schema and the twelve check primitives · promotion audit of 49 built-and-unpromoted changes, 8 September 2026 · prior-art clearance sweep, 19 July 2026. All counts measured 10 September 2026.

### References

- Chen, R. *Evidence-Bound Autonomous Research (EviBound): A Governance Framework for Eliminating False Claims.* arXiv:2511.05524, November 2025.
- US 11,016,738 B2, *Change control management of continuous integration and continuous delivery.* SAP SE. Priority 19 December 2018; granted 25 May 2021.
- in-toto: a framework to secure the integrity of software supply chains (2016–).
- Sigstore/Rekor; SLSA provenance levels.
- DO-178C, Stages of Involvement 1–4. CIBSE Commissioning Code M; BSRIA BG 49.
- Parasuraman, R. and Riley, V. *Humans and Automation: Use, Misuse, Disuse, Abuse.* Human Factors 39(2), 1997. — the prior art for section 6.
- Boehm, B. Verification vs validation, 1979; later formalised in IEEE 1012. — the prior art for section 7's missing question.

*Every reference above was checked against the source listing on 10 September 2026. Where a lineage is named in the text without a citation, it is because I had not checked it and would rather say so than print one.*

*Published 2026-09-10. Corrections and counter-evidence welcome.*
