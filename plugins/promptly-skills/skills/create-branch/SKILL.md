---
name: create-branch
description: Create a git branch following Promptly Health naming conventions. Use when asked to "create a branch", "new branch", "start a branch", "make a branch", "switch to a new branch", or when starting new work on the default branch.
argument-hint: '[optional description of the work]'
---

# Create Branch

Create a git branch following Promptly Health naming conventions.

## Step 1: Check Current State

```bash
git branch --show-current
git status --short
```

If there are uncommitted changes, ask the user whether to stash, commit, or discard. Stash if requested — changes will be restored after branch creation.

## Step 2: Detect Default Branch

```bash
gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'
```

If `gh` fails, fall back to `git branch --list main master` — use the one that exists. If both or neither exist, ask the user.

If `git branch --show-current` is empty (detached HEAD), show the current commit (`git rev-parse --short HEAD`) and ask whether to branch from it or switch to the default branch first.

If the current branch is not the default branch, warn the user and ask whether to branch from the current branch or switch to the default branch first.

## Step 3: Ensure Up-to-Date

```bash
git pull origin <default-branch>
```

## Step 4: Determine Branch Name

**If `$ARGUMENTS` is provided**, use it as the description of the work.

**If no arguments**, check for local changes:

```bash
git diff
git diff --cached
```

- **Changes exist**: read the diff to understand the work and generate a description.
- **No changes**: ask the user what they are about to work on.

### Format

```
type/TICKET-123-short-description
```

### Components

**Type** — matches conventional commit types:

| Type | When |
|------|------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `refactor` | Code restructure |
| `chore` | Maintenance, config, deps |
| `docs` | Documentation |
| `test` | Test additions |
| `ci` | CI/CD changes |
| `perf` | Performance |
| `build` | Build system |

When unsure: `feat` for new things, `refactor` for restructuring existing things, `chore` only when updating/maintaining something that already exists.

**Ticket** — Jira ticket ID (e.g., `PDP-123`, `INFRA-456`, `DATA-789`). If the user hasn't provided one, ask.

**Description** — 2-5 words, lowercase, hyphen-separated. Describe the change, not file names.

### Rules

- All lowercase
- Only ASCII letters, digits, and hyphens — no underscores, spaces, dots, or colons
- Max 60 chars total
- Description must be meaningful — no `fix/PDP-123-fix` or `feat/PDP-456-update`

### Examples

| Work description | Branch name |
|-----------------|-------------|
| Patient enrollment API | `feat/PDP-123-patient-enrollment-api` |
| Login redirect loop bug | `fix/PDP-456-login-redirect-loop` |
| Extract dbt macros | `refactor/DATA-789-extract-dbt-macros` |
| Upgrade Go to 1.22 | `chore/INFRA-101-upgrade-go-1.22` |
| API auth documentation | `docs/PDP-234-api-auth-guide` |

## Step 5: Confirm with User

Present the proposed branch name and ask if they want to use it, modify it, or change the type.

## Step 6: Create the Branch

Before creating, check that the name doesn't already exist:

```bash
git show-ref --verify --quiet refs/heads/<branch-name> && echo "EXISTS" || echo "OK"
git show-ref --verify --quiet refs/remotes/origin/<branch-name> && echo "REMOTE EXISTS" || echo "OK"
```

If it exists, ask the user to choose a different name.

Create the branch:

```bash
git checkout -b <branch-name>
```

Restore any stashed changes:

```bash
git stash pop  # only if changes were stashed in Step 1
```

Confirm:
```bash
git branch --show-current
```
