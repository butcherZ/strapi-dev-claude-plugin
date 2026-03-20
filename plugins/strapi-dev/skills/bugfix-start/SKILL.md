---
name: bugfix-start
description: Use when starting a full bug-fixing workflow from a Linear ticket or GitHub issue. Orchestrates analysis, fix, docs update, Obsidian documentation, and PR creation with checkpoints between each phase.
argument-hint: <ticket-id-or-url>
---

# Bugfix Workflow

Full bug-fixing workflow from ticket to PR. Orchestrates 5 phases with user checkpoints between each.

## Input

The user provides: $ARGUMENTS

Provide one of:
- **Linear ticket ID** (e.g., `CMS-123`)
- **GitHub issue URL** (e.g., `https://github.com/strapi/strapi/issues/45`)
- **Plain text description** of the bug

If no input is provided, ask the user for a ticket ID, URL, or description.

## Branch Setup

Before starting the workflow:

1. Check current branch:
   - If already on a `fix/` branch → use it as-is
   - If on `main`, `develop`, or another base branch → create a new branch

2. Create branch:
   ```bash
   git checkout -b fix/<slug>
   ```
   - Slug is derived from the bug summary: lowercase, hyphenated, concise
   - Example: `fix/query-keyword-collision`, `fix/stale-api-response`
   - Do NOT include the ticket ID in the branch name

## Workflow

Execute each phase in order. Use the Skill tool to invoke each phase command. After each phase, pause at the checkpoint and wait for user approval before proceeding.

**IMPORTANT:** Maintain all context gathered from previous phases and pass it forward to each subsequent phase. Each phase builds on the information from the ones before it.

### Phase 1: Analyze

Use the Skill tool: `skill: "strapi-dev:bugfix-analyze", args: "<ticket-input>"`

This phase will:
- Fetch the ticket context
- Explore the codebase for affected areas
- Attempt to reproduce the bug
- Present a structured analysis

**Checkpoint:** "Here's my analysis. Does this match your understanding? Ready to move to finding the fix?"
- If user says yes → proceed to Phase 2
- If user has corrections → incorporate them, re-present analysis

### Phase 2: Fix

Use the Skill tool: `skill: "strapi-dev:bugfix-fix"`

This phase has **two internal checkpoints:**
1. After proposing fixes: "Which approach do you want to go with?"
2. After implementing: "Fix implemented and tests pass. Here's a summary of changes. Ready to check docs?"

### Phase 3: Docs Update

Use the Skill tool: `skill: "strapi-dev:docs-update"`

**If updates needed:** "These docs reference the changed behavior. Approve the updates?"
**If no updates needed:** "No documentation updates needed, moving on." (proceeds without pausing)

### Phase 4: Obsidian Note

Use the Skill tool: `skill: "strapi-dev:obsidian"`

**Checkpoint:** "Here's the Obsidian note. Want to adjust anything before I save it?"

### Phase 5: PR

Use the Skill tool: `skill: "strapi-dev:pr"`

**Checkpoint:** "Here's the PR description. Want to adjust anything before I create the PR?"

## Error Handling

When any phase fails:
1. **Report clearly** — what failed, why, what was attempted
2. **Ask the user** — options: retry, skip phase, or abort workflow
3. **Never silently continue** past a failure

Specific fallbacks:
- **Linear/GitHub MCP fails** → ask user to paste ticket content manually
- **Reproduction fails** → proceed with static analysis, flag reduced confidence
- **Tests fail after fix** → stop, do not proceed to docs/PR until tests pass
- **Obsidian MCP unavailable** → output note content in terminal for manual save
- **`gh pr create` fails** → output PR description for manual creation

## Key Principles

- **Collaborative autonomy** — Claude runs each phase but checks in after every major step
- **Each phase is self-contained** — can be re-run independently if needed
- **Checkpoints are mandatory** — never skip user approval between phases
