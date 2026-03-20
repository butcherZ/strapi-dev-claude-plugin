---
name: bugfix-analyze
description: Use when starting a bug investigation from a Linear ticket, GitHub issue, or plain text description. Fetches ticket context, explores affected code, and attempts reproduction.
argument-hint: <ticket-id-or-url>
---

# Bug Analysis

Investigate a bug report to understand what's broken, reproduce it, and identify affected code.

## Input

The user provides: $ARGUMENTS

Accepts one of:
- **Linear ticket ID** (e.g., `CMS-123`) — fetched via Linear MCP
- **GitHub issue URL** (e.g., `https://github.com/strapi/strapi/issues/45`) — fetched via `gh issue view`
- **Plain text description** — used directly as the bug summary

If no input is provided, ask the user for a ticket ID, URL, or description.

## Process

### Step 1: Fetch Ticket Context

**If Linear ticket ID:**
- Use `get_issue` from Linear MCP to pull: title, description, priority, assignee, labels
- Use `list_comments` to pull discussion/context
- Present structured summary to user

**If GitHub issue URL:**
- Run `gh issue view <number> --repo <owner/repo>` to pull: title, body, labels, comments
- Present structured summary to user

**If plain text:**
- Use the provided text as the bug summary
- Note: ticket-dependent fields (ticket ID, source URL) will be N/A in later phases

**If Linear/GitHub MCP fails:**
- Report the failure: "Could not fetch ticket from [source]. Error: [message]"
- Ask user to paste the ticket content manually
- Continue with pasted content

### Step 2: Codebase Exploration

Based on the ticket description, identify affected areas by looking for:
- Error messages or stack traces mentioned in the ticket
- File paths or function names referenced
- Feature areas described (e.g., "authentication", "API routes", "content types")

Use Glob, Grep, and Read tools to trace relevant code paths. For broad searches, use an Explore subagent.

Output:
- List of affected files and functions
- Component dependency map (what calls what)

### Step 3: Reproduction

**For frontend bugs** (UI issues, console errors, visual regressions):
- Use Chrome DevTools MCP: `navigate_page` to the affected URL
- `take_screenshot` to capture current state
- `list_console_messages` to check for errors
- `list_network_requests` to check for failed API calls
- `evaluate_script` to inspect DOM state if needed

**For backend bugs** (API errors, data issues, crashes):
- Write a minimal test or script that triggers the bug
- Run it and capture the error output
- Check logs if available

**For full-stack bugs:**
- Combine both approaches above

**If reproduction fails** (environment-specific, intermittent, requires specific data):
- Document what was attempted and why it failed
- Proceed with static analysis
- Flag: "Bug could not be reproduced locally. Analysis is based on code reading and ticket description. Confidence in root cause may be lower."

### Step 4: Present Analysis

Present a structured summary:

```
## Bug Analysis: <ticket-id> — <title>

### Summary
What is broken and who is affected.

### Reproduction
- **Status:** Confirmed / Not confirmed / Partially confirmed
- **Steps taken:** [what you did]
- **Evidence:** [screenshots, error output, test results]

### Affected Code
- `path/to/file.ts:123` — [what this code does]
- `path/to/other.ts:45` — [relationship to the bug]

### Initial Hypothesis
Based on [evidence], the likely root cause is [hypothesis].
This will be verified in the fix phase.
```

**Checkpoint:** "Here's my analysis. Does this match your understanding? Ready to move to finding the fix?"

## Key Principle

This phase is **read-only** — no code changes, only investigation.
