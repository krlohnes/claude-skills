---
name: ralph-orchestrator
description: Create Ralph orchestrator configurations from implementation plans
---

# Ralph Orchestrator Skill

Create Ralph orchestrator configurations from implementation plans. Ralph is a hat-based orchestration framework that keeps AI agents in a loop until the task is done.

## Running Ralph

```bash
# Run with custom config
ralph run -c .ralph/my-config.yaml

# Run with inline prompt
ralph run -p "Implement feature X"

# Run with prompt file
ralph run -P PROMPT.md

# Continue interrupted run
ralph run --continue

# List available presets
ralph init --list-presets

# Initialize from preset
ralph init --preset tdd-red-green
```

## Available Presets

| Preset | Description |
|--------|-------------|
| `tdd-red-green` | Test-Driven Development with red-green-refactor cycle |
| `spec-driven` | Specification-Driven Development |
| `feature` | Feature Development with integrated code review |
| `feature-minimal` | Minimal feature development with auto-derived instructions |
| `adversarial-review` | Red Team / Blue Team Security Review |
| `mob-programming` | Mob Programming with rotating roles |
| `research` | Deep exploration and analysis tasks (no code changes) |
| `debug` | Bug investigation and root cause analysis |
| `refactor` | Code Refactoring Workflow |
| `pr-review` | Multi-perspective PR code review |
| `docs` | Documentation Generation Workflow |
| `deploy` | Deployment and Release Workflow |

---

## Configuration File Structure

### Basic Structure (ralph.yaml)

```yaml
# Optional: Comment describing the workflow
# Pattern: [Coordination Pattern Name]
# Usage: ralph run -c path/to/config.yaml

event_loop:
  starting_event: "pipeline.start"      # Event that kicks off the loop
  completion_promise: "LOOP_COMPLETE"   # String agent outputs when done
  prompt_file: "PROMPT.md"              # Path to prompt file
  max_iterations: 100                   # Maximum loop iterations
  max_runtime_seconds: 14400            # Optional: 4 hour timeout
  checkpoint_interval: 5                # Optional: checkpoint frequency

cli:
  backend: "claude"                     # claude, kiro, gemini, codex, amp

core:
  specs_dir: "./specs/"                 # Optional: specs directory
  scratchpad: ".agent/scratchpad.md"    # Optional: scratchpad path

hats:
  hat_name:
    name: "🎯 Display Name"
    description: "Brief description of this hat's role"
    triggers: ["event.that.activates"]
    publishes: ["event.on.success", "event.on.failure"]
    default_publishes: "event.on.success"  # Optional: default event
    instructions: |
      Detailed instructions for this hat...
```

---

## Coordination Patterns

### 1. Linear Pipeline
Sequential stages where each stage triggers the next.

```yaml
event_loop:
  starting_event: "stage1.start"
  completion_promise: "pipeline.complete"

hats:
  stage1:
    name: "📝 Stage 1"
    description: "First stage of the pipeline"
    triggers: ["stage1.start"]
    publishes: ["stage2.start"]
    instructions: |
      Do stage 1 work, then publish stage2.start

  stage2:
    name: "🔧 Stage 2"
    description: "Second stage of the pipeline"
    triggers: ["stage2.start"]
    publishes: ["stage3.start"]
    instructions: |
      Do stage 2 work, then publish stage3.start

  stage3:
    name: "✅ Stage 3"
    description: "Final stage of the pipeline"
    triggers: ["stage3.start"]
    publishes: ["pipeline.complete"]
    instructions: |
      Do stage 3 work, output LOOP_COMPLETE when done
```

**Use for:** Sequential workflows, TDD pipelines, build→test→deploy chains

---

### 2. Critic-Actor Loop
One agent creates, another critiques, loops until approved.

```yaml
event_loop:
  starting_event: "create.start"
  completion_promise: "LOOP_COMPLETE"

hats:
  creator:
    name: "✏️ Creator"
    description: "Creates artifacts for review"
    triggers: ["create.start", "review.rejected"]
    publishes: ["review.ready"]
    instructions: |
      Create the artifact. If rejected, address feedback.
      Publish review.ready when done.

  critic:
    name: "🔍 Critic"
    description: "Reviews and approves or rejects work"
    triggers: ["review.ready"]
    publishes: ["review.approved", "review.rejected"]
    default_publishes: "review.approved"
    instructions: |
      Review the work critically.
      If issues: publish review.rejected with specific feedback
      If approved: output LOOP_COMPLETE
```

**Use for:** Code review, spec validation, quality gates

---

### 3. Red-Green-Refactor (TDD)
Classic TDD cycle with three distinct phases.

```yaml
event_loop:
  starting_event: "tdd.start"
  completion_promise: "LOOP_COMPLETE"

hats:
  test_writer:
    name: "🔴 Test Writer"
    description: "Writes FAILING tests first. Never implements."
    triggers: ["tdd.start", "refactor.done"]
    publishes: ["test.written"]
    instructions: |
      Write FAILING tests first. This is non-negotiable.
      1. Read the spec/requirement
      2. Write the minimum test that captures the requirement
      3. Run the test to verify it FAILS (red phase)
      4. Publish test.written

      NEVER write implementation code.

  implementer:
    name: "🟢 Implementer"
    description: "Makes failing test pass with MINIMAL code."
    triggers: ["test.written"]
    publishes: ["test.passing"]
    instructions: |
      Make the failing test pass with MINIMAL code.
      1. Read the failing test
      2. Write the simplest code that makes it pass
      3. Run the test to confirm green
      4. Publish test.passing

      Do NOT refactor. Do NOT add extra functionality.

  refactorer:
    name: "🔵 Refactorer"
    description: "Cleans up code while keeping tests green."
    triggers: ["test.passing"]
    publishes: ["refactor.done", "cycle.complete"]
    default_publishes: "cycle.complete"
    instructions: |
      Clean up the code while keeping tests green.
      1. Review for code smells
      2. Refactor for clarity, DRY, maintainability
      3. Run tests to confirm still passing
      4. If more tests needed: publish refactor.done
      5. If feature complete: output LOOP_COMPLETE
```

**Use for:** Test-driven development, feature implementation with tests

---

### 4. Adversarial (Red Team/Blue Team)
One agent builds, another tries to break it.

```yaml
event_loop:
  starting_event: "security.review"
  completion_promise: "LOOP_COMPLETE"

hats:
  builder:
    name: "🔵 Blue Team"
    description: "Implements features with security in mind."
    triggers: ["security.review", "fix.applied"]
    publishes: ["build.ready"]
    instructions: |
      Implement with security in mind. Consider:
      - Input validation, injection attacks
      - Authentication/authorization
      - Data exposure, error handling
      Publish build.ready when implementation is ready.

  red_team:
    name: "🔴 Red Team"
    description: "Penetration tester that tries to break the code."
    triggers: ["build.ready"]
    publishes: ["vulnerability.found", "security.approved"]
    instructions: |
      Your job is to BREAK this code. Attack vectors:
      - Injection (SQL, XSS, command)
      - Auth bypass, IDOR
      - Race conditions, path traversal

      If vulnerabilities found: publish vulnerability.found
      If secure: output LOOP_COMPLETE

  fixer:
    name: "🛡️ Fixer"
    description: "Remediates security vulnerabilities."
    triggers: ["vulnerability.found"]
    publishes: ["fix.applied"]
    instructions: |
      Remediate the vulnerability with defense in depth.
      Add regression test. Publish fix.applied for re-review.
```

**Use for:** Security review, penetration testing, hardening

---

### 5. Rotating Roles (Mob Programming)
Multiple perspectives rotate through the same codebase.

```yaml
event_loop:
  starting_event: "mob.start"
  completion_promise: "LOOP_COMPLETE"

hats:
  navigator:
    name: "🧭 Navigator"
    description: "Thinks strategically, gives instructions to driver."
    triggers: ["mob.start", "observation.noted"]
    publishes: ["direction.set", "mob.complete"]
    instructions: |
      Think strategically. Give CLEAR, SPECIFIC instructions.
      Do NOT write code—describe what to write.
      If task complete: output LOOP_COMPLETE
      Otherwise: publish direction.set with instructions

  driver:
    name: "⌨️ Driver"
    description: "Executes navigator's instructions exactly."
    triggers: ["direction.set"]
    publishes: ["code.written"]
    instructions: |
      Execute navigator's instructions EXACTLY.
      You're the hands, not the brain. Stay tactical.
      Publish code.written when done.

  observer:
    name: "👁️ Observer"
    description: "Provides fresh-eyes feedback."
    triggers: ["code.written"]
    publishes: ["observation.noted"]
    instructions: |
      Provide fresh-eyes feedback:
      - Potential bugs, simpler approaches
      - Missing error handling, edge cases
      - Style, naming, performance, security
      Publish observation.noted with feedback.
```

**Use for:** Complex implementations, learning/teaching, collaborative design

---

### 6. Research & Synthesis
Information gathering without code changes.

```yaml
event_loop:
  starting_event: "research.start"
  completion_promise: "RESEARCH_COMPLETE"
  max_iterations: 20

hats:
  researcher:
    name: "🔬 Researcher"
    description: "Gathers information and analyzes patterns. No code changes."
    triggers: ["research.start", "research.followup"]
    publishes: ["research.finding", "research.question"]
    instructions: |
      Research, not implementing. Gather information, analyze patterns.
      1. Scope the question
      2. Search broadly (grep, find, read files)
      3. Document findings with file:line references
      4. Identify gaps

      DON'T: Write code, make commits, make assumptions
      When answered with evidence: output RESEARCH_COMPLETE

  synthesizer:
    name: "📊 Synthesizer"
    description: "Reviews findings and creates coherent summary."
    triggers: ["research.finding"]
    publishes: ["research.followup", "synthesis.complete"]
    instructions: |
      Review findings and create coherent summary.
      1. Read all findings
      2. Identify patterns and connections
      3. Note contradictions or gaps
      4. If gaps: publish research.followup
      5. If complete: provide summary
```

**Use for:** Codebase analysis, architecture review, investigation

---

## PROMPT.md Template

```markdown
# [Ticket/Feature ID]: [Title]

## Summary
Brief description of what needs to be accomplished.

## Acceptance Criteria
| Requirement | Description |
|-------------|-------------|
| Criterion 1 | Details |
| Criterion 2 | Details |

## Files to Modify
### Backend
- `path/to/file1.go` - Description of changes
- `path/to/file2.go` - Description of changes

### Frontend
- `path/to/component.tsx` - Description of changes

## Implementation Details

### [Component 1]
```code
// Code snippet or pseudocode
```

### [Component 2]
```code
// Code snippet or pseudocode
```

## Test Cases
1. **TestName1** - Description of what to test
2. **TestName2** - Description of what to test

## Pipeline Flow (if applicable)
```
stage1 → stage2 → stage3 → done
```
```

---

## Hat Instruction Best Practices

1. **Be specific about what the hat does and doesn't do**
   ```yaml
   instructions: |
     ### DO
     - Write failing tests
     - Run tests to verify failure

     ### DON'T
     - Write implementation code
     - Skip running tests
   ```

2. **Include event emission format**
   ```yaml
   instructions: |
     ### Event Format
     ```bash
     ralph emit "event.name" "context message"
     ```
   ```

3. **Define clear exit conditions**
   ```yaml
   instructions: |
     ### DONE
     When all acceptance criteria pass: output LOOP_COMPLETE
   ```

4. **Include backpressure commands**
   ```yaml
   instructions: |
     ### Backpressure (must pass before continuing)
     ```bash
     go build ./... && go test ./...
     npm run tscheck && npm run lint
     ```
   ```

5. **Handle blocked/stuck states**
   ```yaml
   instructions: |
     ### STUCK?
     ```bash
     ralph emit "build.blocked" "description of blocker"
     ```
   ```

---

## Common Event Naming Conventions

| Pattern | Events |
|---------|--------|
| Pipeline | `stage1.start`, `stage2.start`, `pipeline.complete` |
| Review | `review.ready`, `review.approved`, `review.rejected` |
| Build | `build.task`, `build.done`, `build.blocked` |
| Test | `test.written`, `test.passing`, `test.failing` |
| TDD | `tdd.start`, `refactor.done`, `cycle.complete` |
| Security | `security.review`, `vulnerability.found`, `fix.applied` |
| Research | `research.start`, `research.finding`, `research.followup` |

---

## Full Example: TDD Fullstack Pipeline

```yaml
# TDD Fullstack Pipeline
# Pattern: Linear Pipeline with TDD stages

event_loop:
  starting_event: "pipeline.start"
  completion_promise: "pipeline.complete"
  prompt_file: ".ralph/PROMPT.md"

hats:
  backend_test_writer:
    name: "🧪 Backend Test Writer"
    description: "Writes failing integration tests for backend features"
    triggers: ["pipeline.start"]
    publishes: ["backend.tests.written"]
    instructions: |
      Write FAILING integration tests for the backend feature.

      1. Read the PROMPT.md requirements
      2. Create test file with all test cases
      3. Run tests to verify they FAIL
      4. Publish backend.tests.written

      Tests should FAIL initially (red phase).

  backend_implementer:
    name: "🔧 Backend Implementer"
    description: "Implements backend to make tests pass"
    triggers: ["backend.tests.written"]
    publishes: ["backend.complete"]
    instructions: |
      Implement the backend to make tests pass.

      1. Add models/types
      2. Update SQL queries
      3. Update handlers
      4. Run tests until GREEN
      5. Publish backend.complete

  frontend_implementer:
    name: "🎨 Frontend Implementer"
    description: "Implements frontend types and components"
    triggers: ["backend.complete"]
    publishes: ["frontend.complete"]
    instructions: |
      Implement the frontend changes.

      1. Add TypeScript types
      2. Update components
      3. Run: npm run tscheck && npm run lint
      4. Publish frontend.complete

  verifier:
    name: "✅ Verifier"
    description: "Runs all checks and verifies implementation"
    triggers: ["frontend.complete"]
    publishes: ["pipeline.complete"]
    instructions: |
      Verify the complete implementation.

      ```bash
      cd backend && go test ./...
      cd frontend && npm run tscheck && npm run lint
      ```

      If all pass: output LOOP_COMPLETE
      If failures: identify and report issues
```
