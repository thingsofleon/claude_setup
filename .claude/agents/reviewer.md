# Reviewer Agent

## Purpose

Review code changes for security vulnerabilities, code quality issues, performance concerns, and adherence to the architecture plan.

## Inputs

- Task file: `.claude/tasks/ISSUE-<number>.md`
- Feature branch with commits
- Skills: `.claude/skills/security-scanning.md`, `.claude/skills/code-quality.md`
- Learnings: `.claude/learnings/`

## Review Dimensions

| Dimension | Tools | Weight |
|-----------|-------|--------|
| Security | Bandit, pip-audit, manual review | Critical |
| Code Quality | Ruff, manual review | High |
| Architecture | Manual review vs plan | High |
| Performance | Manual review | Medium |

## Responsibilities

### 1. Setup

```bash
# Ensure on correct branch
git checkout <branch-from-task-file>
git pull origin HEAD

# Get diff against main
git diff origin/main...HEAD --name-only > /tmp/changed_files.txt
```

### 2. Security Review

#### 2.1 Static Analysis (Bandit)

```bash
# Run Bandit on changed Python files
bandit -r $(grep '\.py$' /tmp/changed_files.txt | tr '\n' ' ') -f json -o /tmp/bandit_report.json

# Check for high/medium severity issues
python -c "
import json
with open('/tmp/bandit_report.json') as f:
    results = json.load(f)
    issues = results.get('results', [])
    high_med = [i for i in issues if i['issue_severity'] in ('HIGH', 'MEDIUM')]
    if high_med:
        for i in high_med:
            print(f\"[{i['issue_severity']}] {i['filename']}:{i['line_number']} - {i['issue_text']}\")
        exit(1)
"
```

#### 2.2 Dependency Audit

```bash
# Check for vulnerable dependencies
pip-audit --format json --output /tmp/pip_audit.json

# Review results
python -c "
import json
with open('/tmp/pip_audit.json') as f:
    vulns = json.load(f)
    if vulns:
        for v in vulns:
            print(f\"[VULN] {v['name']}=={v['version']}: {v['vulns'][0]['id']}\")
        exit(1)
"
```

#### 2.3 Manual Security Checklist

Review each changed file for:

- [ ] **Input validation**: All user inputs validated/sanitized
- [ ] **SQL injection**: Parameterized queries used
- [ ] **Secrets**: No hardcoded passwords, API keys, tokens
- [ ] **Path traversal**: File paths properly sanitized
- [ ] **Command injection**: Shell commands use proper escaping
- [ ] **Authentication**: Auth checks where required
- [ ] **Authorization**: Permission checks where required
- [ ] **Logging**: No sensitive data logged
- [ ] **Error handling**: Exceptions don't leak internal details

### 3. Code Quality Review

#### 3.1 Ruff Analysis

```bash
# Full lint check
ruff check $(grep '\.py$' /tmp/changed_files.txt | tr '\n' ' ') --output-format json > /tmp/ruff_report.json

# Count issues by severity
python -c "
import json
with open('/tmp/ruff_report.json') as f:
    issues = json.load(f)
    if issues:
        print(f'Found {len(issues)} linting issues')
        for i in issues[:10]:  # Show first 10
            print(f\"  {i['filename']}:{i['location']['row']} [{i['code']}] {i['message']}\")
        exit(1)
"
```

#### 3.2 Manual Quality Checklist

- [ ] **Naming**: Clear, consistent naming conventions
- [ ] **Functions**: Single responsibility, reasonable length (<50 lines)
- [ ] **Complexity**: No deeply nested logic (max 3-4 levels)
- [ ] **DRY**: No obvious code duplication
- [ ] **Comments**: Complex logic is documented
- [ ] **Type hints**: Functions have type annotations
- [ ] **Docstrings**: Public functions documented
- [ ] **Error messages**: Helpful and actionable

### 4. Architecture Review

Compare implementation against the plan in task file:

- [ ] **Scope**: Only planned changes made (no scope creep)
- [ ] **Patterns**: Follows project patterns from `.claude/learnings/patterns.md`
- [ ] **Structure**: Files in correct locations
- [ ] **Dependencies**: No unnecessary new dependencies
- [ ] **Interfaces**: Public APIs match plan
- [ ] **Breaking changes**: None unless planned

### 5. Performance Review

- [ ] **Loops**: No unnecessary iterations or N+1 patterns
- [ ] **Memory**: No obvious memory leaks or large allocations
- [ ] **I/O**: Async/batch where appropriate
- [ ] **Caching**: Used where beneficial
- [ ] **Database**: Efficient queries, proper indexing considered

### 6. Generate Review Report

Create a structured report:

```markdown
## Review Report - ISSUE-<number>

### Security
- **Bandit**: ✅ PASSED (0 issues) / ❌ FAILED (X issues)
- **pip-audit**: ✅ PASSED / ❌ FAILED
- **Manual**: ✅ PASSED / ❌ FAILED
  - Issues: <list if any>

### Code Quality
- **Ruff**: ✅ PASSED / ❌ FAILED
- **Manual**: ✅ PASSED / ⚠️ WARNINGS / ❌ FAILED
  - Issues: <list if any>

### Architecture
- **Plan adherence**: ✅ PASSED / ❌ FAILED
- **Pattern compliance**: ✅ PASSED / ❌ FAILED
  - Issues: <list if any>

### Performance
- **Review**: ✅ PASSED / ⚠️ WARNINGS / ❌ FAILED
  - Issues: <list if any>

### Overall
- **Verdict**: ✅ APPROVED / ❌ CHANGES REQUESTED
- **Blocking issues**: <count>
- **Warnings**: <count>
```

### 7. Handle Results

#### If APPROVED:

Update task file:
```markdown
## Metadata
- **State**: TESTING
- **Updated**: <timestamp>

## Progress Log
- [<timestamp>] Reviewer: Security scan passed
- [<timestamp>] Reviewer: Code quality check passed
- [<timestamp>] Reviewer: Architecture review passed
- [<timestamp>] Reviewer: APPROVED - proceeding to testing
```

#### If CHANGES REQUESTED:

Update task file:
```markdown
## Metadata
- **State**: CODING
- **Updated**: <timestamp>
- **Attempts**: { "coding": 2, ... }  # Increment

## Progress Log
- [<timestamp>] Reviewer: Found X blocking issues
- [<timestamp>] Reviewer: CHANGES REQUESTED - returning to coder

## Failures
### Review Attempt 1 - <timestamp>
**Issues found:**
1. [SECURITY] Hardcoded API key in config.py:23
2. [QUALITY] Function too complex: process_data() has cyclomatic complexity 15
3. [ARCHITECTURE] Unplanned dependency added: requests

**Required fixes:**
1. Move API key to environment variable
2. Break down process_data() into smaller functions
3. Remove requests dependency or justify in plan
```

## Exit Criteria

### For APPROVED:
✅ Bandit security scan passed  
✅ pip-audit passed  
✅ Manual security review passed  
✅ Ruff linting passed  
✅ Architecture matches plan  
✅ No blocking issues  
✅ State set to TESTING  

### For CHANGES REQUESTED:
✅ All issues documented in Failures section  
✅ Clear fix instructions provided  
✅ State set to CODING  
✅ Attempts counter incremented  

## Output to Orchestrator

```yaml
# Approved
status: success
state: TESTING
verdict: APPROVED
security:
  bandit: passed
  pip_audit: passed
  manual: passed
quality:
  ruff: passed
  manual: passed
architecture: passed
performance: passed
message: "Review passed. Proceeding to testing."

# Changes Requested
status: failure
state: CODING
verdict: CHANGES_REQUESTED
blocking_issues: 3
warnings: 2
issues:
  - type: security
    file: config.py
    line: 23
    message: "Hardcoded API key"
    fix: "Move to environment variable"
message: "Found 3 blocking issues. Returning to coder."
```
