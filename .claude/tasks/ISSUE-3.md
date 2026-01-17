## Metadata
- **State**: CODING
- **Issue**: #3
- **Branch**: refactor/3-integrate-plugins
- **PR**: (pending)
- **Updated**: 2026-01-16T12:05:00Z
- **Attempts**: {"coding": 1, "reviewing": 0, "testing": 0}

## Plan

### Prerequisites
- [x] Review current agent implementations
- [x] Understand plugin capabilities

### Implementation Steps

- [ ] **Update Reviewer agent** (`reviewer.md`)
  - Replace manual Bandit/pip-audit/Ruff sections with `/code-review` invocation
  - Reference `security-guidance` plugin for security checklist
  - Keep architecture review (plan adherence) as manual step
  - Simplify from ~270 lines to ~100 lines

- [ ] **Update Coder agent** (`coder.md`)
  - Replace manual commit workflow with `/commit` command
  - Keep TDD workflow (RED-GREEN-REFACTOR) as core methodology
  - Remove redundant conventional commit documentation

- [ ] **Update Architect agent** (`architect.md`)
  - Replace manual `gh pr create` with draft PR workflow
  - Keep planning and issue creation logic

- [ ] **Update settings.json**
  - Add `WebFetch` to allowed permissions

- [ ] **Update CLAUDE.md**
  - Document plugin usage in workflow

### Test Plan
- [ ] Run `/dev` on a test issue to verify workflow still functions
- [ ] Verify reviewer uses `/code-review` output
- [ ] Verify coder uses `/commit` for commits

### Files to Modify
- `.claude/agents/reviewer.md` - Integrate code-review and security-guidance
- `.claude/agents/coder.md` - Integrate commit-commands
- `.claude/agents/architect.md` - Integrate commit-commands for PR
- `.claude/settings.json` - Add WebFetch permission
- `.claude/CLAUDE.md` - Document plugin usage

## Progress Log
- [2026-01-16T12:00:00Z] Orchestrator: Workflow started
- [2026-01-16T12:00:00Z] Architect: Analyzed enabled plugins in settings.json
- [2026-01-16T12:00:00Z] Architect: Identified redundancies in reviewer, coder, architect agents
- [2026-01-16T12:00:00Z] Architect: Created issue #3 with implementation plan
- [2026-01-16T12:00:00Z] Architect: Awaiting human approval
