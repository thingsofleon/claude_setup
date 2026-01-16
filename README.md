# Autonomous Development Workflow

A Claude Code agent workflow system that automates the software development lifecycle.

## Adding to New Project
1. Go to the local claude_setup repo
   - `git checkout main`
   - `git pull`
2. Go to the New Project repo
   - `rsync -av --exclude='.git' --exclude='README.md' <path to claude_setup> .`

## Quick Start

```bash
# Start development from a description
/dev Add rate limiting to the API

# Start from an existing issue
/dev 123

# Check progress
/dev status

# Resume after approving plan
/dev resume

# Cancel workflow
/dev abort
```

## How It Works

1. **You describe what you want** → `/dev Add user authentication`
2. **Architect agent plans** → Creates issue with implementation checklist
3. **You review and approve** → Comment "approved" or run `/dev resume`
4. **Agents execute autonomously:**
   - **Coder** → Implements via TDD (tests first)
   - **Reviewer** → Security scan + code quality
   - **Tester** → Runs tests, checks coverage
   - **Documenter** → Updates docs and changelog
   - **SelfReflector** → Captures learnings
5. **You review the PR** → Merge when satisfied

## Your Role

Act as a **Product Manager**:
- Define what you want built
- Review and approve plans
- Review final PRs

The agents handle implementation details, testing, and documentation.

## Workflow States

```
PLANNING → AWAITING_APPROVAL → CODING → REVIEWING → TESTING → DOCUMENTING → REFLECTING → DONE
```

- **AWAITING_APPROVAL** is the only human checkpoint
- Failed reviews/tests automatically retry (max 3 attempts)
- All state persisted in `.claude/tasks/` for recovery

## Project Structure

```
.claude/
├── CLAUDE.md              # Entry point and conventions
├── settings.json          # Permissions
├── agents/                # Agent definitions
│   ├── architect.md
│   ├── coder.md
│   ├── reviewer.md
│   ├── tester.md
│   ├── documenter.md
│   └── self-reflector.md
├── skills/                # Orchestration
│   ├── dev.md             # Main workflow
│   └── git-workflow.md    # Git conventions
├── learnings/             # Auto-updated knowledge
│   ├── patterns.md
│   ├── failures.md
│   └── preferences.md
└── tasks/                 # Runtime state
```

## Conventions

- **Branches**: `<type>/<issue-number>-<description>` (e.g., `feat/123-add-auth`)
- **Commits**: Conventional commits format
- **Python**: Ruff (lint/format), Pytest (testing), Bandit (security)
- **Platforms**: Auto-detects GitHub (`gh`) or GitLab (`glab`)
