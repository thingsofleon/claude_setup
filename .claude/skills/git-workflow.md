# Skill: Git Workflow

## Branch Naming Convention

```
<type>/<issue-number>-<short-description>
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Code refactoring
- `docs` - Documentation only
- `test` - Test additions/changes
- `chore` - Maintenance tasks

**Examples:**
```
feat/123-add-rate-limiting
fix/456-null-pointer-exception
refactor/789-extract-auth-module
```

## Commit Message Format

[Conventional Commits](https://www.conventionalcommits.org/)

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

**Examples:**
```bash
# Simple
git commit -m "feat(api): add rate limiting endpoint"

# With body
git commit -m "fix(auth): handle expired tokens gracefully

Previously, expired tokens caused a 500 error. Now returns 401
with a clear message instructing the user to refresh.

Refs: #456"

# Breaking change
git commit -m "feat(api)!: change authentication to OAuth2

BREAKING CHANGE: Basic auth is no longer supported.
See migration guide in docs/migration-v2.md

Refs: #789"
```

## Platform Detection

```bash
detect_git_platform() {
    local remote_url=$(git remote get-url origin 2>/dev/null)
    
    if [[ "$remote_url" == *"github.com"* ]]; then
        echo "github"
    elif [[ "$remote_url" == *"gitlab"* ]]; then
        echo "gitlab"
    else
        echo "unknown"
    fi
}

get_git_cli() {
    case $(detect_git_platform) in
        github) echo "gh" ;;
        gitlab) echo "glab" ;;
        *) echo "" ;;
    esac
}
```

## Common Operations

### Create Branch

```bash
# From main
git checkout main
git pull origin main
git checkout -b feat/123-description
```

### Push Branch

```bash
# First push (set upstream)
git push -u origin HEAD

# Subsequent pushes
git push
```

### Create Issue

```bash
# GitHub
gh issue create \
  --title "feat: Add rate limiting" \
  --body "## Problem
...

## Solution
..."

# GitLab
glab issue create \
  --title "feat: Add rate limiting" \
  --description "..."
```

### View Issue

```bash
# GitHub
gh issue view 123
gh issue view 123 --json title,body,state,comments

# GitLab
glab issue view 123
```

### Comment on Issue

```bash
# GitHub
gh issue comment 123 --body "Implementation plan ready for review"

# GitLab
glab issue note 123 --message "..."
```

### Create Pull/Merge Request

```bash
# GitHub
gh pr create \
  --title "feat: Add rate limiting" \
  --body "Closes #123" \
  --base main

# GitLab
glab mr create \
  --title "feat: Add rate limiting" \
  --description "Closes #123" \
  --target-branch main
```

### View PR/MR

```bash
# GitHub
gh pr view
gh pr view --json state,reviews,comments

# GitLab
glab mr view
```

### Check CI Status

```bash
# GitHub
gh pr checks

# GitLab
glab ci status
```

## Rebase Workflow

```bash
# Update with latest main
git fetch origin main
git rebase origin/main

# If conflicts:
# 1. Resolve conflicts in each file
# 2. Stage resolved files
git add <resolved-files>
# 3. Continue rebase
git rebase --continue

# Force push after rebase
git push --force-with-lease
```

## Squash Commits (if needed)

```bash
# Interactive rebase to squash
git rebase -i origin/main

# In editor, change 'pick' to 'squash' (or 's') for commits to combine
# Save and edit the combined commit message
```

## Undo Operations

```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Undo staged changes
git restore --staged <file>

# Undo unstaged changes
git restore <file>

# Undo a pushed commit (creates revert commit)
git revert <commit-hash>
```

## Stash Operations

```bash
# Save work in progress
git stash push -m "WIP: rate limiter"

# List stashes
git stash list

# Apply and remove stash
git stash pop

# Apply without removing
git stash apply stash@{0}
```
