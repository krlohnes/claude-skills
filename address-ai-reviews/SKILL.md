---
name: address-ai-reviews
description: Guide for addressing ai feedback on pull requests
---

# Address AI Reviews Skill

This skill guides Claude through the process of reviewing and addressing AI-generated
feedback on pull requests.

## Workflow

### Step 1: Find the Current PR

Use `gh` to find the PR associated with the current branch:

```bash
# Get PR number and URL for current branch
gh pr view --json number,url,title,body

# If no PR exists, inform the user
gh pr list --head $(git branch --show-current)
```

### Step 2: Retrieve AI Review Comments

Fetch all review comments on the PR:

```bash
# Get all review comments
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments

# Get all reviews (including pending)
gh pr view {pr_number} --json reviews

# Get specific review threads
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews
```

### Step 3: Validate and Verify Each Suggestion

For each AI-generated suggestion, critically evaluate:

1. **Is the suggestion technically correct?**
   - Read the relevant code context
   - Understand the existing patterns in the codebase
   - Check if the suggestion aligns with project conventions

2. **Is the suggestion actually applicable?**
   - Some AI suggestions are generic and don't apply to the specific context
   - Check if the "problem" actually exists in the code
   - Verify the suggestion doesn't break existing functionality

3. **Is the suggestion valuable?**
   - Does it improve security, performance, or maintainability?
   - Is it fixing a real bug or just cosmetic?
   - Does the benefit outweigh the change complexity?

4. **Common AI review false positives to watch for:**
   - Suggesting error handling that already exists elsewhere
   - Recommending patterns that conflict with project conventions
   - Flagging intentional design decisions as issues
   - Over-engineering simple code
   - Suggesting changes that would break existing tests

### Step 4: Implement Valid Fixes

For suggestions deemed valid:

1. Read the affected files thoroughly
2. Implement the fix following existing code patterns
3. Run relevant tests to ensure the fix doesn't break anything
4. If tests exist for the affected code, verify they still pass

### Step 5: Create the Commit

Create a commit following tbaggery style (50 char subject, blank line, wrapped body):

```
Address AI review feedback on PR #<number>

Valid suggestions implemented:
- <brief description of fix 1>
- <brief description of fix 2>

Suggestions ignored:
- <suggestion>: <reason it was ignored>
- <suggestion>: <reason it was ignored>
```

**Commit message guidelines:**
- Subject line: 50 characters or less, imperative mood
- Body: Wrap at 72 characters
- Separate subject from body with blank line
- List what was fixed and what was intentionally ignored with rationale

## Important Notes

- Never blindly apply AI suggestions without verification
- The codebase conventions take precedence over generic best practices
- If uncertain about a suggestion, investigate the codebase patterns first
- Document why suggestions were ignored to prevent repeat feedback
- Run tests after making changes to verify nothing broke
