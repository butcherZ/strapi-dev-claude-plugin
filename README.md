# strapi-dev — Claude Code Plugin

Strapi development workflows for Claude Code. Two entry points: **bug fixing** (from Linear ticket or GitHub issue through analysis, fix, documentation, and PR creation) and **PR review** (code analysis, breaking change detection, bug hunting, and docs impact check). Both workflows have checkpoints at every step.

## Skills

### Bug Fix Workflow

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `strapi-dev:bugfix-start` | `/strapi-dev:bugfix-start <ticket>` | Orchestrator — runs all bugfix phases with checkpoints |
| `strapi-dev:bugfix-analyze` | `/strapi-dev:bugfix-analyze <ticket>` | Fetch ticket, explore code, reproduce the bug |
| `strapi-dev:bugfix-fix` | `/strapi-dev:bugfix-fix` | Find root cause, propose fixes, implement with TDD |

### PR Review Workflow

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `strapi-dev:review-start` | `/strapi-dev:review-start [PR-URL]` | Orchestrator — runs all review phases with checkpoints |
| `strapi-dev:review-analyze` | `/strapi-dev:review-analyze [PR-URL]` | Code vs description, breaking changes, alternatives |
| `strapi-dev:review-bugs` | `/strapi-dev:review-bugs` | Bug hunting with proof tests |

### Shared Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `strapi-dev:docs-update` | `/strapi-dev:docs-update` | Check if project or Strapi docs need updating |
| `strapi-dev:obsidian` | `/strapi-dev:obsidian` | Write structured bug report to Obsidian vault |
| `strapi-dev:pr` | `/strapi-dev:pr` | Generate PR description from repo template |

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [GitHub CLI](https://cli.github.com/) (`gh`) — for issue fetching, PR data, and PR creation
- **MCP servers** (optional but recommended):
  - [Linear MCP](https://github.com/anthropics/linear-mcp) — for fetching Linear tickets
  - [Obsidian MCP](https://github.com/MarkusMaal/obsidian-mcp) — for saving bug documentation
  - [Chrome DevTools MCP](https://github.com/anthropics/chrome-devtools-mcp) — for frontend bug reproduction

## Installation

```bash
# Add as a marketplace
claude plugin marketplace add https://github.com/butcherZ/claude-bugfix-workflow

# Install the plugin
claude plugin install strapi-dev@claude-bugfix-workflow
```

Restart Claude Code to load the plugin.

## Quick Start

```
# Bug fix — full workflow from a Linear ticket
/strapi-dev:bugfix-start CMS-123

# Bug fix — full workflow from a GitHub issue
/strapi-dev:bugfix-start https://github.com/strapi/strapi/issues/45

# PR review — from a PR URL
/strapi-dev:review-start https://github.com/strapi/strapi/pull/123

# PR review — auto-detect from current branch
/strapi-dev:review-start

# Run individual phases
/strapi-dev:bugfix-analyze CMS-123
/strapi-dev:bugfix-fix
/strapi-dev:review-analyze https://github.com/strapi/strapi/pull/123
/strapi-dev:review-bugs
/strapi-dev:docs-update
/strapi-dev:obsidian
/strapi-dev:pr
```

## Workflows

### Bug Fix

```
/strapi-dev:bugfix-start CMS-123
  → Create branch fix/<slug>
  → Analyze      → checkpoint
  → Fix          → checkpoint (propose) → checkpoint (implement)
  → Docs Update  → checkpoint (if needed)
  → Obsidian     → checkpoint
  → PR           → checkpoint
```

### PR Review

```
/strapi-dev:review-start [PR-URL]
  → Fetch PR metadata
  → Code vs Description  → checkpoint
  → Breaking Changes     → checkpoint
  → Alternatives         → checkpoint
  → Bug Hunting          → checkpoint
  → Docs Impact          → checkpoint
  → Summary
```

Each phase can also be run independently.

## Uninstall

```bash
claude plugin uninstall strapi-dev@claude-bugfix-workflow
claude plugin marketplace remove claude-bugfix-workflow
```

## License

MIT
