# Promptly Health Skills

Agent skills for Promptly Health engineers, following the [Agent Skills](https://agentskills.io) open format.

## Installation

### Claude Code (plugin)

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

### Pi

```bash
pi install git:github.com/promptlylabs/promptly-skills
```

Skills are available immediately. Use `/skill:name` or let the agent load them automatically.

**Update:**

```bash
pi update
```

### Skills Package (skills.sh)

For agents supporting the [skills.sh](https://skills.sh) ecosystem (Claude Code, Cursor, Cline, GitHub Copilot, and others):

**Install all skills:**

```bash
npx skills add promptlylabs/promptly-skills
```

**Install specific skills:**

```bash
npx skills add promptlylabs/promptly-skills -s code-review
npx skills add promptlylabs/promptly-skills -s code-review -s security-review
```

**Update:**

```bash
npx skills update
```

## Available Skills

| Skill | Description | Source |
|-------|-------------|--------|
| [acli](plugins/promptly-skills/skills/acli/SKILL.md) | Atlassian CLI (acli) reference for Jira Cloud. | [ivanrvpereira/.agents](https://github.com/ivanrvpereira/.agents) |
| [agents-md](plugins/promptly-skills/skills/agents-md/SKILL.md) | Create and maintain AGENTS.md files with research-backed best practices. | [ivanrvpereira/.agents](https://github.com/ivanrvpereira/.agents) |
| [better-auth-best-practices](plugins/promptly-skills/skills/better-auth-best-practices/SKILL.md) | Better Auth authentication patterns and best practices. | [better-auth/skills](https://github.com/better-auth/skills) |
| [claude-settings-audit](plugins/promptly-skills/skills/claude-settings-audit/SKILL.md) | Analyze a repository to generate recommended Claude Code settings.json permissions. | [getsentry/skills](https://github.com/getsentry/skills) |
| [code-review](plugins/promptly-skills/skills/code-review/SKILL.md) | Perform code reviews covering security, performance, testing, and design. | [getsentry/skills](https://github.com/getsentry/skills) |
| [code-simplifier](plugins/promptly-skills/skills/code-simplifier/SKILL.md) | Simplify and refine code for clarity, consistency, and maintainability. | [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official) |
| [commit](plugins/promptly-skills/skills/commit/SKILL.md) | Create commits following Promptly Health conventional commit conventions. | [getsentry/skills](https://github.com/getsentry/skills) |
| [create-branch](plugins/promptly-skills/skills/create-branch/SKILL.md) | Create git branches following Promptly Health naming conventions. | [getsentry/skills](https://github.com/getsentry/skills) |
| [django-access-review](plugins/promptly-skills/skills/django-access-review/SKILL.md) | Django access control and IDOR security review. | [getsentry/skills](https://github.com/getsentry/skills) |
| [django-perf-review](plugins/promptly-skills/skills/django-perf-review/SKILL.md) | Django performance code review for N+1 queries, ORM issues, and more. | [getsentry/skills](https://github.com/getsentry/skills) |
| [doc-coauthoring](plugins/promptly-skills/skills/doc-coauthoring/SKILL.md) | Structured workflow for co-authoring documentation and specs. | [getsentry/skills](https://github.com/getsentry/skills) |
| [docker](plugins/promptly-skills/skills/docker/SKILL.md) | Docker best practices for containerization. | [bobmatnyc/claude-mpm-skills](https://github.com/bobmatnyc/claude-mpm-skills) |
| [drizzle-orm](plugins/promptly-skills/skills/drizzle-orm/SKILL.md) | Drizzle ORM patterns and best practices. | [bobmatnyc/claude-mpm-skills](https://github.com/bobmatnyc/claude-mpm-skills) |
| [find-bugs](plugins/promptly-skills/skills/find-bugs/SKILL.md) | Find bugs, security vulnerabilities, and code quality issues in branch changes. | [getsentry/skills](https://github.com/getsentry/skills) |
| [frontend-design](plugins/promptly-skills/skills/frontend-design/SKILL.md) | Create distinctive, production-grade frontend interfaces with high design quality. | [anthropics/skills](https://github.com/anthropics/skills) |
| [gh-review-requests](plugins/promptly-skills/skills/gh-review-requests/SKILL.md) | Fetch unread GitHub notifications for PRs needing team review. | [getsentry/skills](https://github.com/getsentry/skills) |
| [gha-security-review](plugins/promptly-skills/skills/gha-security-review/SKILL.md) | GitHub Actions security review for workflow exploitation vulnerabilities. | [getsentry/skills](https://github.com/getsentry/skills) |
| [golang-patterns](plugins/promptly-skills/skills/golang-patterns/SKILL.md) | Go coding patterns and conventions. | [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) |
| [golang-testing](plugins/promptly-skills/skills/golang-testing/SKILL.md) | Go testing patterns and conventions. | [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) |
| [iterate-pr](plugins/promptly-skills/skills/iterate-pr/SKILL.md) | Iterate on a PR until CI passes and review feedback is addressed. | [getsentry/skills](https://github.com/getsentry/skills) |
| [k8s-crd-design-review](plugins/promptly-skills/skills/k8s-crd-design-review/SKILL.md) | Kubernetes CRD design review: schema, conditions, SSA, versioning, compatibility. | [configbutler/skills](https://github.com/configbutler/skills) |
| [kubebuilder-api-design](plugins/promptly-skills/skills/kubebuilder-api-design/SKILL.md) | Kubebuilder Go API type design, CRD markers, validation, and generation workflow. | [configbutler/skills](https://github.com/configbutler/skills) |
| [pr-writer](plugins/promptly-skills/skills/pr-writer/SKILL.md) | Create and update PRs following Promptly Health conventions with flexible guidelines. | [getsentry/skills](https://github.com/getsentry/skills) |
| [react-hook-form](plugins/promptly-skills/skills/react-hook-form/SKILL.md) | React Hook Form patterns and best practices. | [pproenca/dot-skills](https://github.com/pproenca/dot-skills) |
| [security-review](plugins/promptly-skills/skills/security-review/SKILL.md) | Security code review for vulnerabilities (OWASP). | [getsentry/skills](https://github.com/getsentry/skills) |
| [sentry](plugins/promptly-skills/skills/sentry/SKILL.md) | Fetch and analyze Sentry issues, events, transactions, and logs. | [mitsuhiko/agent-stuff](https://github.com/mitsuhiko/agent-stuff) |
| [shadcn-ui](plugins/promptly-skills/skills/shadcn-ui/SKILL.md) | shadcn/ui component patterns. | [jezweb/claude-skills](https://github.com/jezweb/claude-skills) |
| [skill-scanner](plugins/promptly-skills/skills/skill-scanner/SKILL.md) | Scan agent skills for security issues. | [getsentry/skills](https://github.com/getsentry/skills) |
| [skill-writer](plugins/promptly-skills/skills/skill-writer/SKILL.md) | Create, synthesize, and iteratively improve agent skills. | [getsentry/skills](https://github.com/getsentry/skills) |
| [tailwind-theme-builder](plugins/promptly-skills/skills/tailwind-theme-builder/SKILL.md) | Tailwind CSS theming and customization. | [jezweb/claude-skills](https://github.com/jezweb/claude-skills) |
| [vercel-composition-patterns](plugins/promptly-skills/skills/vercel-composition-patterns/SKILL.md) | React composition patterns that scale. | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) |
| [vercel-react-best-practices](plugins/promptly-skills/skills/vercel-react-best-practices/SKILL.md) | React and Next.js performance optimization guidelines. | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) |
| [zod](plugins/promptly-skills/skills/zod/SKILL.md) | Zod schema validation patterns. | [bobmatnyc/claude-mpm-skills](https://github.com/bobmatnyc/claude-mpm-skills) |

## Available Subagents

| Subagent | Description |
|----------|-------------|
| [code-simplifier](plugins/promptly-skills/agents/code-simplifier.md) | Simplifies and refines code for clarity, consistency, and maintainability while preserving all functionality |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for details on testing, PR workflow, and adding new skills.

### Local Development

```bash
git clone git@github.com:promptlylabs/promptly-skills.git ~/promptly-skills
claude plugin marketplace add ~/promptly-skills
claude plugin install promptly-skills
```

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

Skills should live in the appropriate location based on their scope:

| Scope | Location | Example |
|-------|----------|---------|
| **Global** - Used across Promptly Health | `promptly-skills` plugin | `code-review`, `security-review`, `iterate-pr` |
| **Domain-specific** - Used by a team | Dedicated plugin in this repo | `infra-skills`, `data-skills` |
| **Repo-specific** - Only relevant to one repo | The repository itself (`.claude/skills/`) | Project-specific workflows |

When deciding where to place a skill:
- If most Promptly Health engineers would benefit, add it to `promptly-skills`
- If only a specific team needs it, create or use a domain-specific plugin
- If it only makes sense in one repo, keep it in that repo

#### Marketplace Structure

This repository is a Claude Code **marketplace** — a collection of plugins that can be installed independently. The marketplace manifest (`.claude-plugin/marketplace.json`) lists all available plugins:

```json
{
  "plugins": [
    { "name": "promptly-skills", "source": "./plugins/promptly-skills" },
    { "name": "infra-skills", "source": "./plugins/infra-skills" }
  ]
}
```

Each plugin lives in its own directory under `plugins/` with its own `plugin.json` manifest. Users can install individual plugins:

```bash
# Install just the global skills
claude plugin install promptly-skills@promptly-skills

# Install domain-specific skills
claude plugin install infra-skills@promptly-skills
```

To add a new domain-specific plugin:

1. Create `plugins/<plugin-name>/.claude-plugin/plugin.json`
2. Add skills under `plugins/<plugin-name>/skills/`
3. Register the plugin in `.claude-plugin/marketplace.json`

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

## Examples

Concrete examples showing expected input/output.

## Guidelines

- Specific rules to follow
- Edge cases to handle
```

For repeatable `skill-writer` evaluation prompts, see [plugins/promptly-skills/skills/skill-writer/EVAL.md](plugins/promptly-skills/skills/skill-writer/EVAL.md).

#### Naming Conventions

- **name**: 1-64 characters, lowercase alphanumeric with hyphens only
- **description**: Up to 1024 characters, include trigger keywords
- Keep SKILL.md under 500 lines; split longer content into reference files

#### Optional Fields

| Field | Description |
|-------|-------------|
| `license` | License name or path to license file |
| `compatibility` | Environment requirements (max 500 chars) |
| `allowed-tools` | Comma-separated list of tools the skill can use |
| `metadata` | Arbitrary key-value pairs for additional properties |

```yaml
---
name: my-skill
description: What this skill does
license: Apache-2.0
allowed-tools: Read, Grep, Glob
---
```

### Vendoring Skills

We vendor (copy) skills from external sources into this repository rather than depending on them at runtime. This approach:

- **Ensures consistency** — Everyone on the team uses the same version of each skill
- **Enables customization** — We can adapt skills to Promptly Health conventions
- **Improves reliability** — No external dependencies that could change or disappear

#### Attribution

When vendoring a skill or agent from an external source, retain proper attribution:

1. **Add a comment** at the top of the file (after the frontmatter `---`) referencing the original source:
   ```markdown
   <!--
   Based on [Original Name] by [Author/Org]:
   https://github.com/example/original-source
   -->
   ```

2. **Include a LICENSE file** in the skill directory if the original has specific licensing requirements:
   ```
   plugins/promptly-skills/skills/vendored-skill/
   ├── SKILL.md
   └── LICENSE      # Original license text
   ```

3. **Record the source** in the Available Skills table's Source column so provenance is always visible.

#### Example: code-simplifier

The `code-simplifier` agent is vendored from [Anthropic's official plugins](https://github.com/anthropics/claude-plugins-official). See the attribution comment at the top of the agent file.

## References

- [Agent Skills Specification](https://agentskills.io/specification)

## Acknowledgements

This project was heavily inspired by [getsentry/skills](https://github.com/getsentry/skills).

## License

Apache-2.0
