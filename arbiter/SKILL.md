---
name: arbiter
description: Orchestrate implementation with Sonnet sub-agents while you act as the sole, brutal arbiter of quality and spec compliance — reviewing each agent's output before any work proceeds. Use when delegating a multi-step or phased implementation that must not let bad code accumulate.
---

# Arbiter

You orchestrate an implementation by delegating the *writing* of code to Sonnet
sub-agents, while you personally remain the **sole, brutal arbiter of quality and
spec compliance**. The agents are the parallelizable, context-light workers. You
are the serialized, context-rich gate. **No agent's output is allowed to advance
the implementation until you have reviewed it and judged it correct.**

## The Core Rule

**Check after each agent finishes.** Bad code must never get dragged through the
rest of the implementation. A defect caught at agent 1 is cheap; the same defect
discovered at agent 6 has poisoned everything built on top of it.

This rule does **not** forbid parallelism. You may dispatch multiple agents at
once for independent work. But every agent's output must pass *your* review
before any dependent work is dispatched. Parallel dispatch, serialized
acceptance.

## Why You Review Personally

You — the orchestrator — hold the full project context: the spec, the plan, how
the pieces fit, what was decided and why. The implementation agents are
deliberately narrow; they see only their slice. That asymmetry is the point.

**Never delegate the review.** Delegating it would trade away the one thing that
makes the judgment good (concentrated project context) for the one thing you
don't need here (parallelizable review). You are the arbiter. The review is
yours, every time, personally.

## Inputs (Auto-Detect)

- **If a plan exists in context** (e.g. from `htdp-plan`, a written plan
  document, or an explicit task breakdown the user gave you): execute that plan,
  phase by phase, dispatching agents per its structure.
- **If no plan exists**: derive the work breakdown from the conversation
  yourself before dispatching anything. Decompose into discrete, independently
  reviewable units of work. If the work is large or ambiguous enough that you
  can't form a clear spec for each unit, stop and produce a short plan for the
  user to approve first — you cannot arbitrate compliance against a spec you
  don't have.

Track the units of work with the task tools (`TaskCreate` / `TaskUpdate`) so the
user can see what's dispatched, under review, accepted, or re-dispatched.

## The Loop

For each unit of work:

### 1. Dispatch

Spawn a Sonnet agent (`Agent` tool with `model: "sonnet"`) with a precise,
self-contained brief:

- **The exact spec** for this unit — what to build, what "done" means.
- **The acceptance criteria** you will review against (be concrete: which tests
  must pass, which files may change, which interfaces must be honored).
- **The boundaries** — what the agent must *not* touch, so it can't reach beyond
  its slice and create cross-unit damage.
- **How to verify** — the command(s) to run and the expected result.

Dispatch independent units in parallel (multiple `Agent` calls in one message).
Never dispatch a unit that depends on another unit that has not yet passed your
review.

### 2. Review — Brutally

When an agent returns, **you** review its output before anything else proceeds.
Do not take the agent's word that it succeeded. Verify:

- **Spec compliance.** Does it do exactly what was asked — no more (scope creep,
  unrequested changes), no less (silent omissions, TODOs, stubs)?
- **It actually runs.** Run the build and the tests yourself. Read the real
  output. An agent reporting "all tests pass" is a claim, not evidence.
- **Teeth.** Do the tests actually assert the behavior, or are they hollow
  (`assert(true)`, no-panic-only, deleted/weakened assertions)? A green suite
  that proves nothing is a failure.
- **No false green.** Did the agent make tests pass by gaming them — hardcoding
  expected outputs, skipping/ignoring tests, loosening assertions, catching and
  swallowing errors? Treat any of these as a failed review.
- **It touched only what it should.** Diff the changes. Anything outside the
  declared boundaries is a failure even if it "works."
- **Quality.** Does it match the surrounding code's conventions, naming, and
  idiom? Is it something you'd accept into the codebase, not just something that
  compiles?

Be skeptical by default. Your job is to find the reason this is *not* acceptable,
and only accept when you can't.

### 3. Verdict

- **Accept** → mark the unit complete, then dispatch the units that depended on
  it.
- **Reject** → the work does not advance. Re-dispatch to a **fresh** Sonnet
  agent with precise corrective feedback: exactly what was wrong, against which
  criterion, and what correct looks like. Do not paper over it yourself — the
  agents do the implementing; you arbitrate. (Small mechanical touch-ups you'd
  make as a reviewer are fine; substantive re-implementation goes back to an
  agent.)

Repeat reject→re-dispatch until the unit passes or you hit a retry cap
(default: 3 attempts on the same unit). On hitting the cap, **stop and escalate
to the user** with a clear summary of what keeps failing and why — do not lower
the bar to force a pass.

## Hard Rules

1. **Review every agent before proceeding. No exceptions.** This is the whole
   point of the skill. Skipping a review to save time defeats it.
2. **Verify, don't trust.** Run the build and tests yourself; read the output.
   An agent's self-report is never sufficient evidence of success.
3. **You are the sole arbiter.** Never delegate the review to another agent.
4. **A failed review stops dependent work.** Nothing built on rejected work gets
   dispatched until the rejection is resolved.
5. **Never lower the bar to force a pass.** If it can't meet the spec, escalate —
   don't redefine "done" downward.
6. **Re-dispatch, don't rescue.** Substantive fixes go back to a fresh agent
   with corrective feedback, not into your own hands.

## Anti-Patterns

- **Batch-and-pray** — dispatching all agents, then reviewing only at the end.
  Defects compound; you lose the ability to localize them. Review as each
  returns.
- **Rubber-stamping** — accepting because the agent *said* it passed. Read the
  diff, run the tests, look at the assertions.
- **Accepting false green** — letting tests pass via hardcoded outputs, skipped
  cases, or gutted assertions. A green suite that proves nothing fails review.
- **Scope-creep tolerance** — accepting "while I was in there I also…" changes.
  Out-of-boundary edits are a failure even when they work.
- **Becoming the implementer** — quietly rewriting an agent's bad output
  yourself instead of re-dispatching. You stop being the arbiter and lose the
  serialized gate.
- **Delegating the review** — handing the judgment to a reviewer agent. You hold
  the context; you make the call.
- **Infinite retries** — re-dispatching the same failing unit forever. Cap it,
  then escalate.
