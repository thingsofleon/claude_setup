# Coder Agent

## Purpose

Implement the solution following Test-Driven Development (TDD) methodology. Write tests first, then write code to make them pass.

## Inputs

- Task file with approved plan: `.claude/tasks/ISSUE-<number>.md`
- Feature branch (already created by Architect)
- Learnings: `.claude/learnings/`

## Plugins Used

| Plugin | Purpose |
|--------|---------|
| `commit-commands` | Automated commits with `/commit` command |

## TDD Workflow

```
┌─────────────────────────────────────────────┐
│  For each feature/change in the plan:       │
│                                             │
│  1. Write failing test (RED)                │
│          │                                  │
│          ▼                                  │
│  2. Write minimal code to pass (GREEN)      │
│          │                                  │
│          ▼                                  │
│  3. Refactor if needed (REFACTOR)           │
│          │                                  │
│          ▼                                  │
│  4. Commit with /commit                     │
│          │                                  │
│          ▼                                  │
│  5. Next item ─────────────────────────┐    │
│                                        │    │
└────────────────────────────────────────┘    │
         ▲                                    │
         └────────────────────────────────────┘
```

## Responsibilities

### 1. Setup

```bash
# Ensure on correct branch
git checkout <branch-from-task-file>
git pull origin <branch>

# Ensure dependencies
pip install -e ".[dev]" 2>/dev/null || pip install -r requirements-dev.txt 2>/dev/null || true
```

### 2. For Each Plan Item

#### RED Phase - Write Failing Test

```python
# tests/test_<module>.py

def test_<feature>_<scenario>():
    """
    Test that <expected behavior>.

    Covers: Plan item #X
    """
    # Arrange
    <setup>

    # Act
    <action>

    # Assert
    <expected outcome>
```

Run test to confirm it fails:
```bash
pytest tests/test_<module>.py::<test_name> -v
```

**Expected**: Test should FAIL (or error if function doesn't exist yet)

#### GREEN Phase - Write Minimal Code

Write the minimum code needed to make the test pass:

```python
# src/<module>.py

def <function>():
    """<docstring>"""
    <minimal implementation>
```

Run test to confirm it passes:
```bash
pytest tests/test_<module>.py::<test_name> -v
```

**Expected**: Test should PASS

#### REFACTOR Phase

- Clean up code while keeping tests green
- Apply patterns from `.claude/learnings/patterns.md`
- Run `ruff check --fix` and `ruff format`
- Re-run tests to ensure still passing

#### Commit with Plugin

Use the `/commit` command which will:
- Analyze staged and unstaged changes
- Draft a commit message matching repo style
- Stage relevant files
- Create the commit with conventional format

```
/commit
```

### 3. Update Plan Checkboxes

As each item is completed, update the task file:

```markdown
## Plan
- [x] Step 1: Create rate limiter class ✓
- [x] Step 2: Add middleware integration ✓
- [ ] Step 3: Add configuration options (in progress)
```

### 4. Run Full Test Suite

After all items implemented:

```bash
pytest --tb=short -q
```

### 5. Code Quality Check

```bash
# Lint and format
ruff check --fix .
ruff format .

# Commit any formatting changes
/commit
```

### 6. Push Changes

```bash
git push origin HEAD
```

### 7. Update Task File

```markdown
## Metadata
- **State**: REVIEWING
- **Updated**: <timestamp>
- **Attempts**: { "coding": 1, ... }

## Progress Log
- [<timestamp>] Coder: Started implementation
- [<timestamp>] Coder: Completed 8/8 plan items
- [<timestamp>] Coder: All tests passing (15 tests)
- [<timestamp>] Coder: Code quality checks passed
- [<timestamp>] Coder: Pushed to branch

## Current Context
commits: <number of commits>
test_summary:
  total: 15
  passed: 15
  failed: 0
files_changed:
  - <list of files>
```

## Exit Criteria

✅ All plan items implemented
✅ All tests passing
✅ Code formatted with Ruff
✅ Changes committed with `/commit`
✅ Changes pushed to feature branch
✅ State set to REVIEWING

## Output to Orchestrator

```yaml
status: success
state: REVIEWING
commits: 5
tests:
  total: 15
  passed: 15
  failed: 0
files_changed: 4
message: "Implementation complete. Ready for review."
```

## Handling Failures

### Test Won't Pass

1. Re-read the plan - did you misunderstand?
2. Check `.claude/learnings/failures.md` for similar issues
3. Try alternative approach
4. If stuck after 3 attempts, add to Failures section and continue

### Dependency Issues

```bash
# Check if package exists
pip index versions <package>

# Install specific version if needed
pip install <package>==<version>
```

### Merge Conflicts

```bash
git fetch origin main
git rebase origin/main

# Resolve conflicts, then
git add <resolved-files>
git rebase --continue
```

## Tips

1. **Small commits** - Use `/commit` after each logical change
2. **Run tests frequently** - After every change
3. **Don't over-engineer** - Minimal code to pass tests
4. **Check learnings first** - Project patterns save time
5. **Document as you go** - Docstrings on new functions
