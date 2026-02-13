# CLAUDE.md - Project Intelligence

## Project Overview

This repository uses an autonomous development workflow powered by Claude Code agents.

---

## Plan Mode and `/dev` Integration

**Plan mode and `/dev` are unified.** Both entry points converge on the same
subagent pipeline (coder → reviewer → tester → documenter → self-reflector).

### Flow 1: User enters plan mode directly

When the user describes an implementation task and you enter plan mode:

1. Explore the codebase, design the solution, write the plan
2. User approves the plan (via ExitPlanMode)
3. **Immediately invoke `/dev --from-plan`** to hand off to the subagent pipeline
4. The architect creates the issue, branch, and draft PR from the approved plan
5. The pipeline runs: coder → reviewer → tester → documenter → self-reflector

**This is mandatory.** After plan approval for any implementation task, always
invoke `/dev --from-plan`. Do not implement directly in the main conversation.

### Flow 2: User invokes `/dev` explicitly

When the user runs `/dev <issue>` or `/dev <description>`:

1. The architect agent uses plan mode (EnterPlanMode) to explore and design
2. User approves the plan in plan mode
3. The architect creates the issue, branch, and draft PR
4. The pipeline runs: coder → reviewer → tester → documenter → self-reflector

### Summary

| Entry point | Who plans? | Who approves? | Then what? |
|-------------|-----------|---------------|------------|
| Plan mode → approve | You in plan mode | User approves plan | `/dev --from-plan` runs the pipeline |
| `/dev <task>` | Architect via plan mode | User approves plan | Pipeline runs automatically |

Both paths use plan mode for planning and the subagent pipeline for execution.

---

## Commands

### `/dev <issue-number | description>`

Initiates the autonomous development workflow.

**Usage:**
```bash
# From an existing issue
/dev #123
/dev 123

# From a description (creates issue automatically)
/dev Add rate limiting to the API endpoints
/dev Replace the logging system with structlog
```

### `/dev --from-plan`

Invoked automatically after plan mode approval. Takes the approved plan from
the conversation context and runs the full subagent pipeline.

### `/dev status`

Check status of current task.

### `/dev resume`

Resume a paused workflow (after human approval or failure).

### `/dev abort`

Cancel the current workflow.

**Workflow Stages:**
1. **Architect** → Creates issue, branch, draft PR (plan already approved)
2. **Coder** → Implements via TDD
3. **Reviewer** → Security, quality, architecture review
4. **Tester** → Executes tests, ensures passing
5. **Documenter** → Updates documentation
6. **SelfReflector** → Captures learnings

---

## Agent System

Agents are defined in `.claude/agents/`. Each agent:
- Operates in isolation (fresh context)
- Reads/writes to task state file in `.claude/tasks/`
- Has specific responsibilities and exit criteria

See: [Dev Command](.claude/commands/dev.md) for state machine logic.

---

## Plugins

The workflow uses official Claude plugins to reduce redundancy and leverage specialized capabilities.

### Enabled Plugins

| Plugin | Used By | Purpose |
|--------|---------|---------|
| `commit-commands` | Coder, SelfReflector | `/commit` for commits, `/commit-push-pr` for final PR |
| `code-review` | Reviewer | Automated code review with parallel agents and confidence scoring |
| `security-guidance` | Reviewer | Security best practices and vulnerability detection |
| `pr-review-toolkit` | Reviewer | PR review tools |
| `code-simplifier` | Coder | Code simplification and refactoring |
| `frontend-design` | Coder | Frontend design guidance (when applicable) |

### Plugin Commands in Workflow

- **Coder**: Uses `/commit` after each TDD cycle
- **Reviewer**: Uses `/code-review` for automated security/quality scanning
- **SelfReflector**: Marks draft PR as ready for review

---

## Project Conventions

### Git
- Branch naming: `<type>/<issue-number>-<short-description>`
- Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`
- Commits: Conventional commits format

### Python
- Formatter/Linter: Ruff
- Testing: Pytest
- Security: Bandit + pip-audit

### Git Platform
- Detect from remote: `gh` for GitHub, `glab` for GitLab
- Assume CLI is authenticated

### Git Rules
Absolutely DO NOT add CLAUDE/Claude/claude/Anthropic to any commit message, issue, or PR.

---

## Learnings

The SelfReflector agent maintains learned patterns in `.claude/learnings/`:
- `patterns.md` - Project-specific code patterns
- `failures.md` - Failure modes to avoid
- `preferences.md` - Human preferences from corrections

**These files are auto-updated. Review periodically.**

---

## File Locations

| Purpose | Location |
|---------|----------|
| Agent definitions | `.claude/agents/` |
| Dev command | `.claude/commands/dev.md` |
| Reusable skills | `.claude/skills/` |
| Active task state | `.claude/tasks/` |
| Learned patterns | `.claude/learnings/` |
