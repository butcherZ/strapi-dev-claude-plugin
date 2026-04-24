# strapi-dev — Claude Code Plugin

Strapi development workflows for Claude Code: **bug fixing**, **PR review**, and **issue triage**. All workflows run end-to-end with numbered option menus at every checkpoint — no free-form prompts.

## Installation

```bash
claude plugin marketplace add https://github.com/butcherZ/strapi-dev-claude-plugin
claude plugin install strapi-dev@strapi-dev-claude-plugin
```

Restart Claude Code to load the plugin.

## Prerequisites

- [GitHub CLI](https://cli.github.com/) (`gh`) — issue fetching, PR data, PR creation
- **MCP servers** (optional but recommended):
  - [Linear MCP](https://github.com/anthropics/linear-mcp) — fetch Linear tickets
  - [Obsidian MCP](https://github.com/MarkusMaal/obsidian-mcp) — save bug notes to Obsidian
  - [Chrome DevTools MCP](https://github.com/anthropics/chrome-devtools-mcp) — reproduce frontend bugs
  - **GitHub MCP** — required for Strapi docs update (Router + Drafter pipeline); falls back to basic search without it
  - **Playwright MCP** — executes Admin UI steps during PR manual testing; falls back to a printed checklist without it
- [Bruno](https://www.usebruno.com/) — API client used during PR manual testing; collections are written to your Bruno folder automatically

---

## Bug Fix Workflow

Start from a ticket and go all the way to a merged PR.

```
/strapi-dev:bugfix-start CMS-123
/strapi-dev:bugfix-start https://github.com/strapi/strapi/issues/45
```

Phases run in order with a checkpoint between each:

1. **Analyze** — fetches the ticket, explores the affected code, attempts to reproduce the bug, presents a structured analysis
2. **Fix** — identifies the root cause, proposes 2–3 fix options for you to choose from, implements the chosen fix with a TDD approach
3. **Docs Update** — checks if internal or Strapi docs need updating based on what changed
4. **Obsidian** — writes a structured bug report note to your Obsidian vault
5. **PR** — generates a PR description from the repo template and creates the PR

### Running phases individually

```
/strapi-dev:bugfix-analyze CMS-123   # investigate only
/strapi-dev:bugfix-fix               # fix only (needs analysis context)
```

---

## PR Review Workflow

Analyze a pull request across eight dimensions.

```
/strapi-dev:review-start https://github.com/strapi/strapi/pull/123
/strapi-dev:review-start   # auto-detects PR from current branch
```

Phases run in order with a checkpoint between each:

0. **Smoke Test** — checks out the PR branch locally and runs automated tests for affected packages; flags failures before any analysis begins
1. **Code vs Description** — reads every changed file and checks it against the PR description; flags missing or undocumented changes
2. **Breaking Changes** — traces exports and package boundaries to assess user-facing impact
3. **Alternative Approaches** — proposes 2–3 alternatives to the PR's approach with a recommendation
4. **Bug Hunting** — looks for bugs in the changed code, proves each one with a failing test, applies a minimal fix, leaves all files unstaged for you to review
5. **Fix Proposals** — synthesizes all findings into ready-to-post GitHub review comments with friendly, reasoning-forward tone
6. **Manual Testing** *(optional)* — follows the PR's "How to test" instructions: executes Admin UI steps via Playwright MCP and creates a Bruno API collection for HTTP request steps; can be skipped if static analysis was sufficient
7. **Docs Impact** — checks if the changes require documentation updates

### Running phases individually

```
/strapi-dev:review-smoke                                             # checkout + smoke test only
/strapi-dev:review-analyze https://github.com/strapi/strapi/pull/123
/strapi-dev:review-bugs                                              # bug hunting only
/strapi-dev:review-manual-test                                       # follow PR test instructions only
```

---

## Issue Triage Workflow

Investigate a GitHub issue or Linear ticket, produce a triage verdict, and optionally hand off to the bug fix workflow.

```
/strapi-dev:issue-start CMS-123
/strapi-dev:issue-start https://github.com/strapi/strapi/issues/456
```

Phases run in order with a checkpoint between each:

1. **Triage Analysis** — fetches the issue, classifies its type (bug / feature request / question / docs gap), checks for duplicates, explores affected code, and attempts reproduction for bugs
2. **Verdict & Response** — presents a triage verdict (confirmed / not a bug / needs more info / duplicate); drafts a response comment for you to copy and post manually
3. **Fix Handoff** (confirmed bugs only) — optionally hands off to the bug fix workflow

All response comments are displayed in the terminal for you to review and post — nothing is ever auto-posted to GitHub.

### Running the triage phase individually

```
/strapi-dev:issue-analyze CMS-123   # triage only, with its own checkpoint
```

---

## Shared Skills

These run as part of the workflows above but can also be invoked standalone.

### `/strapi-dev:docs-update`

Checks whether the current branch's changes require documentation updates — both in the current repo and in `strapi/documentation`.

- Detects doc-relevant changes in the diff (API behavior, config options, error messages, etc.)
- Searches the current repo for outdated references and proposes fixes
- Uses the Router agent from `strapi/documentation` to identify which Strapi doc pages need updating
- Uses the Drafter agent to write style-compliant MDX content for each target page
- Presents the final output as ready-to-paste content — never auto-creates a PR on `strapi/documentation`

Requires GitHub MCP for the Router + Drafter pipeline. Falls back to a basic `gh api` search if GitHub MCP is unavailable.

### `/strapi-dev:obsidian`

Generates a structured bug report note and saves it to your Obsidian vault. Covers: context, reproduction steps, root cause, fix, files changed, tests added, and verification steps. Includes labels from the original issue. Asks for your vault path the first time and remembers it. You can also pass the path directly: `/strapi-dev:obsidian Notes/Strapi/bug-fix/`.

### `/strapi-dev:pr`

Reads the repo's PR template, fills it from the fix context (what changed, why, how to test), and creates the PR via `gh pr create`. Presents the title and description for review before creating.

---

## Update

```bash
claude plugin update strapi-dev@strapi-dev-claude-plugin
```

Restart Claude Code to apply the update.

## Uninstall

```bash
claude plugin uninstall strapi-dev@strapi-dev-claude-plugin
claude plugin marketplace remove strapi-dev-claude-plugin
```

## License

MIT
