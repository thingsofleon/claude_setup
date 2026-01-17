# CLAUDE.md - Project Intelligence

## Project Overview

This repository uses an autonomous development workflow powered by Claude Code agents.

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

**Workflow Stages:**
1. **Architect** → Plans solution, creates/updates issue with checklist
2. **[HUMAN APPROVAL]** → Review plan before implementation
3. **Coder** → Implements via TDD
4. **Reviewer** → Security, quality, architecture review
5. **Tester** → Executes tests, ensures passing
6. **Documenter** → Updates documentation
7. **SelfReflector** → Captures learnings

### `/dev status`

Check status of current task.

### `/dev resume`

Resume a paused workflow (after human approval or failure).

### `/dev abort`

Cancel the current workflow.

---

## Agent System

Agents are defined in `.claude/agents/`. Each agent:
- Operates in isolation (fresh context)
- Reads/writes to task state file in `.claude/tasks/`
- Has specific responsibilities and exit criteria

See: [Dev Skill](.claude/skills/dev.md) for state machine logic.

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
| Reusable skills | `.claude/skills/` |
| Active task state | `.claude/tasks/` |
| Learned patterns | `.claude/learnings/` |


