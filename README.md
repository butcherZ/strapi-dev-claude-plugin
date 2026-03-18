# bugfix — Claude Code Plugin

A modular bug-fixing workflow for Claude Code. Guides you from Linear ticket or GitHub issue through analysis, fix, documentation, and PR creation — with checkpoints at every step.

## Skills

| Skill | Trigger | Purpose |
|-------|---------|---------|
| `bugfix:start` | `/bugfix:start <ticket>` | Orchestrator — runs all phases with checkpoints |
| `bugfix:analyze` | `/bugfix:analyze <ticket>` | Fetch ticket, explore code, reproduce the bug |
| `bugfix:fix` | `/bugfix:fix` | Find root cause, propose fixes, implement with TDD |
| `bugfix:docs-update` | `/bugfix:docs-update` | Check if project or Strapi docs need updating |
| `bugfix:obsidian` | `/bugfix:obsidian` | Write structured bug report to Obsidian vault |
| `bugfix:pr` | `/bugfix:pr` | Generate PR description from repo template |

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [GitHub CLI](https://cli.github.com/) (`gh`) — for issue fetching and PR creation
- **MCP servers** (optional but recommended):
  - [Linear MCP](https://github.com/anthropics/linear-mcp) — for fetching Linear tickets
  - [Obsidian MCP](https://github.com/MarkusMaal/obsidian-mcp) — for saving bug documentation
  - [Chrome DevTools MCP](https://github.com/anthropics/chrome-devtools-mcp) — for frontend bug reproduction

## Installation

```bash
# Add as a marketplace
claude plugin marketplace add https://github.com/butcherZ/claude-bugfix-workflow

# Install the plugin
claude plugin install bugfix@claude-bugfix-workflow
```

Restart Claude Code to load the plugin.

## Quick Start

```
# Full workflow from a Linear ticket
/bugfix:start CMS-123

# Full workflow from a GitHub issue
/bugfix:start https://github.com/strapi/strapi/issues/45

# Run individual phases
/bugfix:analyze CMS-123
/bugfix:fix
/bugfix:obsidian
/bugfix:pr
```

## Workflow

```
/bugfix:start CMS-123
  → Create branch fix/<slug>
  → Analyze      → checkpoint
  → Fix          → checkpoint (propose) → checkpoint (implement)
  → Docs Update  → checkpoint (if needed)
  → Obsidian     → checkpoint
  → PR           → checkpoint
```

Each phase can also be run independently.

## Uninstall

```bash
claude plugin uninstall bugfix@claude-bugfix-workflow
claude plugin marketplace remove claude-bugfix-workflow
```

## License

MIT
