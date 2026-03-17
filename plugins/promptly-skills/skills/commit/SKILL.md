---
name: commit
description: ALWAYS use this skill when committing code changes — never commit directly without it. Creates commits following Promptly Health conventions with proper conventional commit format. Trigger on any commit, git commit, save changes, or commit message task.
---

# Commit

ALWAYS follow this workflow when committing. Never run `git commit` without it.

## Phase 1: Review Changes

```bash
git status
git diff --staged
```

If nothing is staged, identify which files should be staged. Group related changes into logical commits — one concern per commit.

**Never commit:**
- `.env`, credentials, secrets, API keys
- Large binaries or generated files
- Unrelated changes (split into separate commits)

## Phase 2: Generate Commit Message

### Format

```
type(scope): concise description

[optional body — explain WHY, not WHAT]
```

### Types

| Type | When |
|------|------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code restructure, no behavior change |
| `perf` | Performance improvement |
| `test` | Adding or fixing tests |
| `build` | Build system or dependencies |
| `ci` | CI/CD configuration |
| `chore` | Maintenance, tooling, config |
| `revert` | Reverting a previous commit |

### Rules

- **Subject line:** imperative mood, lowercase, no period, max 72 chars
- **Scope:** optional but encouraged — use the module, package, or feature name (e.g., `api`, `auth`, `models`, `ui`)
- **Body:** wrap at 72 chars, explain motivation and context when non-obvious
- **No ticket IDs in commits** — tickets are linked in the PR
- **Breaking changes:** add `!` after type/scope and explain in body:
  ```
  feat(api)!: change patient response schema

  BREAKING CHANGE: removed deprecated `legacy_id` field from response.
  Clients should use `patient_id` instead.
  ```

### AI Attribution

AI-assisted commits MUST include:
```
Co-Authored-By: (the agent's name and attribution byline)
```

## Phase 3: Commit

```bash
git add <specific files>
git commit -m "$(cat <<'EOF'
type(scope): description

optional body
EOF
)"
```

Verify with `git log --oneline -3` after committing.

## Commit Sizing

Prefer small, focused commits:
- **One logical change per commit** — don't mix a bug fix with a refactor
- **Tests with implementation** — test changes go in the same commit as the code they test
- **If you can't summarize in 72 chars, the commit is too big** — split it

## Examples

```
feat(api): add patient enrollment endpoint
fix(auth): handle expired refresh tokens gracefully
refactor(models): extract shared validation logic
test(api): add coverage for rate limiting edge cases
ci: add Go lint step to PR workflow
docs: update API authentication guide
```
