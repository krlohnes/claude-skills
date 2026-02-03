---
name: dont-get-stupid
description: Summarize conversation key points before clearing context to preserve critical knowledge
---

# Don't Get Stupid Skill

This skill preserves critical conversation knowledge before context is cleared. It creates a
timestamped summary file weighted by **importance to the overall conversation**, not recency.

## Workflow

### Step 1: Analyze the Conversation

Review the entire conversation and identify:

1. **Critical Decisions Made** - Architecture choices, design patterns selected, rejected alternatives
2. **Key Technical Context** - File paths, function names, class structures, API endpoints involved
3. **Problem Definition** - What was the user trying to solve? What were the constraints?
4. **Implementation Progress** - What was completed, what's in progress, what's remaining
5. **Gotchas & Pitfalls** - Bugs encountered, edge cases discovered, things that didn't work
6. **User Preferences** - Explicit preferences stated, implicit patterns observed
7. **Outstanding Questions** - Unresolved issues, things that need verification

**IMPORTANT**: Weight by importance to achieving the goal, NOT by recency. A critical decision made
at the start of the conversation is more important than a minor clarification at the end.

### Step 2: Generate the Summary

Create a markdown file with the following structure:

```markdown
# Conversation Summary

**Generated**: <timestamp>
**Working Directory**: <cwd>
**Branch**: <git branch if applicable>

## Goal
<One paragraph describing what we were trying to accomplish>

## Critical Decisions
- <Decision 1>: <Rationale>
- <Decision 2>: <Rationale>

## Key Files & Locations
- `path/to/file1` - <what it does/why it matters>
- `path/to/file2` - <what it does/why it matters>

## Implementation Status
### Completed
- <Item 1>
- <Item 2>

### In Progress
- <Item 1>

### Remaining
- <Item 1>

## Important Context
<Any critical context that would be lost - gotchas, constraints, dependencies>

## Next Steps
1. <Step 1>
2. <Step 2>

## Open Questions
- <Question 1>
- <Question 2>
```

### Step 3: Write the Summary File

Generate a filename with timestamp:

```bash
# Generate filename
TIMESTAMP=$(date +"%Y-%m-%d-%H-%M-%S")
FILENAME="i_get_stupified_bak_${TIMESTAMP}.md"
```

Write the summary to the working directory (where the conversation started).

### Step 4: Create the TODO Entry

Use the TaskCreate tool to add a task at the front of the list:

```
Subject: Read conversation summary
Description: Read ${FILENAME} to restore context from previous session
```

**IMPORTANT**: This task should be created with high priority context so it's addressed first.

### Step 5: Confirm and Clear

1. Confirm the file was written successfully
2. Show the user the filename and location
3. Tell the user to run `/clear` to clear the context

**DO NOT automatically clear** - let the user verify and clear manually.

## Example Output

```
✓ Saved conversation summary to: ./i_get_stupified_bak_2026-02-03-14-30-45.md
✓ Added TODO: "Read i_get_stupified_bak_2026-02-03-14-30-45.md"

Key points preserved:
- Goal: Implementing attack chain extraction with weighted confidence
- 3 critical decisions documented
- 5 key files referenced
- 2 open questions noted

Run /clear to reset context. The summary will guide your next session.
```

## Notes

- The summary should be comprehensive enough that a fresh Claude session can pick up where this one left off
- Prioritize actionable information over verbose explanations
- Include specific file paths and line numbers when relevant
- Don't include sensitive information (API keys, passwords, etc.)
