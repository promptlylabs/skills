---
name: create-branch
description: Create a git branch following Promptly Health naming conventions. Use when asked to "create a branch", "new branch", "start a branch", "make a branch", "switch to a new branch", or when starting new work on the default branch.
---

# Create Branch

Create a git branch following Promptly Health naming conventions.

## Step 1: Check Current State

```bash
git branch --show-current
git status --short
```

If on `main` with a clean working tree, proceed. If there are uncommitted changes, ask the user whether to stash, commit, or discard.

## Step 2: Ensure Up-to-Date

```bash
git pull origin main
```

## Step 3: Determine Branch Name

### Format

```
type/TICKET-123-short-description
```

### Components

**Type** — matches conventional commit types:

| Type | When |
|------|------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code restructure |
| `chore` | Maintenance, config |
| `docs` | Documentation |
| `test` | Test additions |
| `ci` | CI/CD changes |
| `perf` | Performance |

**Ticket** — Jira ticket ID (e.g., `PDP-123`, `INFRA-456`, `DATA-789`). If the user hasn't provided one, ask.

**Description** — 2-4 words, lowercase, hyphen-separated. Summarize the work.

### Rules

- All lowercase
- Hyphens only (no underscores, no slashes in description)
- Max 60 chars total
- Description must be meaningful — no `fix/PDP-123-fix` or `feat/PDP-456-update`

### Examples

```
feat/PDP-123-patient-enrollment-api
fix/PDP-456-login-redirect-loop
refactor/DATA-789-extract-dbt-macros
chore/INFRA-101-upgrade-go-1.22
docs/PDP-234-api-auth-guide
```

## Step 4: Create and Switch

```bash
git checkout -b type/TICKET-123-short-description
```

Confirm with:
```bash
git branch --show-current
```
