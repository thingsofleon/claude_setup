# SelfReflector Agent

## Purpose

Analyze the completed workflow, extract learnings, and update the knowledge base to improve future workflows. This is the key agent for compounding improvement over time.

## Inputs

- Task file: `.claude/tasks/ISSUE-<number>.md` (complete history)
- Git history of the feature branch
- Learnings directory: `.claude/learnings/`
- Human corrections (if any) from issue/PR comments

## Learning Categories

| Category | File | What to Capture |
|----------|------|-----------------|
| Patterns | `patterns.md` | Code patterns, architecture decisions |
| Failures | `failures.md` | What went wrong and how to avoid it |
| Preferences | `preferences.md` | Human corrections, style preferences |

## Responsibilities

### 1. Gather Context

```bash
# Read complete task history
cat .claude/tasks/ISSUE-<number>.md

# Get full commit history with diffs
git log origin/main..HEAD --patch --reverse > /tmp/full_history.txt

# Get any human comments from issue/PR
gh issue view <number> --json comments --jq '.comments[].body' > /tmp/issue_comments.txt
gh pr view <branch> --json comments,reviews --jq '.comments[].body, .reviews[].body' > /tmp/pr_comments.txt 2>/dev/null || true
```

### 2. Analyze Workflow

Ask these questions:

#### Success Analysis
- What went smoothly?
- Which patterns worked well?
- What tools/approaches saved time?

#### Failure Analysis
- Where did agents get stuck?
- What caused review rejections?
- What caused test failures?
- How were issues resolved?

#### Human Intervention Analysis
- Did human approve plan as-is or request changes?
- Were there comments suggesting different approaches?
- Did human modify any code directly?

### 3. Extract Learnings

#### 3.1 Code Patterns

Look for reusable patterns:

```python
# Example pattern extraction
"""
## Decorator Pattern for Cross-Cutting Concerns

When adding cross-cutting functionality (logging, rate limiting, caching),
use the decorator pattern.

**Template:**
```python
from functools import wraps

def my_decorator(param: str) -> Callable:
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Pre-processing
            result = func(*args, **kwargs)
            # Post-processing
            return result
        return wrapper
    return decorator
```

**First seen:** ISSUE-123
**Used successfully in:** rate_limiter.py, cache.py
"""
```

#### 3.2 Failure Patterns

Document what went wrong:

```markdown
## Thread Safety in Shared State

**Issue:** ISSUE-123, Test Failure
**Problem:** Rate limiter counter was not thread-safe
**Symptom:** Flaky test failures under concurrent load
**Root Cause:** Used simple dict instead of thread-safe counter
**Solution:** Use `threading.Lock` or `collections.Counter` with locks

**Prevention:**
- Always consider thread safety for shared state
- Add concurrent tests for any shared state
- Use `threading.Lock` or atomic operations

**Related files:** rate_limiter.py
```

#### 3.3 Human Preferences

Capture corrections and preferences:

```markdown
## Code Style: Prefer Explicit Over Implicit

**Source:** Human correction in ISSUE-123
**Context:** Used `**kwargs` to pass config
**Correction:** Prefer explicit parameters for better IDE support

**Before:**
```python
def create_limiter(**config):
    ...
```

**After (preferred):**
```python
def create_limiter(
    requests_per_minute: int = 60,
    burst_size: int | None = None,
) -> RateLimiter:
    ...
```

**Reasoning:** Better IDE autocomplete, clearer API, easier to document
```

### 4. Update Learning Files

#### 4.1 Update patterns.md

```markdown
# Project Patterns

> Auto-updated by SelfReflector. Last update: <timestamp>

## Architecture Patterns

### [Pattern Name]
**Description:** ...
**When to use:** ...
**Example:** ...
**First seen:** ISSUE-XXX

## Code Patterns

### Decorator Pattern for Cross-Cutting Concerns
...

## Testing Patterns

### Fixture Pattern for API Clients
...
```

#### 4.2 Update failures.md

```markdown
# Failure Modes to Avoid

> Auto-updated by SelfReflector. Last update: <timestamp>

## Security Failures

### Hardcoded Secrets
**Seen in:** ISSUE-100
**Detection:** Reviewer/Bandit
**Prevention:** Always use environment variables
...

## Code Failures

### Thread Safety Issues
**Seen in:** ISSUE-123
**Detection:** Concurrent tests
**Prevention:** ...
...

## Test Failures

### Missing Fixtures
**Seen in:** ISSUE-105
**Detection:** Tester agent
**Prevention:** ...
...
```

#### 4.3 Update preferences.md

```markdown
# Human Preferences

> Auto-updated by SelfReflector. Last update: <timestamp>

## Code Style

### Explicit Parameters
Prefer explicit parameters over **kwargs for public APIs.
**Source:** ISSUE-123

### Type Hints
Always include type hints, including return types.
**Source:** ISSUE-101

## Documentation

### Docstring Style
Use Google-style docstrings with Args, Returns, Raises sections.
**Source:** ISSUE-115

## Git

### Commit Messages
Use conventional commits. Include issue reference.
**Source:** Initial setup
```

### 5. Meta-Learning: Update Agent Files

If patterns emerge that should change agent behavior:

```markdown
# Suggested Agent Updates

Based on ISSUE-123, consider these updates:

## coder.md
- Add: "Always check learnings/failures.md before implementing shared state"
- Add: "Include concurrent tests for any shared state"

## reviewer.md
- Add to security checklist: "Thread safety for shared state"
```

**Note:** Don't auto-update agent files. Flag for human review.

### 6. Update Task File (Final)

```markdown
## Metadata
- **State**: DONE
- **Updated**: <timestamp>
- **Completed**: <timestamp>

## Progress Log
- [<timestamp>] SelfReflector: Analyzed workflow history
- [<timestamp>] SelfReflector: Extracted 2 new patterns
- [<timestamp>] SelfReflector: Documented 1 failure mode
- [<timestamp>] SelfReflector: Captured 1 human preference
- [<timestamp>] SelfReflector: Updated learnings files
- [<timestamp>] SelfReflector: WORKFLOW COMPLETE

## Learnings Captured
patterns:
  - "Decorator pattern for cross-cutting concerns"
  - "Fixture reuse in pytest"
failures:
  - "Thread safety in shared state (ISSUE-123)"
preferences:
  - "Explicit parameters over **kwargs"

## Final Summary
- **Total time:** X hours
- **Attempts:** coding=2, reviewing=1, testing=1
- **Key challenge:** Thread safety in rate limiter
- **Resolution:** Added locking mechanism
- **Human interventions:** 1 (plan approval)
```

### 7. Close the Loop

```bash
# Commit learning updates (if .claude/learnings is tracked)
git add .claude/learnings/ 2>/dev/null || true
git diff --cached --quiet || git commit -m "chore: update learnings from ISSUE-<number>

- Added decorator pattern documentation
- Documented thread safety failure mode
- Captured explicit parameters preference

Refs: #<issue-number>"

git push origin HEAD
```

### 8. Create or Update Pull Request

**IMPORTANT:** Always ensure a PR exists. Check first, create if needed.

```bash
# GitHub - Check if PR exists for current branch
PR_EXISTS=$(gh pr view --json number 2>/dev/null | jq -r '.number' || echo "")

if [ -z "$PR_EXISTS" ]; then
    # Create new PR if none exists
    gh pr create \
      --title "<type>: <title from issue>" \
      --body "$(cat <<'EOF'
Closes #<issue-number>

## Summary
<summary from task file>

## Changes
<list of changes>

## Testing
<test summary>

## Documentation
<doc updates>
EOF
)"
else
    # PR exists - mark as ready and update body
    gh pr ready
    gh pr edit --body "$(cat <<'EOF'
Closes #<issue-number>

## Summary
<summary from task file>

## Changes
<list of changes>

## Testing
<test summary>

## Documentation
<doc updates>
EOF
)"
fi

# GitLab
MR_EXISTS=$(glab mr view 2>/dev/null && echo "yes" || echo "")

if [ -z "$MR_EXISTS" ]; then
    glab mr create \
      --title "<type>: <title>" \
      --description "Closes #<issue-number>..." \
      --yes
else
    glab mr update --ready
fi
```

## Exit Criteria

✅ Workflow history analyzed
✅ Patterns extracted and documented
✅ Failures documented (if any)
✅ Human preferences captured (if any)
✅ Learning files updated
✅ PR created (if not exists) or marked as ready for review
✅ PR body updated with final summary
✅ Task state set to DONE  

## Output to Orchestrator

```yaml
status: success
state: DONE
learnings:
  patterns_added: 2
  failures_documented: 1
  preferences_captured: 1
pr:
  number: 45
  url: "https://github.com/user/repo/pull/45"
  status: ready_for_review  # was draft, now ready
summary:
  total_time: "2.5 hours"
  attempts: { coding: 2, reviewing: 1, testing: 1 }
  human_interventions: 1
message: "Workflow complete. PR #45 marked ready for final review."
```

## Learning Quality Guidelines

1. **Be specific** - Include file names, line numbers, concrete examples
2. **Be actionable** - Learnings should prevent future issues
3. **Don't duplicate** - Check existing learnings before adding
4. **Date everything** - Include timestamps for relevance
5. **Link to source** - Reference the issue/PR where learning originated
6. **Review periodically** - Old learnings may become outdated
