# Dev Skill

## Purpose

Orchestrate the autonomous development workflow by managing state transitions and dispatching agents in a loop until completion.

## Commands

- `/dev <issue-number | description>` - Start development workflow
- `/dev status` - Check current state
- `/dev resume` - Resume paused workflow
- `/dev abort` - Cancel workflow

## Main Orchestration Loop

When `/dev` is invoked, run this loop:

```
WHILE state NOT IN [DONE, FAILED, AWAITING_APPROVAL]:
    1. Read state from task file
    2. Dispatch agent for current state
    3. Wait for agent completion
    4. Read updated state from task file
    5. Continue loop
```

## Implementation

### 1. Initialize or Resume

```bash
# Detect git platform
remote_url=$(git remote get-url origin 2>/dev/null)
if [[ "$remote_url" == *"github.com"* ]]; then
    GIT_CLI="gh"
elif [[ "$remote_url" == *"gitlab"* ]]; then
    GIT_CLI="glab"
fi
```

**If numeric input (e.g., `123` or `#123`):**
- Look for existing task file: `.claude/tasks/ISSUE-<number>.md`
- If exists and active → resume from current state
- If not exists → fetch issue and create task file

**If text input:**
- Create new issue using `$GIT_CLI issue create`
- Create task file with new issue number

### 2. State Dispatch Table

| Current State | Action | Next State (success) |
|---------------|--------|---------------------|
| `PLANNING` | Run Architect agent | `AWAITING_APPROVAL` |
| `AWAITING_APPROVAL` | Pause, notify user | (paused) |
| `CODING` | Run Coder agent | `REVIEWING` |
| `REVIEWING` | Run Reviewer agent | `TESTING` or `CODING` |
| `TESTING` | Run Tester agent | `DOCUMENTING` or `CODING` |
| `DOCUMENTING` | Run Documenter agent | `REFLECTING` |
| `REFLECTING` | Run SelfReflector agent | `DONE` |

### 3. Agent Dispatch

For each agent dispatch, use the Skill tool:

```
Skill(skill: "<agent-name>")
```

Agent names map to skills:
- `PLANNING` → `architect`
- `CODING` → `coder`
- `REVIEWING` → `reviewer`
- `TESTING` → `tester`
- `DOCUMENTING` → `documenter`
- `REFLECTING` → `reflect`

### 4. After Each Agent Completes

1. **Read the task file** to get updated state
2. **Check for failures:**
   - If agent failed and attempts < 3 → retry or go back to CODING
   - If attempts >= 3 → set state to FAILED, exit loop
3. **Check for pause:**
   - If state is AWAITING_APPROVAL → exit loop, notify user to `/dev resume`
4. **Continue** to next iteration

### 5. State Transition Logic

```python
# Pseudo-code for state transitions
def get_next_state(current_state, agent_result, task_metadata):
    if current_state == "PLANNING":
        return "AWAITING_APPROVAL"

    elif current_state == "CODING":
        return "REVIEWING"

    elif current_state == "REVIEWING":
        if agent_result.success:
            return "TESTING"
        else:
            task_metadata.attempts["coding"] += 1
            if task_metadata.attempts["coding"] >= 3:
                return "FAILED"
            return "CODING"  # Back to coder to fix issues

    elif current_state == "TESTING":
        if agent_result.success:
            return "DOCUMENTING"
        else:
            task_metadata.attempts["testing"] += 1
            if task_metadata.attempts["testing"] >= 3:
                return "FAILED"
            return "CODING"  # Back to coder to fix tests

    elif current_state == "DOCUMENTING":
        return "REFLECTING"

    elif current_state == "REFLECTING":
        return "DONE"
```

## Execution Flow

### `/dev 123` or `/dev Add new feature`

```
1. Parse input
2. Create/load task file
3. Set initial state to PLANNING if new
4.
   ┌──────────────────────────────────────────────────────────┐
   │ ORCHESTRATION LOOP                                       │
   │                                                          │
   │ state = read_state_from_task_file()                      │
   │                                                          │
   │ WHILE state NOT IN [DONE, FAILED, AWAITING_APPROVAL]:    │
   │     │                                                    │
   │     ├─► Log: "Dispatching {agent} for state {state}"     │
   │     │                                                    │
   │     ├─► Invoke agent skill                               │
   │     │                                                    │
   │     ├─► Wait for agent completion                        │
   │     │                                                    │
   │     ├─► state = read_state_from_task_file()              │
   │     │                                                    │
   │     └─► Check retry limits, handle failures              │
   │                                                          │
   │ END WHILE                                                │
   │                                                          │
   │ IF state == AWAITING_APPROVAL:                           │
   │     Print "Plan ready. Review and run /dev resume"  │
   │ ELIF state == DONE:                                      │
   │     Print "Workflow completed successfully!"             │
   │ ELIF state == FAILED:                                    │
   │     Print "Workflow failed. Check task file for details" │
   └──────────────────────────────────────────────────────────┘
```

### `/dev status`

1. Find active task file in `.claude/tasks/`
2. Read and display:
   - Current state
   - Issue number and branch
   - Progress (completed plan items)
   - Recent log entries

### `/dev resume`

1. Find task in `AWAITING_APPROVAL` state
2. Confirm user has reviewed the plan
3. Update state to `CODING`
4. Re-enter orchestration loop

### `/dev abort`

1. Find active task file
2. Set state to `FAILED`
3. Add abort entry to progress log
4. Optionally close/label the issue

## Task File Location

`.claude/tasks/ISSUE-<number>.md`

## Important Notes

1. **The loop is the key** - Don't exit after one agent; keep looping until terminal state
2. **Read state from file** - Agents update the task file; orchestrator reads it
3. **Handle all transitions** - Including failures that loop back to CODING
4. **Pause cleanly** - AWAITING_APPROVAL exits loop but preserves state for resume
5. **Log everything** - Append to Progress Log for debugging

## Error Recovery

If the workflow crashes mid-execution:
1. Run `/dev status` to see current state
2. Run `/dev resume` to continue from last state
3. The task file preserves all progress
