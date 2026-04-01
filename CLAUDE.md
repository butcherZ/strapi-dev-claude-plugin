# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

A Claude Code plugin (`strapi-dev`) providing three end-to-end development workflows for Strapi contributors:

1. **Bug Fix Workflow** — `bugfix-start` → `bugfix-analyze` → `bugfix-fix` → `docs-update` → `obsidian` → `pr`
2. **PR Review Workflow** — `review-start` → `review-smoke` → `review-analyze` → `review-manual-test` → `review-bugs` → `docs-update`
3. **Issue Triage Workflow** — `issue-start` → `issue-analyze` → optional handoff to `bugfix-start`

The plugin has no executable code. It is entirely implemented as Claude Code skills (Markdown files with YAML frontmatter). There are no tests, no build steps, and no npm dependencies to install.

## Repository Structure

```
plugins/strapi-dev/
  .claude-plugin/plugin.json   # Plugin manifest (name, version, author)
  package.json                 # Minimal package metadata (no deps)
  skills/                      # One subdirectory per skill, each with SKILL.md
    bugfix-start/
    bugfix-analyze/
    bugfix-fix/
    docs-update/
    issue-analyze/
    issue-start/
    obsidian/
    pr/
    review-start/
    review-analyze/
    review-smoke/
    review-manual-test/
    review-bugs/
docs/superpowers/
  plans/                       # Implementation plans for past/future features
  specs/                       # Design specifications
```

## Skill File Format

Each skill is a `SKILL.md` with YAML frontmatter:

```markdown
---
name: <skill-id>
description: <one-liner shown in skill picker>
argument-hint: <optional — shown as placeholder in UI>
---

# Title

## Input
## Process
## Key Principles
```

The `name` field must match the skill's directory name. The `description` is what users see when browsing skills.

## Architecture Patterns

**Two-tier orchestration:** `bugfix-start`, `review-start`, and `issue-start` are orchestrators that invoke sub-skills sequentially via the Skill tool. Sub-skills can also be invoked standalone.

**Checkpoint system:** Every phase ends with a numbered menu (Continue / Change / Skip / Stop). The skill must wait for user input before advancing. This is mandatory — never skip checkpoints.

**Context passing between phases:** The orchestrator accumulates output from each phase and passes it forward. Sub-skills receive prior phase summaries as context.

**Fallback model:** When optional MCP servers are unavailable (Linear, GitHub, Chrome DevTools, Obsidian), skills fall back gracefully — ask user to paste content manually, proceed with reduced confidence, or output results to terminal.

**TDD in bugfix-fix:** Write a failing test first, then fix, then verify tests pass. Never apply a fix without a corresponding test.

**Bug proof requirement in review-bugs:** Never report a potential bug without a failing test proving it exists. Leave all test files and fixes unstaged.

**Issue triage read-only principle:** `issue-analyze` never writes to GitHub. Response comments are drafted and displayed in the terminal for the user to copy and post manually.

**Working directory resolution:** Skills that operate on the Strapi monorepo (`review-start`, `review-bugs`, `docs-update`, `issue-start`) resolve the repo path from Claude's memory — asking the user once and saving it. No hardcoded paths.

**Obsidian vault path:** Configurable via memory or passed directly as a skill argument. Labels from GitHub/Linear issues are stored in the note frontmatter.

## Versioning

Plugin version lives in three places — keep them all in sync when bumping:
- `plugins/strapi-dev/.claude-plugin/plugin.json` → `"version"`
- `plugins/strapi-dev/package.json` → `"version"`
- `.claude-plugin/marketplace.json` → `"plugins[0].version"`

## Key Tool Integrations

Skills rely on these external tools being available in the user's environment:
- `gh` (GitHub CLI) — required for PR creation and fetching PR data
- Linear MCP server — optional, for fetching Linear ticket context
- GitHub MCP server — optional, used by `docs-update` for Router + Drafter pipeline
- Obsidian MCP server — optional, used by `obsidian` skill to save notes
- Chrome DevTools MCP — optional, used by `bugfix-analyze` for frontend reproduction
- Playwright MCP — optional, used by `review-manual-test` for Admin UI test execution
- Bruno API client — optional, used by `review-manual-test` to create API test collections; path stored in memory as `bruno collection path`
