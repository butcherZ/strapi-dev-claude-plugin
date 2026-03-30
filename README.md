# strapi-dev — Claude Code Plugin

Strapi development workflows for Claude Code. Two entry points: **bug fixing** (from Linear ticket or GitHub issue through analysis, fix, documentation, and PR creation) and **PR review** (code analysis, breaking change detection, bug hunting, and docs impact check). Every checkpoint presents a numbered option menu — no free-form "yes/no" prompts.

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
| `strapi-dev:docs-update` | `/strapi-dev:docs-update` | Check if project or Strapi docs need updating (Router + Drafter pipeline) |
| `strapi-dev:obsidian` | `/strapi-dev:obsidian` | Write structured bug report to Obsidian vault |
| `strapi-dev:pr` | `/strapi-dev:pr` | Generate PR description from repo template |

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- [GitHub CLI](https://cli.github.com/) (`gh`) — for issue fetching, PR data, and PR creation
- **MCP servers** (optional but recommended):
  - [Linear MCP](https://github.com/anthropics/linear-mcp) — for fetching Linear tickets
  - [Obsidian MCP](https://github.com/MarkusMaal/obsidian-mcp) — for saving bug documentation
  - [Chrome DevTools MCP](https://github.com/anthropics/chrome-devtools-mcp) — for frontend bug reproduction
  - **GitHub MCP** — required for `docs-update` Router + Drafter pipeline (falls back to `gh api` search without it)

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
  → Analyze      → [1] proceed  [2] corrections  [3] dig deeper  [4] stop
  → Fix          → [1] Option A  [2] Option B  [3] Option C  [4] trade-offs  [5] stop
                 → [1] check docs  [2] show diff  [3] rerun tests  [4] back  [5] stop
  → Docs Update  → [1] approve  [2] adjust  [3] skip  [4] stop  (if internal docs affected)
                 → [1] approve targets  [2] remove  [3] adjust  [4] stop  (Router phase)
                 → [1] approve drafts  [2] regenerate  [3] edit  [4] stop  (Drafter phase)
  → Obsidian     → [1] save  [2] adjust content  [3] adjust tags  [4] skip  [5] stop
  → PR           → [1] create  [2] edit title  [3] edit description  [4] stop
```

### PR Review

```
/strapi-dev:review-start [PR-URL]
  → Fetch PR metadata
  → Code vs Description  → [1] continue  [2] corrections  [3] dig deeper  [4] stop
  → Breaking Changes     → [1] continue  [2] dig deeper  [3] missed something  [4] stop
  → Alternatives         → [1] continue  [2] explore alt  [3] disagree  [4] stop
  → Bug Hunting          → [1] continue  [2] another area  [3] dispute bug  [4] stop
  → Docs Impact          → (handled by docs-update internally)
  → Summary
```

Each phase can also be run independently.

## docs-update: Router + Drafter Pipeline

`docs-update` checks whether a code change requires documentation updates — in the current repo and in `strapi/documentation`. It runs a 6-step pipeline and presents numbered checkpoints at every decision point.

### Steps

**Step 1: Get the diff** — reads `git diff` from the branch (standalone) or uses the diff passed in from the orchestrator.

**Step 2: Detect doc-relevant changes** — analyzes the diff for things that affect documentation: API endpoint behavior, config options, error messages, CLI flags, env vars, breaking interface changes. Exits silently if nothing doc-relevant is found.

**Step 3: Internal project docs** — searches `README.md`, `docs/`, `CONTRIBUTING.md`, and inline docstrings for references to the changed behavior. If outdated references are found, presents a before/after diff with a checkpoint:
```
  [1] Approve — apply the updates
  [2] Adjust the proposed changes
  [3] Skip docs update
  [4] Stop here
```

**Step 4: Router** — fetches `agents/prompts/router.md`, `docusaurus/sidebars.js`, and `docusaurus/static/llms.txt` from `strapi/documentation` via GitHub MCP. Runs the Router prompt against the diff to produce a YAML report of target pages, actions (`update_section`, `add_section`, `create_page`, `add_row`, `add_link`, etc.), and priorities. Presents a checkpoint:
```
  [1] Approve — proceed to drafting
  [2] Remove a target
  [3] Adjust a target
  [4] Stop here
```
If the Router returns no targets, the skill exits: no Strapi doc changes needed.

**Step 5: Drafter** — fetches `agents/prompts/drafter.md` from `strapi/documentation` (only after the Router checkpoint passes). For each target, fetches the existing page content (skipped for `create_page` targets) and runs the Drafter prompt to produce style-compliant MDX. Selects mode automatically based on action type:
- `create_page` / `add_section` → **Compose**
- `update_section` / `update_text` / `add_row` → **Patch**
- `add_link` / `add_mention` / `add_tip` → **Micro-edit**

Presents all drafted changes with a checkpoint:
```
  [1] Approve — present for submission
  [2] Regenerate a specific change
  [3] Edit a change manually
  [4] Stop here
```

**Step 6: Final output** — presents each change as ready-to-paste MDX with file path and action type. No PR is created. No file in `strapi/documentation` is modified directly. The user submits manually.

### Fallback

If GitHub MCP is unavailable, the skill falls back to `gh api search/code` against `strapi/documentation` and presents a rough suggested edit. It flags that Router placement analysis and Drafter style compliance are not available without GitHub MCP.

## Uninstall

```bash
claude plugin uninstall strapi-dev@claude-bugfix-workflow
claude plugin marketplace remove claude-bugfix-workflow
```

## License

MIT
