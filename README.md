# Promptly Health Skills

Agent skills for Promptly Health engineers, following the [Agent Skills](https://agentskills.io) open format.

## Installation

### Claude Code

```bash
claude plugin marketplace add promptlylabs/promptly-skills
claude plugin install promptly-skills@promptly-skills
```

Restart Claude Code after installation. Skills activate automatically when relevant.

**Update:**

```bash
claude plugin marketplace update
claude plugin update promptly-skills@promptly-skills
```

Or run `/plugin` to open the plugin manager.

### Skills Package (skills.sh)

For agents supporting the [skills.sh](https://skills.sh) ecosystem:

```bash
npx skills add promptlylabs/promptly-skills
```

Works with Claude Code, Cursor, Cline, GitHub Copilot, and other compatible agents.

## Available Skills

| Skill | Description |
|-------|-------------|
| [acli](plugins/promptly-skills/skills/acli/SKILL.md) | Atlassian CLI (acli) reference for Jira Cloud. |
| [agents-md](plugins/promptly-skills/skills/agents-md/SKILL.md) | Create and maintain AGENTS.md files with research-backed best practices. |
| [better-auth-best-practices](plugins/promptly-skills/skills/better-auth-best-practices/SKILL.md) | Better Auth authentication patterns and best practices. |
| [claude-settings-audit](plugins/promptly-skills/skills/claude-settings-audit/SKILL.md) | Analyze a repository to generate recommended Claude Code settings.json permissions. |
| [code-review](plugins/promptly-skills/skills/code-review/SKILL.md) | Perform code reviews covering security, performance, testing, and design. |
| [code-simplifier](plugins/promptly-skills/skills/code-simplifier/SKILL.md) | Simplify and refine code for clarity, consistency, and maintainability. |
| [commit](plugins/promptly-skills/skills/commit/SKILL.md) | Create commits following Promptly Health conventional commit conventions. |
| [create-branch](plugins/promptly-skills/skills/create-branch/SKILL.md) | Create git branches following Promptly Health naming conventions. |
| [django-access-review](plugins/promptly-skills/skills/django-access-review/SKILL.md) | Django access control and IDOR security review. |
| [django-perf-review](plugins/promptly-skills/skills/django-perf-review/SKILL.md) | Django performance code review for N+1 queries, ORM issues, and more. |
| [doc-coauthoring](plugins/promptly-skills/skills/doc-coauthoring/SKILL.md) | Structured workflow for co-authoring documentation and specs. |
| [docker](plugins/promptly-skills/skills/docker/SKILL.md) | Docker best practices for containerization. |
| [drizzle-orm](plugins/promptly-skills/skills/drizzle-orm/SKILL.md) | Drizzle ORM patterns and best practices. |
| [find-bugs](plugins/promptly-skills/skills/find-bugs/SKILL.md) | Find bugs, security vulnerabilities, and code quality issues in branch changes. |
| [gh-review-requests](plugins/promptly-skills/skills/gh-review-requests/SKILL.md) | Fetch unread GitHub notifications for PRs needing team review. |
| [gha-security-review](plugins/promptly-skills/skills/gha-security-review/SKILL.md) | GitHub Actions security review for workflow exploitation vulnerabilities. |
| [golang-patterns](plugins/promptly-skills/skills/golang-patterns/SKILL.md) | Go coding patterns and conventions. |
| [golang-testing](plugins/promptly-skills/skills/golang-testing/SKILL.md) | Go testing patterns and conventions. |
| [iterate-pr](plugins/promptly-skills/skills/iterate-pr/SKILL.md) | Iterate on a PR until CI passes and review feedback is addressed. |
| [pr-writer](plugins/promptly-skills/skills/pr-writer/SKILL.md) | Create and update PRs following Promptly Health conventions with flexible guidelines. |
| [react-hook-form](plugins/promptly-skills/skills/react-hook-form/SKILL.md) | React Hook Form patterns and best practices. |
| [security-review](plugins/promptly-skills/skills/security-review/SKILL.md) | Security code review for vulnerabilities (OWASP). |
| [sentry](plugins/promptly-skills/skills/sentry/SKILL.md) | Fetch and analyze Sentry issues, events, transactions, and logs. |
| [shadcn-ui](plugins/promptly-skills/skills/shadcn-ui/SKILL.md) | shadcn/ui component patterns. |
| [skill-scanner](plugins/promptly-skills/skills/skill-scanner/SKILL.md) | Scan agent skills for security issues. |
| [skill-writer](plugins/promptly-skills/skills/skill-writer/SKILL.md) | Create, synthesize, and iteratively improve agent skills. |
| [tailwind-theme-builder](plugins/promptly-skills/skills/tailwind-theme-builder/SKILL.md) | Tailwind CSS theming and customization. |
| [vercel-composition-patterns](plugins/promptly-skills/skills/vercel-composition-patterns/SKILL.md) | React composition patterns that scale. |
| [vercel-react-best-practices](plugins/promptly-skills/skills/vercel-react-best-practices/SKILL.md) | React and Next.js performance optimization guidelines. |
| [zod](plugins/promptly-skills/skills/zod/SKILL.md) | Zod schema validation patterns. |

## Available Subagents

| Subagent | Description |
|----------|-------------|
| [code-simplifier](plugins/promptly-skills/agents/code-simplifier.md) | Simplifies and refines code for clarity, consistency, and maintainability while preserving all functionality |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for details on testing, PR workflow, and adding new skills.

### Repository Structure

```
promptly-skills/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace manifest
├── plugins/
│   └── promptly-skills/
│       ├── .claude-plugin/
│       │   └── plugin.json   # Plugin manifest
│       ├── agents/
│       │   └── code-simplifier.md
│       └── skills/
│           ├── code-review/
│           │   └── SKILL.md
│           └── ...
├── AGENTS.md                 # Agent-facing documentation
├── CLAUDE.md                 # Symlink to AGENTS.md
└── README.md                 # This file
```

### Where Skills Belong

| Scope | Location | Example |
|-------|----------|---------|
| **Global** - Used across Promptly Health | `promptly-skills` plugin | `code-review`, `security-review`, `iterate-pr` |
| **Domain-specific** - Used by a team | Dedicated plugin in this repo | `infra-skills`, `data-skills` |
| **Repo-specific** - Only relevant to one repo | The repository itself (`.claude/skills/`) | Project-specific workflows |

### Creating New Skills

Skills follow the [Agent Skills specification](https://agentskills.io/specification). Each skill requires a `SKILL.md` file with YAML frontmatter.

#### Skill Template

Create a new directory under `plugins/promptly-skills/skills/`:

```yaml
---
name: my-skill
description: A clear description of what this skill does and when to use it. Include keywords that help agents identify when this skill is relevant.
---

# My Skill Name

## Instructions

Step-by-step guidance for the agent.
```

#### Naming Conventions

- **name**: 1-64 characters, lowercase alphanumeric with hyphens only
- **description**: Up to 1024 characters, include trigger keywords
- Keep SKILL.md under 500 lines; split longer content into reference files

## References

- [Agent Skills Specification](https://agentskills.io/specification)

## Acknowledgements

This project was heavily inspired by [getsentry/skills](https://github.com/getsentry/skills).

## License

Apache-2.0
