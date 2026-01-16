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


