# Human Preferences

> Auto-updated by SelfReflector agent when human corrections are detected.
> Last update: 2026-01-12

## Code Style

### Explicit Environment Variable Naming

**Source:** ISSUE-env-config
**Category:** Code Style

**Context:**
During planning for the .env config feature, the human interrupted to specify exact environment variable names.

**Preference:**
Use explicit, descriptive environment variable names that clearly indicate their purpose and category. For algorithm types, prefix with `DEFAULT_ALGORITHM_` followed by the uppercase category name.

**Example Before:**
```python
ENV_DISCRIMINATION_ALGO = "DISCRIMINATION_ALGO"
ENV_YIELD_ALGO = "YIELD_ALGORITHM"
```

**Example After (Preferred):**
```python
ENV_DEFAULT_ALGORITHM_DISCRIMINATION = "DEFAULT_ALGORITHM_DISCRIMINATION"
ENV_DEFAULT_ALGORITHM_YIELD_ESTIMATION = "DEFAULT_ALGORITHM_YIELD_ESTIMATION"
ENV_DEFAULT_ALGORITHM_O2D2 = "DEFAULT_ALGORITHM_O2D2"
```

**Reasoning:**
Consistent naming pattern makes configuration self-documenting and reduces confusion.

## Documentation

<!-- 
Preferences about documentation, like:
- Docstring format
- Comment level (more/less)
- README detail level
-->

## Testing

<!-- 
Preferences about testing approach, like:
- Coverage expectations
- Test naming
- When to use mocks
-->

## Git Workflow

<!-- 
Preferences about git usage, like:
- Commit message style
- Branch naming tweaks
- PR description format
-->

## Communication

<!-- 
Preferences about how agents communicate, like:
- Verbosity of explanations
- When to ask vs proceed
- Summary formats
-->

---

*This file is automatically updated from human corrections in issues/PRs.*
*Preferences help future workflows align with your expectations.*

## Template for New Entries

```markdown
### [Preference Name]

**Source:** ISSUE-XXX / PR-XXX
**Category:** [Code Style/Documentation/Testing/Git/Communication]

**Context:**
What triggered this preference discovery.

**Preference:**
The preferred approach.

**Example Before:**
```python
# What was done
```

**Example After (Preferred):**
```python
# What is preferred
```

**Reasoning:**
Why this is preferred.
```
