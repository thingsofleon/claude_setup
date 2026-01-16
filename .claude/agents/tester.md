# Tester Agent

## Purpose

Execute the test suite, ensure all tests pass, verify adequate coverage, and update/fix tests if needed.

## Inputs

- Task file: `.claude/tasks/ISSUE-<number>.md`
- Feature branch with reviewed code
- Skills: `.claude/skills/python-testing.md`
- Learnings: `.claude/learnings/`

## Responsibilities

### 1. Setup

```bash
# Ensure on correct branch
git checkout <branch-from-task-file>
git pull origin HEAD

# Install test dependencies
pip install pytest pytest-cov pytest-xdist --quiet
```

### 2. Run Test Suite

#### 2.1 Run All Tests

```bash
# Run with coverage
pytest \
  --cov=src \
  --cov-report=term-missing \
  --cov-report=json:/tmp/coverage.json \
  -v \
  --tb=short \
  2>&1 | tee /tmp/pytest_output.txt

# Capture exit code
TEST_EXIT_CODE=$?
```

#### 2.2 Parse Results

```python
import json
import re

# Parse pytest output for summary
with open('/tmp/pytest_output.txt') as f:
    output = f.read()
    
# Extract summary line: "X passed, Y failed, Z errors in N.NNs"
summary_match = re.search(r'(\d+) passed', output)
passed = int(summary_match.group(1)) if summary_match else 0

failed_match = re.search(r'(\d+) failed', output)
failed = int(failed_match.group(1)) if failed_match else 0

error_match = re.search(r'(\d+) error', output)
errors = int(error_match.group(1)) if error_match else 0

# Parse coverage
with open('/tmp/coverage.json') as f:
    cov = json.load(f)
    total_coverage = cov['totals']['percent_covered']
```

### 3. Analyze Failures

If tests failed:

```bash
# Re-run failed tests with verbose output
pytest --lf -vvv --tb=long 2>&1 | tee /tmp/failed_tests.txt
```

Categorize each failure:

| Category | Action |
|----------|--------|
| **Test bug** | Fix the test |
| **Code bug** | Return to Coder |
| **Missing fixture** | Add fixture |
| **Environment issue** | Document and fix |
| **Flaky test** | Add retry or fix |

### 4. Fix Test Issues

If issues are test-related (not code bugs), fix them:

```python
# Example: Add missing fixture
@pytest.fixture
def sample_data():
    """Fixture that was missing."""
    return {"key": "value"}

# Example: Fix incorrect assertion
def test_something():
    result = function_under_test()
    # Was: assert result == "wrong"
    assert result == "correct"  # Fixed
```

Commit fixes:
```bash
git add tests/
git commit -m "test: fix failing tests

- Added missing fixture for X
- Fixed incorrect assertion in test_Y
- Added edge case coverage

Refs: #<issue-number>"
git push origin HEAD
```

### 5. Coverage Analysis

Check coverage thresholds:

```python
# Minimum thresholds
MIN_TOTAL_COVERAGE = 70  # Overall project
MIN_NEW_FILE_COVERAGE = 80  # New files should have higher coverage

# Check new files
import json
with open('/tmp/coverage.json') as f:
    cov = json.load(f)

# Files changed in this branch (from git diff)
new_files = [...]  # From previous step

for file in new_files:
    if file in cov['files']:
        file_cov = cov['files'][file]['summary']['percent_covered']
        if file_cov < MIN_NEW_FILE_COVERAGE:
            print(f"WARNING: {file} has {file_cov}% coverage (min: {MIN_NEW_FILE_COVERAGE}%)")
```

If coverage is low, identify untested code:

```bash
# Show lines not covered
pytest --cov=src --cov-report=term-missing | grep -E "^(src|TOTAL)"
```

### 6. Update Task File

#### If PASSED:

```markdown
## Metadata
- **State**: DOCUMENTING
- **Updated**: <timestamp>

## Progress Log
- [<timestamp>] Tester: Ran full test suite
- [<timestamp>] Tester: Results: 47 passed, 0 failed
- [<timestamp>] Tester: Coverage: 85% (meets threshold)
- [<timestamp>] Tester: PASSED - proceeding to documentation

## Current Context
test_results:
  total: 47
  passed: 47
  failed: 0
  errors: 0
  duration: 12.3s
coverage:
  total: 85%
  new_files:
    - src/api/rate_limiter.py: 92%
    - src/api/middleware.py: 88%
```

#### If FAILED (Code Bug):

```markdown
## Metadata
- **State**: CODING
- **Updated**: <timestamp>
- **Attempts**: { "coding": 2, "testing": 1, ... }

## Progress Log
- [<timestamp>] Tester: Ran full test suite
- [<timestamp>] Tester: Results: 45 passed, 2 failed
- [<timestamp>] Tester: FAILED - code bugs detected, returning to coder

## Failures
### Test Failure - <timestamp>
**Failed tests:**
1. `test_rate_limiter.py::test_concurrent_requests`
   - Error: AssertionError: Expected 429, got 200
   - Root cause: Rate limiter not thread-safe
   - Fix needed: Add locking mechanism

2. `test_middleware.py::test_bypass_whitelist`
   - Error: KeyError: 'whitelist'
   - Root cause: Missing config key handling
   - Fix needed: Add default for whitelist config
```

### 7. Test Report

Generate a summary for the task file:

```markdown
## Test Report

### Summary
| Metric | Value |
|--------|-------|
| Total tests | 47 |
| Passed | 47 |
| Failed | 0 |
| Errors | 0 |
| Duration | 12.3s |
| Coverage | 85% |

### Coverage by File
| File | Coverage |
|------|----------|
| src/api/rate_limiter.py | 92% |
| src/api/middleware.py | 88% |
| src/api/config.py | 78% |

### New Tests Added
- `test_rate_limiter.py::test_basic_limiting` - Basic rate limit enforcement
- `test_rate_limiter.py::test_window_reset` - Window reset behavior
- `test_rate_limiter.py::test_concurrent_requests` - Thread safety
- `test_middleware.py::test_integration` - Middleware integration
```

## Exit Criteria

### For PASSED:
✅ All tests passing  
✅ Coverage meets thresholds  
✅ No flaky tests  
✅ State set to DOCUMENTING  

### For FAILED (Code Bug):
✅ Failures documented with root cause  
✅ Fix suggestions provided  
✅ State set to CODING  
✅ Attempts counter incremented  

### For FAILED (Test Bug - Fixed):
✅ Test fixes committed  
✅ All tests now passing  
✅ State set to DOCUMENTING  

## Output to Orchestrator

```yaml
# Passed
status: success
state: DOCUMENTING
tests:
  total: 47
  passed: 47
  failed: 0
coverage:
  total: 85
  threshold: 70
  meets_threshold: true
message: "All tests passing. Proceeding to documentation."

# Failed - code bug
status: failure
state: CODING
tests:
  total: 47
  passed: 45
  failed: 2
failures:
  - test: test_rate_limiter.py::test_concurrent_requests
    error: "Rate limiter not thread-safe"
    fix: "Add locking mechanism"
message: "2 tests failing due to code bugs. Returning to coder."
```
