---
name: htdp-plan
description: Create a detailed TDD implementation plan from the current conversation context, with assertions that validate correct implementation
---

# TDD Implementation Plan

This skill takes the current conversation context and produces a detailed, phased implementation
plan rooted in Test-Driven Development and the design recipe from Felleisen et al.'s "How to
Design Programs." The key insight: **the structure of the data determines the structure of the
code, and concrete examples (tests) written before implementation are what prove the design is
correct.**

## Philosophy

The design recipe says: understand your data, define what the function consumes and produces,
write concrete examples of expected behavior, then implement. TDD says: write those examples
as executable tests, then write the code that makes them pass. Combined:

1. **Data first.** Before you can write good tests, you need to know what data you're working
   with. Define the data representations — what information is being tracked, what are the
   cases, how is it structured. The structure of the data drives the structure of the code.
2. **Signature and purpose.** For each function or component, state what it consumes, what it
   produces, and why. This is the contract in plain language.
3. **Every data case becomes a test.** The data definitions enumerate the cases. Every case
   must have at least one test that uses it as input. If a data case has no test, it's not
   defined — it's decoration.
4. **Tests get written to disk before any production code.** Not in a plan document. Not in
   your head. Actually written to the test files. This is non-negotiable.
5. **Once implementation is complete, every assertion MUST pass.** If an assertion fails, the
   implementation is wrong — not the test. The assertions are the contract.

## Invocation

Accepts optional arguments to narrow scope:
- No argument: plans based on the full conversation context
- A description: `/htdp-plan user authentication flow` to focus the plan

## How This Skill Works

This skill produces a plan. The plan is presented to the user for approval. Only after
approval do you execute it. When executing: write the test files to disk first, then
implement the production code. Do not write production code before test files exist on disk.

### Building the Plan

Before producing the plan, work through the design recipe:

#### 1. Understand the Problem

Review the conversation and identify:
- **The goal** -- what are we building or changing?
- **Invariants** -- what must always be true?
- **Edge cases** -- what happens at boundaries, with empty inputs, with errors?
- **Integration points** -- what other systems/modules does this touch?
- **Acceptance criteria** -- how do we know it's done and correct?

If any of these are unclear or ambiguous, **ask the user before proceeding**. Do not guess at
requirements. But do not stall — ask only about things that would change the data definitions
or the tests. If you can make a reasonable assumption, state it as an explicit **Assumption**
at the top of the plan so the user sees it before approving.

#### 2. Define the Data

Before writing any tests, define the data representations:
- **What information** is being tracked or transformed?
- **What are the cases?** Is this an enumeration, a compound structure, a recursive type?
- **How is it represented?** Structs, enums, maps, primitives?
- **What are examples** of each data case?

The data definitions belong in the plan. They drive the structure of both the tests and the
implementation. If you get the data wrong, the tests will be wrong and the implementation
will be wrong.

Edge cases are data cases. Empty inputs, nil values, zero values, boundary values — these
are cases in your data definition, not afterthoughts. List them alongside the happy path.

#### 3. Signature and Purpose

For each function or component the plan will cover, state:
- **What it consumes** -- the input types and their meaning
- **What it produces** -- the output types and their meaning
- **Why** -- a one-sentence purpose statement

#### 4. Identify Conventions

Read existing test files in the project and identify:
- What kinds of tests are needed (unit, integration, contract)
- The assertion library, test helpers, setup/teardown patterns, and naming schemes
- File organization and naming conventions

The tests you write must follow these conventions. Do not introduce new patterns or libraries
unless the project has none.

### Plan Structure

The plan is organized into phases. Each phase has the same two-step structure:

**Step 1 is always: Write the tests.** This is the most important part of the entire plan.
The tests specify exactly how the code should behave. Every assertion encodes a concrete,
verifiable claim. The tests are real test code, not pseudocode. Write them to the test
files on disk before writing any production code.

**Step 2 is always: Implement the code to make the tests pass.** This includes creating
whatever types, interfaces, and functions the tests reference. Implementation is not done
until every assertion passes.

**Every assertion must be concrete and testable:**
- BAD: "it should work correctly"
- BAD: "it should handle errors"
- GOOD: `assert.Equal(t, ErrNilResource, err)`
- GOOD: `assert.Equal(t, "r2", results[0].ResourceID) // highest score first`

The tests should be detailed enough that the implementation is fully determined by them — there
should be only one reasonable way to make the tests pass.

#### Phase Template

```
## Phase N: <descriptive name>

### What We're Proving
<1-2 sentences: what capability this phase validates. Every claim made here must be
backed by at least one test assertion below.>

### Data Definitions
<What data types this phase introduces or uses. Define the structure: fields, cases,
relationships. Include concrete examples of each data case. Every data case defined here
must appear as a test input in the Tests & Assertions section below.>

### Signature and Purpose
<For each function/method this phase covers:
- Name, inputs (with types), output (with type)
- One-sentence purpose statement>

### Step 1: Write the Tests

#### Test File(s)
- `path/to/test_file.go` (or .ts, .rs, etc.)

#### Tests & Assertions
<Specific test functions with their assertions. Real test code, not pseudocode.
These tests define the spec. Write them to disk before any production code.
Tests must cover every data case from the Data Definitions section and exercise
every signature from the Signature and Purpose section.>

### Step 2: Implement

Do not start this step until the test files from Step 1 exist on disk.

#### Implementation Scope
<What production code will be written to make these tests pass. Must create every type
from Data Definitions and every function from Signature and Purpose.>

#### Verification
<How to run the tests, expected output>
```

#### Phase Ordering Rules

1. **Start with the simplest, most isolated unit** -- build confidence from the inside out.
2. **Each phase should end with all its tests passing** -- write the tests, implement until they
   pass, then move to the next phase. This does NOT mean write easy tests — it means finish
   each phase before starting the next.
3. **Later phases can build on earlier ones** -- integration tests come after the units they
   integrate are proven correct.
4. **The final phase should include an end-to-end or integration test** that exercises the full
   feature path.

### Presenting the Plan

Output the full plan with:

1. **Summary** -- one paragraph describing what we're building and the approach
2. **Data definitions** -- all data types across the plan, with examples of each case
3. **Phase breakdown** -- each phase with its signatures, tests, and implementation scope
4. **Assertion inventory** -- a numbered list of every `assert.*` and `require.*` call (or
   equivalent) across all phases, so the user can see at a glance what "done" looks like
5. **Execution order** -- clear sequence of what to do first, second, etc.

**Wait for user approval** before writing any test or production code. The user may want to:
- Adjust assertion specifics
- Reorder phases
- Add missing edge cases
- Remove out-of-scope items

If the user modifies the plan, re-verify that data definitions, signatures, and tests still
connect. A change to a data definition may require changes to tests. A removed test may leave
a data case uncovered.

## Critical Rules

1. **Data definitions drive everything.** If you get the data wrong, the tests will be wrong
   and the implementation will be wrong. Define the data first, with concrete examples of each
   case. The structure of the data determines the structure of the code.

2. **Tests encode the design.** If the test doesn't assert the right thing, the implementation
   can pass while being wrong. Spend more time on assertion design than on test scaffolding.

3. **Once implementation is complete, every assertion must pass.** If an assertion fails, the
   implementation is wrong -- not the test. (Unless the user explicitly agrees to change a test.)

4. **No empty tests.** Every test function must contain at least one meaningful assertion. A test
   that only checks "no panic" or "returns without error" is almost never sufficient.

5. **No mocking what you own.** Prefer real implementations and in-memory fakes over mocks for
   code you control. Mock external services and I/O boundaries.

6. **Match existing project conventions.** Read the existing test files before designing new ones.
   Use the same test helpers, assertion libraries, setup/teardown patterns, and naming schemes.

7. **Be specific about expected values.** Don't assert `!= nil` when you can assert the exact
   value. Don't assert `len > 0` when you know the exact length.

8. **Test names describe the scenario.** `TestCalculate_EmptySignals` tells you what data case
   is being tested. `TestCalculate2` does not. Name tests after the data case or behavior they
   verify.

## Anti-Patterns to Avoid

- **Writing tests after implementation** -- that's validation, not TDD. The tests come first.
- **Skipping or hand-waving data definitions** -- "takes a string and returns a struct" is not
  a data definition. Define every field, every case, every constraint, with concrete examples.
  If you can't define the data, you don't understand the problem yet.
- **Data definitions that don't connect to tests** -- if a data case appears in the Data
  Definitions section but no test uses it as input, the definition is decorative. Every data
  case must be exercised by at least one test.
- **Signatures that don't match tests** -- if the Signature and Purpose section says a function
  takes X and returns Y, the tests must call that function with X and assert on Y.
- **Tests that test the mock** -- if your test is mostly mock setup, you're testing the mock
  framework, not your code.
- **Assertions that can't fail** -- `assert(true)` or testing that a constructor returns a
  non-nil value. These prove nothing.
- **Changing tests to match broken implementation** -- if an assertion fails, fix the code, not
  the test. The test was written to spec. (Only change tests if the spec itself was wrong, and
  confirm with the user first.)
- **Skipping edge cases** -- empty inputs, nil values, max values, concurrent access. These are
  where bugs hide.
- **Gigantic phases** -- if a phase has more than 5-7 test functions, break it up. Smaller phases
  give faster feedback.
- **Writing tests and production code in the same step** -- write the test files to disk, confirm
  they exist, then write the production code. These are separate steps, not one action.
- **Stuffing multiple data cases into one test** -- each test should verify one scenario. A test
  named `TestCalculate_MultipleSignals` that also tests sorting and aggregation is three tests
  pretending to be one. If it fails, you don't know which case broke.

## Example Phase

```
## Phase 1: Resource Score Calculation

### What We're Proving
The risk score calculator correctly computes a weighted score from individual
signal scores and returns results sorted by severity.

### Data Definitions
- Signal: a single risk signal for a resource
  - ResourceID (string): which resource this signal applies to
  - Category (string): the type of signal, e.g. "vulnerability", "exposure"
  - Score (float64): severity from 0.0 (none) to 1.0 (critical)
  - Cases and examples:
    - Single signal: Signal{ResourceID: "r1", Category: "vulnerability", Score: 0.8}
    - Different category: Signal{ResourceID: "r1", Category: "exposure", Score: 0.3}
    - Different resource: Signal{ResourceID: "r2", Category: "vulnerability", Score: 0.95}
    - Empty list: []Signal{}
    - Nil: nil

- ScoredResource: the aggregated risk score for a single resource
  - ResourceID (string): which resource
  - TotalScore (float64): weighted aggregate of all signals for this resource
  - Breakdown (map[string]float64): score per category
  - Cases and examples:
    - Multiple signals aggregated: ScoredResource{ResourceID: "r1", TotalScore: 0.55, Breakdown: {"vulnerability": 0.8, "exposure": 0.3}}
    - Single signal: ScoredResource{ResourceID: "r2", TotalScore: 0.95, Breakdown: {"vulnerability": 0.95}}
    - Empty result (from empty/nil input): []ScoredResource{}

### Signature and Purpose
- DefaultWeights() map[string]float64
  Return the default per-category weight configuration.
- NewScoreCalculator(weights map[string]float64) *ScoreCalculator
  Create a calculator with per-category weights.
- (*ScoreCalculator).Calculate(signals []Signal) ([]ScoredResource, error)
  Aggregate signals by resource, compute weighted scores, return sorted by severity descending.

### Step 1: Write the Tests

#### Test File(s)
- `internal/scoring/calculator_test.go`

#### Tests & Assertions
func TestCalculate_SingleSignal(t *testing.T) {
    calc := NewScoreCalculator(DefaultWeights())
    signals := []Signal{
        {ResourceID: "r1", Category: "vulnerability", Score: 0.95},
    }

    results, err := calc.Calculate(signals)

    require.NoError(t, err)
    require.Len(t, results, 1)
    assert.Equal(t, "r1", results[0].ResourceID)
    assert.InDelta(t, 0.95, results[0].TotalScore, 0.01)
}

func TestCalculate_MultipleSignalsSameResource(t *testing.T) {
    calc := NewScoreCalculator(DefaultWeights())
    signals := []Signal{
        {ResourceID: "r1", Category: "vulnerability", Score: 0.8},
        {ResourceID: "r1", Category: "exposure", Score: 0.3},
    }

    results, err := calc.Calculate(signals)

    require.NoError(t, err)
    require.Len(t, results, 1)
    assert.Equal(t, "r1", results[0].ResourceID)
    assert.InDelta(t, 0.55, results[0].TotalScore, 0.01)
    assert.InDelta(t, 0.8, results[0].Breakdown["vulnerability"], 0.01)
    assert.InDelta(t, 0.3, results[0].Breakdown["exposure"], 0.01)
}

func TestCalculate_MultipleResources_SortedBySeverity(t *testing.T) {
    calc := NewScoreCalculator(DefaultWeights())
    signals := []Signal{
        {ResourceID: "r1", Category: "vulnerability", Score: 0.3},
        {ResourceID: "r2", Category: "vulnerability", Score: 0.95},
    }

    results, err := calc.Calculate(signals)

    require.NoError(t, err)
    require.Len(t, results, 2)
    assert.Equal(t, "r2", results[0].ResourceID) // highest score first
    assert.Equal(t, "r1", results[1].ResourceID)
}

func TestCalculate_EmptySignals(t *testing.T) {
    calc := NewScoreCalculator(DefaultWeights())
    results, err := calc.Calculate([]Signal{})

    require.NoError(t, err)
    assert.Empty(t, results)
}

func TestCalculate_NilSignals(t *testing.T) {
    calc := NewScoreCalculator(DefaultWeights())
    results, err := calc.Calculate(nil)

    require.NoError(t, err)
    assert.Empty(t, results)
}

### Step 2: Implement

Do not start until calculator_test.go exists on disk.

#### Implementation Scope
- `internal/scoring/calculator.go`: Signal, ScoredResource types,
  NewScoreCalculator, DefaultWeights, Calculate method
- Weighted average logic with configurable weights per category

#### Verification
cd internal/scoring && go test -v -run TestCalculate
Expected: 5 tests pass, 0 failures
```

## Notes

- This skill produces a plan first, then executes it after user approval.
- The plan should be detailed enough that the implementation is fully determined by the tests.
- If the conversation doesn't have enough context to produce concrete data definitions and
  assertions, say so and ask for clarification rather than producing vague output.
