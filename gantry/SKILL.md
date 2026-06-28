---
name: gantry
description: Build software *through gantry* — operate its gates instead of routing around them. Use when working in a gantry repo (a `.gantry` store, the `gantry` CLI, a PRD/spec to drive to a green). Teaches the carve → clause → checker → discharge discipline; the agent is a proposer the gates check, not an author who declares done.
---

# Building with gantry

> **The green has to mean something.** A passing check must depend on declared inputs, be
> attributable to a requirement, and pass a human ratification gate. Your self-report — "I wrote
> the code," "tests pass," "done" — is *exactly* the thing gantry does not trust.
>
> **You propose; the gates dispose.**

This skill teaches a **constraint**, not a capability. Building with gantry is more work than
writing the code and calling it done — and that extra work *is the point*. If you find yourself
looking for the faster path that skips a gate, stop: that path is the failure mode this whole
system exists to catch.

The command surface below is read from the live binary and may drift. **The binary is ground
truth — run `gantry --help` and `gantry <cmd> --help` and trust that over any command spelled here.**
What does *not* drift is the stance and the workflow shape. Lead with those.

---

## Layer 1 — Why gantry exists

A check is only trustworthy if three things hold at once:
1. **It depends on its declared inputs** — change an input, lose the green (no stale green).
2. **It is attributable** — every green traces to a specific requirement it discharges.
3. **It passed a human gate** — a person ratified the contract the check enforces.

Your training pushes you to *complete the task and report success*. Gantry is built on the premise
that self-reported success is untrustworthy. So your job here is not to produce a green — it is to
**earn one through the gates**, and to report honestly when you can't.

---

## Layer 2 — Your stance: a proposer inside a system that checks you

**You MAY:**
- Draft the decomposition (how the spec carves into per-requirement leaves).
- Draft clause statements (the contract each requirement is judged against).
- Draft checker test sources (the test that runs against real code).
- Write the implementation.
- Run the gates and `discharge`; read and report their verdicts exactly.

**You MUST NOT:**
- **Author the final clause the adversary judges.** You *draft*; the human authors the binding
  contract. (The CLI lets you `clause edit` — that is the draft channel; it does not make you the
  author of record.)
- **Mark an accept or a ratify as a substitute for the gate.** `clause accept` / `checker accept`
  are *adversary* judgments (they require `--adversary <NAME>`; there is no `--force`/`--manual`
  accept). `ratify` is the human seal. You invoke these to *submit to* the gate, never to wave
  something through.
- **Declare "done."** Only `discharge` produces a green. "I finished" is not a verdict.
- **Invent requirements** the spec did not state.
- **Reach for an axiom** to assume away something that should be verified.
- **Route around an obstruction** (fake/stub/hardcode to turn Blocked into green).
- **Touch the frozen verification core / kernel** to make something pass.

Working *with* the gates, not around them, is the job. **An agent that games a gate has failed even
if it produced green.**

---

## Layer 3 — The workflow: carve → clause → checker → discharge

The store is addressed with `gantry --root <store-dir> <cmd>`. Keep the run store **out of the
tree** (e.g. a `/tmp` dir); it is git-backed and not part of the deliverable. Read the exact flags
from `--help` for each step; the *shape* is fixed.

Your loop at every step is: **propose → submit to the gate → read the response → fix the real gap →
resubmit.** Never: propose → declare done.

### The operating model: satisfy as you go

**gantry drives the build. It is not a final gate you run after the work is done.**

Carve the spec into per-requirement leaves first. Then work **one leaf at a time**: take a single
leaf, author its clause, write *just enough implementation to discharge that leaf*, earn its green,
and move to the next. Each piece of code you write exists to discharge a claim you have already
established. The carve is your worklist; let it drive the order of the work.

You should **never** accumulate a large body of unverified implementation and submit it to gantry at
the end. When a gate rejects work that was built all at once and verified last, the implementation
was written toward an idea of "done" rather than toward the clauses — so the fix is a large, late
rewrite. Satisfying each leaf as you go keeps every increment small and grounded: a rejection is
local to one leaf and cheap to fix, and there is no end-of-run rewrite. The friction lands
continuously and small, instead of all at once and large.

The steps below (carve → clause → checker → discharge) describe the gates each leaf passes through —
**not** a phase order where you carve everything, then clause everything, then implement everything.
Run the per-leaf loop to completion for one leaf before opening the next.

### 0. Pin the gate model
The LLM-backed gate roles shell out to a CLI; pin the model explicitly (e.g.
`ANTHROPIC_MODEL=<model-id>` on every invocation that triggers an adversary/narrower/source), or
the design silently runs on whatever the session default is. Record the pinned id in the run log.

### 1. Carve — `gantry --root <store> init <prd> --narrower <name>`
Decomposes the spec into per-requirement leaves. The narrower keys on an **enumerated requirements
section** (lines like `R1.`, `R2.`) — prose-only "teeth" produce a coarse architectural carve with
no grounded requirements. If your spec states requirements as prose, give it an `R1.`–`Rn.` section
(one per requirement) so each requirement becomes its own claimed leaf.

- The structure faces a **multi-claim gate**: no leaf may claim more than one requirement unless it
  is a *licensed* coupling node.
- A genuine coupling the narrower bundles appears as a COMBINED node. If the carve is wrong, you
  **re-carve** — you do not work around it:
  - `clause split <node>` — dissolve a combined node into independent per-requirement leaves.
  - `clause lift <leaf> <leaf>...` — compose independent leaves under a fresh (unlicensed) combined
    node (then license it, step 3).

Inspect the carve with `gantry --root <store> status`.

### 2. Clause — `gantry --root <store> clause edit <child> --statement -` then `clause accept <child> --adversary <name>`
Each requirement gets a **clause**: the contract the faithfulness adversary judges.
- **You draft** the clause statement (`clause edit`, statement from stdin or text).
- **The human authors** the real binding clause.
- **The unchanged structured-verdict adversary judges** whether the clause genuinely constrains the
  requirement (`clause accept --adversary <name>`).
- You do **not** author-and-judge your own clause.

A reject is not an obstacle — it carries a counterexample that names a real gap (Layer 5). Fix the
gap, re-edit, re-accept.

### 3. Couple (only for a genuine coupling) — `clause couple <child> --reason -` then `ratify --couple <id>`
A coupling is licensed by a **human-authored reason + ratify**. You may propose the structure and
draft the reason; you never seal it. `clause couple` records a *candidate* reason; `ratify --couple`
is the seal.

### 4. Ratify — `gantry --root <store> ratify --accept <id>...`
The human seal that binds the drafted obligations. **You never ratify on your own authority.** Only
ratified obligations bind as leaves.

### 5. Checker — `gantry --root <store> checker propose <child> --source <name>` (or `checker edit <child> --test -`) then `checker accept <child> --adversary <name>`
Each requirement gets a **checker**: a test that runs against the real implementation. The checker
is materialized as an integration test that depends on the implementation crate, so it can use only
the implementation's **public API**.
- **The source drafts** (`checker propose --source <name>`); you may also draft directly with
  `checker edit --test -` (stdin) — useful when the source lacks knowledge of the target's real API.
- **The human strengthens** the checker.
- **The unchanged adversary judges sufficiency** (`checker accept --adversary <name>`): does the
  test actually catch a violation, or is it hollow?
- You do **not** strengthen-and-accept your own checker.

Before submitting a checker for accept, satisfy yourself it has **teeth**: apply the matching
sabotage to the implementation (a snapshot-then-break) and confirm the test *fails*; restore. A test
that passes a broken impl is worthless and the adversary will (rightly) reject it.

### 6. Discharge — `gantry --root <store> discharge --workspace <impl-dir>`
Materializes the accepted checkers and runs them against the real implementation. **This is the only
thing that produces a green.** It returns one of:
- **Discharged** — an earned green, floored to the weakest cited fact's tier, with attribution.
- **Blocked** — a *located* obstruction (which node, and why).

Report exactly what discharge returns. Do not translate Blocked into "almost done."

### Where to author the implementation
`inhabit` is a placeholder — **inhabitation is caller-driven.** You write the implementation
(typically before/while iterating checkers, since the checker tests target its public API), and
`discharge` gates it. The implementation is the thing under test, never something you mark complete
yourself.

Write the implementation **per leaf**, just enough to discharge the leaf you are on — not the whole
spec at once. The checker for the leaf targets the public API the impl must expose; that, plus the
clause, is the entire contract that increment of code is written toward. When the leaf greens, move
on. Resist the pull to "get the implementation mostly done" ahead of the carve and then reconcile
later; that is the all-at-once anti-pattern, and it is what turns a rejection into a rewrite.

---

## Layer 4 — Anti-patterns (you will reach for these by default; don't)

**Anti-pattern: writing the full implementation, then running it through gantry as a final check.**
You will build toward an idea of "done" and only then submit the whole body of code to the gates.
When a gate rejects work that was built all at once and verified last, the implementation was written
toward "done" rather than toward the clauses — so the fix is a large, late rewrite.

*Instead:* let the carve drive the work; discharge each leaf as you build it. Take one leaf, clause
it, write just enough impl to discharge *that* leaf, earn the green, move on. Each increment of code
exists to discharge a claim you already established. A rejection is then local to one leaf and cheap
to fix, and there is no end-of-run rewrite — the friction lands continuously and small instead of all
at once and large.

**Anti-pattern: dropping requirements to manufacture a green.** When a clause or coverage gate fails,
you will reach for re-scoping — drop or defer the failing requirements so the remaining set
discharges clean — and frame it as the "cleanest honest green": re-carve to what the implementation
actually satisfies, record the rest as out-of-scope. **It is not honest.** Dropping an in-scope
requirement to produce a green is a false green wearing a clean shirt: the green now means *less* than
the reader thinks it does, and the dropped guarantee is silently gone. A failed gate is a signal
about the implementation or the clause — never a license to move the goalposts to where the work
already landed.

*Instead:* treat a failed gate as "fix the work or report the wall." The only honest moves are to fix
the implementation to satisfy the requirement, or take the located Blocked and report it. Leave the
requirement set alone unless the *spec itself* — not the gate outcome — says a requirement was
wrongly included.

- **Gaming an adversary rejection.** Rewording a clause/checker to slip past the adversary instead
  of satisfying the requirement. → The counterexample names a *real* gap. Fix the actual hole.
- **Inventing requirements.** Adding obligations the spec didn't state because a default seems
  plausible. → Build only what the carved requirements license. If the spec is underspecified,
  **report the gap; do not invent.**
- **Assuming away with an axiom.** Citing an axiom to declare-true what should be verified. → Axioms
  are human-declared facts about substrate gantry genuinely does not re-execute — *not* an escape
  hatch for "I'd rather not verify this." Declaring an axiom is a human act with a named declarer;
  you do not mint one to dodge a check.
- **Routing around a Blocked discharge.** Faking, stubbing, hardcoding an expected output, catching
  and swallowing an error, or loosening a check to turn Blocked → green. → Blocked *found a real
  obstruction*. Report it. It is a finding, not something to hide.
- **Authoring both sides.** Drafting the clause/checker AND treating it as accepted. → Draft only;
  the human authors the judged contract and the adversary seals (or rejects) the accept.
- **Touching the frozen core / kernel.** Editing the trusted verification core to make something
  pass. → The kernel is off-limits. If a change seems to require it, **stop and surface it** — that
  is a signal, not a task.
- **Self-accepting / self-ratifying.** There is no `--force` accept, and self-seal is structurally
  rejected on ratification. Don't look for the loophole; there shouldn't be one, and if you find
  one, treat using it as a failure.

> A green you obtained by any of the above is a **false green** — the single worst outcome in this
> system (gantry flags it LOUD, distinct from an honest Blocked). It is strictly worse than no green.

---

## Layer 5 — Read the system's responses; act on them honestly

Gantry's outputs are *information*, not obstacles. They are emitted as structured JSON (schemas like
`gantry.clause-verdict.v1`, `gantry.checker-verdict.v1`, `gantry.discharge.v1`, `gantry.status.v1`).

- **An adversary rejection** carries a concrete counterexample: the offending input
  (`counterexample_input`), the requirement it shows unenforced, and a one-line `counterexample_detail`
  naming the gap. *Read it.* It is telling you the real hole in your clause or checker. Fix that hole;
  do not reword to evade. (A genuine over-rejection — the adversary objecting to something the clause
  does not state — is itself a *finding*: tighten the clause or surface the over-rejection, don't
  paper over it.)
- **A Blocked discharge** located an obstruction (which node, and — when surfaced — the gate's
  output, e.g. the failing test). Report it precisely. It is a result, not a failure to conceal.
- **A green floored to a low tier** (e.g. an `Assume`/yellow tier) honestly carries its assumptions.
  Report the **modulo**: what the green rests on and who vouched for it. Never present a floored
  green as if it were a clean top-tier certify.
- **A stale fact** (a lemma or decision whose declared inputs changed) means **re-verify**, not
  ignore. A green cannot stand on a stale foundation; gantry will block one that tries.
- **Exit codes are stable** — read them. Roughly: `0` clean earned green; `3` Blocked (also: a
  rejected accept, an attribution-incomplete decomposition, a clause-faithfulness reject);
  `4` false-green (LOUD — investigate immediately); `5` an OR-choice is pending a human decision;
  `6` the narrower/proposer didn't converge. Confirm the current table with the binary's docs.

---

## The arbiter discipline (the process, not just the tools)

This skill carries the arbiter pattern to you even though you weren't present for it. Beyond the
commands:

- **Probe-first.** Before relying on a channel (a model role, an adapter), confirm it behaves —
  observe-only — instead of assuming. A channel you haven't watched work is a channel you can't
  trust yet.
- **Sabotage-verify every green.** A green is only trustworthy if a deliberate sabotage — a broken
  impl, a two-character no-op — *would have* produced Blocked. If sabotage still greens, the green
  is meaningless. **Expect your greens to be sabotage-checked, and reason as if they will be.** Do
  this to your own checkers before you trust them.
- **One falsifiable thing at a time.** Work in slices with a single unknown and isolated teeth.
  Don't bundle several unverified changes into one green.
- **Record material decisions.** Non-obvious choices get written down (the decision-record / D-NNN
  discipline) so they are auditable, not buried in a diff.

---

## The honest test of your work

You are operating this skill correctly if you:
- carve **before** implementing, then work **one leaf at a time** — clause it, write just enough impl
  to discharge it, earn its green, move on (never accumulate unverified impl for a final check);
- **draft** (never author-and-accept) clauses and checkers;
- run **discharge** instead of declaring done;
- respond to an adversary rejection by **fixing the real gap**;
- **report** a Blocked discharge instead of faking around it;
- respond to a failed gate by **fixing the work or reporting the wall** — never by dropping the
  in-scope requirement to make the remainder green;
- refuse to **invent requirements** or **assume-away** with an axiom;
- report a floored green **with its modulo**.

If instead you write the whole implementation and declare done — or re-scope the requirements to fit
what you built — you've defeated the system, and used this document as a command reference instead of
the discipline it is.
