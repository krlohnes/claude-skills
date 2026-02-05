---
name: commit-message
description: Generate commit messages following tbaggery conventions (50/72 rule, imperative mood)
---

# Commit Message Generator

This skill generates git commit messages following the conventions established by Tim Pope in
"A Note About Git Commit Messages" (tbaggery.com).

## Conventions

### Subject Line (First Line)
- **50 characters or less** - hard limit
- **Capitalized** - start with uppercase letter
- **No period** at the end
- **Imperative mood** - "Fix bug" not "Fixed bug" or "Fixes bug"
- Describes WHAT changed at a high level

### Blank Line
- **Mandatory** separator between subject and body
- Critical for tools like `git log --oneline`, rebase, format-patch

### Body (Optional but Recommended)
- **Wrap at 72 characters** - hard limit per line
- Explains **WHY** the change was made, not just what
- Use present tense imperative: "Add feature" not "Added feature"
- Include context that isn't obvious from the diff
- Separate paragraphs with blank lines
- Bullet points are acceptable (use hyphens or asterisks)
- Use hanging indent for multi-line bullets

## Workflow

### Step 1: Analyze Staged Changes

Run these commands to understand what's being committed:

```bash
# See what's staged
git diff --cached --stat

# See the actual changes
git diff --cached

# Check for unstaged changes that might be missing
git status
```

### Step 2: Review Recent Commit History

Check the repository's commit message style:

```bash
git log --oneline -10
```

Adapt to any project-specific conventions while maintaining the core 50/72 rules.

### Step 3: Draft the Commit Message

Structure:
```
<type>: <subject line - 50 chars max>

<body - 72 chars per line max>

<optional footer for issue refs, breaking changes, etc.>
```

Common type prefixes (if project uses them):
- `feat:` - new feature
- `fix:` - bug fix
- `refactor:` - code restructuring without behavior change
- `docs:` - documentation only
- `test:` - adding/updating tests
- `chore:` - maintenance tasks

### Step 4: Validate the Message

Before presenting to user, verify:
- [ ] Subject ≤ 50 characters
- [ ] Subject is capitalized
- [ ] Subject has no trailing period
- [ ] Subject uses imperative mood
- [ ] Blank line after subject (if body exists)
- [ ] Body lines ≤ 72 characters
- [ ] Body explains WHY, not just WHAT

### Step 5: Present to User

Output the commit message in a code block:

```
<the commit message>
```

Also show:
- Character count of subject line
- Whether body was included and why

**DO NOT commit automatically** - present the message for user approval.

## Examples

### Simple Fix (Subject Only)
```
Fix null pointer exception in user validation
```
(47 characters)

### Feature with Context
```
Add rate limiting to authentication endpoints

The login and password reset endpoints were vulnerable to brute force
attacks. This adds a sliding window rate limiter that allows 5 attempts
per minute per IP address.

Rate limits are configurable via environment variables:
- AUTH_RATE_LIMIT_REQUESTS (default: 5)
- AUTH_RATE_LIMIT_WINDOW_SECONDS (default: 60)
```

### Refactoring with Rationale
```
Extract database connection logic into separate module

The connection pooling code was duplicated across three services and
had subtle differences that caused connection leaks in production.

This consolidates the logic into a shared module with:
- Consistent timeout handling
- Proper connection cleanup on shutdown
- Centralized configuration

Closes #1234
```

### Multi-line Bullets
```
Update dependencies to address security vulnerabilities

- Upgrade lodash from 4.17.15 to 4.17.21 to fix prototype
  pollution vulnerability (CVE-2020-8203)
- Upgrade minimist from 1.2.0 to 1.2.6 to fix prototype
  pollution (CVE-2021-44906)
- Upgrade node-fetch from 2.6.1 to 2.6.7 to fix information
  exposure vulnerability (CVE-2022-0235)
```

## Anti-Patterns to Avoid

### Bad Subject Lines
- `fixed stuff` - vague, lowercase
- `Updated the user authentication module to handle edge cases better.` - too long, period
- `Fixes bug #123` - not imperative
- `WIP` - not descriptive

### Bad Body Text
- Restating what the diff shows without explaining why
- Lines longer than 72 characters that wrap poorly in terminals
- No blank line between subject and body

## Notes

- The 50/72 rule exists for practical reasons: git tooling, email patches, terminal display
- When in doubt, add a body explaining the reasoning
- Reference issue numbers in the footer, not the subject
- If you can't summarize in 50 chars, the commit might be too large
