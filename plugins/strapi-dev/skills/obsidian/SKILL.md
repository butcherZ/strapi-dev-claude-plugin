---
name: obsidian
description: Use after fixing a bug, completing a feature, or finishing any Strapi work to create a structured Obsidian note. Saves to the right subfolder under the user-configured Strapi base path with auto-generated tags.
argument-hint: <subfolder-path>
---

# Obsidian Note Documentation

Create a structured note in Obsidian documenting Strapi work — bug fixes, features, architecture decisions, and more.

## Input

**When run via orchestrator:** Has full context from all previous phases (ticket, analysis, fix, docs updates).

**When run standalone:** Ask the user to describe:
1. What was done?
2. What was the root cause or motivation?
3. What ticket/issue is this for? (optional)

Claude will prompt for any missing details needed to fill the template.

## Configuration

- **Filename format:** `<ticket-id> <short description>.md` (e.g., `CMS-148 Fix Query Keyword Collision with User-Defined Attribute Names.md`)
- **Created via:** Obsidian MCP `write_note` tool

### Vault Path Resolution

Before generating the note, resolve the save location:

1. **If a subfolder path was passed as an argument** (`$ARGUMENTS`) → use it as the full save path
2. **Otherwise, check Claude's memory** for a previously saved Strapi base path (search for "obsidian strapi base path")
3. **If found in memory** → use it as the base, then determine the subfolder (see below)
4. **If not found** → ask the user: "Where is your Strapi notes folder in Obsidian? (e.g., `notes/work/strapi`)"
5. **Save the user's answer to Claude's memory** as the Strapi base path so they are never asked again

### Subfolder Selection

Based on the context, select or create the appropriate subfolder under the Strapi base path:

| Context | Subfolder |
|---------|-----------|
| Bug fix, production issue | `bug-fixes/` |
| Feature development | `feature/<feature-name>/` |
| Architecture, design decision, ADR | `architecture/` |
| Dev setup, testing, developer guide | `setup-and-reference/` |
| MCP integration | `mcp/` |
| OpenAPI, route validation, Zod | `openapi-validation/` |
| Doesn't fit any existing folder | Create a new descriptive subfolder |

If the context is ambiguous, infer from the ticket labels, title, and content. If still unclear, ask the user which subfolder to use.

## Process

### Step 1: Generate Note Content

Build the note following this template:

**Frontmatter** (pass as structured data to `write_note`):
```json
{
  "title": "<ticket-id> <short description>",
  "type": "note",
  "permalink": "<base-path-slug>/<subfolder-slug>/<slug>",
  "tags": ["strapi", "<type>", "<area>", "<component>"],
  "date": "YYYY-MM-DD",
  "ticket": "<ticket-id>",
  "source": "<linear or github URL>",
  "severity": "<critical|major|minor|N/A>",
  "status": "resolved",
  "labels": ["<label-1>", "<label-2>"]
}
```

The `permalink` is derived from the full resolved save path: convert to lowercase, replace spaces with hyphens, strip leading/trailing slashes, then append `/<slug>`.

For example: `notes/work/strapi/bug-fixes/` → `notes/work/strapi/bug-fixes/<slug>`

**Body for bug fixes:**
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

**Body for features/architecture:**
```markdown
# <Title>

## Context
What problem this solves and why it was needed.

## Design
Key decisions made and why.

## Implementation
What was built and how it works.

## Files Changed
| File | Change |
|------|--------|

## References
Links to tickets, PRs, or related notes.
```

Use the bug fix body when the context is a bug fix; use the feature/architecture body otherwise.

### Step 2: Auto-tag

Infer tags from the context:

**Type tags:** `bug`, `feature`, `architecture`, `setup`, `mcp`

**Area tags** — based on what part of the system was affected:
- `relations`, `draft-and-publish`, `api`, `admin`, `content-type-builder`, `authentication`, `database`, `middleware`, `query-engine`, `upload`, `i18n`, etc.

**Component tags** — based on Strapi architecture:
- `v5`, `document-service`, `entity-service`, `content-manager`, `rest-api`, `graphql`, etc.

### Step 3: Classify Severity (bug fixes only)

Determine severity based on:
- **critical** — data loss, security vulnerability, or complete feature outage in production
- **major** — feature partially broken, workaround exists but is painful
- **minor** — cosmetic issue, edge case, or low-impact behavior

Use `N/A` for features, architecture notes, and setup guides.

If the Linear ticket has a priority field, map it: Urgent/High → critical, Medium → major, Low/None → minor. Otherwise, infer from bug impact and ask for confirmation.

### Step 4: Present for Review

Show the complete note to the user before saving.

**Checkpoint:** Present the complete note above, then show:

```
  [1] Save as-is
  [2] Adjust content
  [3] Adjust tags or severity
  [4] Change subfolder
  [5] Skip — don't save the note, continue workflow
  [6] Stop here
```

Wait for the user's selection. If `[2]`, ask what to adjust, apply the changes, and re-present with the same menu. If `[3]`, ask which tags or severity to change, apply, and re-present. If `[4]`, ask which subfolder to use instead, update, and re-present. If `[5]`, skip saving and proceed to the next phase. If `[6]`, stop the workflow leaving no changes.

### Step 5: Save to Obsidian

Use the Obsidian MCP `write_note` tool:
- Path: `<resolved-save-path>/<filename>.md`
- Pass frontmatter as structured data
- Pass body as content string

**If Obsidian MCP is unavailable:**
- Output the complete note (frontmatter + body) in the terminal
- Tell the user: "Obsidian MCP is not available. Here's the note content — you can save it manually."

## Key Principle

Write in past tense — this is a record of what happened, not a plan.
