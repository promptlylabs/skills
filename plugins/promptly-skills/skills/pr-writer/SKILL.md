---
name: pr-writer
description: ALWAYS use this skill when creating or updating pull requests — never create or edit a PR directly without it. Follows Promptly Health conventions for PR titles, descriptions, and Jira linking. Trigger on any create PR, open PR, submit PR, make PR, update PR, push and create PR, or prepare changes for review task.
---

# PR Writer

ALWAYS follow this workflow when creating or updating PRs.

**Requires**: GitHub CLI (`gh`) authenticated.

## Prerequisites

Before creating a PR, check for uncommitted changes:

```bash
git status --porcelain
```

If there are uncommitted changes (modified, added, or untracked files that should be included), invoke the `promptly-skills:commit` skill first to commit them properly.

## Phase 1: Verify Branch State

Detect the default branch — use it in subsequent commands:

```bash
gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'
```

Review what will be included:

```bash
git status
git log <default-branch>..HEAD --oneline
git diff <default-branch>...HEAD --stat
```

Ensure:
- All changes are committed
- Branch is up to date with remote
- Changes are rebased on the base branch if needed

Read the full diff to understand what changed and why. If a `plan.md` exists in the repo or working directory, read it — its content will be embedded in the PR.

## Phase 2: Determine Jira Ticket

Check the branch name for a ticket reference:

```bash
git branch --show-current
```

Extract the ticket ID (e.g., `PDP-123` from `feat/PDP-123-patient-enrollment-api`). If no ticket is in the branch name, ask the user.

## Phase 3: Write the PR

### Title

```
type(scope): concise description
```

- Same format as commit messages
- Max 72 characters
- No ticket ID in the title

### Description

Follow flexible guidelines — adapt to PR size:

**Small PRs (1-3 files, simple change):**
```markdown
Brief explanation of why this change was made.

Jira: PDP-123
```

**Medium PRs:**
```markdown
## Summary
Why this change exists — the motivation, not a file list.

## Key decisions
- Chose X over Y because Z
- [any non-obvious trade-offs]

## Test plan
- [ ] Unit tests pass
- [ ] Manually verified X

Jira: PDP-123
```

**Large PRs (with a plan):**
When a `plan.md` exists, embed its content:
```markdown
## Summary
Why this change exists.

## Plan
[Embed the plan.md content here — goals, approach, key decisions, scope]

## Key changes
- [highlight the most important changes for reviewers]

## Test plan
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manually verified X

Jira: PDP-123
```

### Principles

- **Lead with WHY, not WHAT** — the diff shows what changed; explain the motivation
- **Never list every file changed** — highlight what matters for reviewers
- **Include test plan for non-trivial changes** — what was tested and how
- **Always link the Jira ticket** — `Jira: TICKET-123` at the bottom
- **One PR per concern** — don't mix unrelated changes
- **Mark WIP early** — use draft PRs for early feedback

## Phase 4: Create the PR

```bash
gh pr create --draft --base <default-branch> --title "type(scope): description" --body "$(cat <<'EOF'
[PR body here]
EOF
)"
```

Note: PRs are created as **draft** by default. Mark as ready for review when CI passes and the PR is complete.

## Phase 5: Verify

```bash
gh pr view --json number,url,title,isDraft
```

Confirm the PR was created and share the URL.

## Updating an Existing PR

Use `gh api` to update PRs (avoids known issues with `gh pr edit`):

```bash
# Get owner/repo
gh repo view --json owner,name --jq '"\(.owner.login)/\(.name)"'

# Update description
gh api -X PATCH repos/{owner}/{repo}/pulls/PR_NUMBER -f body="$(cat <<'EOF'
Updated description here
EOF
)"

# Update title
gh api -X PATCH repos/{owner}/{repo}/pulls/PR_NUMBER -f title='type(scope): new title'

# Update both
gh api -X PATCH repos/{owner}/{repo}/pulls/PR_NUMBER \
  -f title='type(scope): new title' \
  -f body='New description'
```

Re-read the current PR body first (`gh pr view --json body`) to preserve content you're not changing.
