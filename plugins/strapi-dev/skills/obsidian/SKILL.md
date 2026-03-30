---
name: obsidian
description: Use after fixing a bug to create a structured Obsidian note documenting the bug, root cause, fix, and lessons learned. Saves to the Strapi bug-fix folder with auto-generated tags.
---

# Obsidian Bug Documentation

Create a structured bug report note in Obsidian for future reference.

## Input

**When run via orchestrator:** Has full context from all previous phases (ticket, analysis, fix, docs updates).

**When run standalone:** Ask the user to describe:
1. What was the bug?
2. What was the root cause?
3. What was the fix?
4. What ticket/issue is this for? (optional)

Claude will prompt for any missing details needed to fill the template.

## Configuration

- **Vault folder:** Configurable per user (see below)
- **Filename format:** `<ticket-id> <short description>.md` (e.g., `CMS-148 Fix Query Keyword Collision with User-Defined Attribute Names.md`)
- **Created via:** Obsidian MCP `write_note` tool

### Vault Path Resolution

Before generating the note, determine where to save it:

1. **Check Claude's memory** for a previously saved vault path preference (search for memories about "obsidian vault path" or "bug notes folder")
2. **If found** → use the remembered path
3. **If not found** → ask the user: "Where in your Obsidian vault should I save bug notes? (e.g., `2 - Full Notes/Strapi/bug-fix/`)"
4. **Save the user's answer to Claude's memory** so they are never asked again in future sessions

## Process

### Step 1: Generate Note Content

Build the note following this template:

**Frontmatter** (pass as structured data to `write_note`):
```json
{
  "title": "<ticket-id> <short description>",
  "type": "note",
  "permalink": "full-notes/strapi/<slug>",
  "tags": ["strapi", "bug", "<area>", "<component>"],
  "date": "YYYY-MM-DD",
  "ticket": "<ticket-id>",
  "source": "<linear or github URL>",
  "severity": "<critical|major|minor>",
  "status": "resolved"
}
```

**Body:**
```markdown
# <Ticket-ID>: <Bug Title>

## Context
What was reported, who's affected, link to ticket.

## Reproduction Steps
Step-by-step instructions to trigger the bug.

## Root Cause Analysis
Why it happened — architecture context, code traces, file/line references.

## Fix
What was changed, why this approach, code snippets of before/after.

## Files Changed
| File | Change |
|------|--------|

## Tests Added
What test cases were written.

## Verification
Manual test steps + automated test commands.
```

### Step 2: Auto-tag

Infer tags from the context:

**Area tags** — based on what part of the system was affected:
- `relations`, `draft-and-publish`, `api`, `admin`, `content-type-builder`, `authentication`, `database`, `middleware`, `query-engine`, `upload`, `i18n`, etc.

**Component tags** — based on Strapi architecture:
- `v5`, `document-service`, `entity-service`, `content-manager`, `rest-api`, `graphql`, etc.

### Step 3: Classify Severity

Determine severity based on:
- **critical** — data loss, security vulnerability, or complete feature outage in production
- **major** — feature partially broken, workaround exists but is painful
- **minor** — cosmetic issue, edge case, or low-impact behavior

If the Linear ticket has a priority field, map it: Urgent/High → critical, Medium → major, Low/None → minor. Otherwise, infer from bug impact and ask for confirmation.

### Step 4: Present for Review

Show the complete note to the user before saving.

**Checkpoint:** Present the complete note above, then show:

```
  [1] Save as-is
  [2] Adjust content
  [3] Adjust tags or severity
  [4] Skip — don't save the note, continue workflow
  [5] Stop here
```

Wait for the user's selection. If `[2]`, ask what to adjust, apply the changes, and re-present the note with the same menu. If `[3]`, ask which tags or severity to change, apply, and re-present. If `[4]`, skip saving and proceed to the next phase. If `[5]`, stop the workflow leaving no changes.

### Step 5: Save to Obsidian

Use the Obsidian MCP `write_note` tool:
- Path: `<vault-folder>/<filename>.md` (using the resolved vault path)
- Pass frontmatter as structured data
- Pass body as content string

**If Obsidian MCP is unavailable:**
- Output the complete note (frontmatter + body) in the terminal
- Tell the user: "Obsidian MCP is not available. Here's the note content — you can save it manually."

## Key Principle

Write in past tense — this is a record of what happened, not a plan.
