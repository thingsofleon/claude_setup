# Failure Modes to Avoid

> Auto-updated by SelfReflector agent when failures occur.
> Last update: (not yet updated)

## Security Failures

<!-- 
Document security issues found during review, like:
- Hardcoded secrets
- SQL injection patterns
- Missing auth checks
-->

## Code Failures

<!-- 
Document code issues that caused test/review failures, like:
- Thread safety problems
- Resource leaks
- Edge cases missed
-->

## Test Failures

<!-- 
Document testing issues, like:
- Flaky tests and their causes
- Missing fixtures
- Environment-specific failures
-->

## Integration Failures

<!-- 
Document issues discovered during integration, like:
- API contract mismatches
- Database migration issues
- Configuration problems
-->

---

*This file is automatically updated when workflow failures occur.*
*Use this to avoid repeating past mistakes.*

## Template for New Entries

```markdown
### [Descriptive Title]

**Seen in:** ISSUE-XXX
**Agent:** [Which agent detected it]
**Severity:** [High/Medium/Low]

**Problem:**
What went wrong.

**Symptom:**
How it manifested.

**Root Cause:**
Why it happened.

**Solution:**
How it was fixed.

**Prevention:**
How to avoid in future.
```
