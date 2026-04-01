---
name: issue-analyze
description: Use when triaging a GitHub issue or Linear ticket. Fetches context, explores affected code, attempts reproduction, and presents a triage verdict with a checkpoint.
argument-hint: <issue-url-or-ticket-id>
---

# Issue Triage

Investigate a GitHub issue or Linear ticket to produce a triage verdict: confirmed bug, not a bug, needs more info, or duplicate.

## Input

The user provides: $ARGUMENTS

Accepts one of:
- **GitHub issue URL** (e.g., `https://github.com/strapi/strapi/issues/456`)
- **Linear ticket ID** (e.g., `CMS-123`)
- **Plain text description**

**When run via orchestrator:** Receives pre-fetched context (title, body, labels, comments) as arguments — skip Step 1.
**When run standalone:** Fetch context in Step 1.

## Process

### Step 1: Fetch Context (standalone only)

**If GitHub issue URL:**
```bash
gh issue view <number> --repo <owner/repo> --json title,body,labels,comments
```
Extract: title, body, labels (array of label name strings), comments.

**If Linear ticket ID:**
- Use `get_issue` from Linear MCP → title, description, priority, assignee, labels
- Use `list_comments` → discussion context

**If plain text:**
- Use the provided text as the issue summary
- Labels: none

**If fetch fails:**
- Report: "Could not fetch issue from [source]. Error: [message]"
- Ask user to paste the content manually and continue

### Step 2: Classify Issue Type

Determine one of: `bug report` / `feature request` / `question` / `docs gap`

Use:
- Issue title and body language ("not working", "error", "crash" → bug; "add support for", "would be nice" → feature request; "how do I" → question; "docs say X but" → docs gap)
- Labels if present (e.g., `type: bug`, `type: enhancement`)

This classification controls which subsequent steps run.

### Step 3: Check for Duplicates

```bash
gh search issues "<3-5 keywords from issue title>" --repo <owner/repo> --state open --limit 5
```

Use the repo extracted from the GitHub issue URL. For Linear tickets or plain text input, default to `strapi/strapi`.

Report any close matches with their issue numbers and titles. If the search fails or returns no results, note it and continue.

### Step 4: Explore Affected Code

Based on the issue description, trace relevant code areas:
- Error messages or stack traces → `Grep` for the exact string
- File paths referenced in the issue → `Read` those files
- Feature area described → `Glob` for relevant directories (e.g., `packages/core/content-manager/src/**` for content manager issues)

For broad or uncertain searches, use an Explore subagent.

Output a list of relevant files/functions with a one-line explanation of each one's relationship to the issue.

Skip this step only if the issue is clearly a `question` or `docs gap` with no code reference.

### Step 5: Attempt Reproduction (bug reports only)

Skip this step for `feature request`, `question`, and `docs gap` types.

**For frontend bugs** (UI issues, visual regressions, console errors):
- Chrome DevTools MCP: `navigate_page` to the affected URL
- `take_screenshot` to capture current state
- `list_console_messages` to check for JS errors
- `list_network_requests` to check for failed API calls

**For backend bugs** (API errors, wrong data, crashes):
- Write a minimal curl sequence or test script that triggers the reported behavior
- Run it and capture the output

**If reproduction fails:**
- Document what was attempted and why it failed
- Flag: "Could not reproduce locally. Analysis is based on code reading only. Confidence may be lower."

### Step 6: Present Triage Verdict

```
## Issue Triage: <id> — <title>

### Type: <bug | feature request | question | docs gap>
### Verdict: <confirmed | not a bug | needs more info | duplicate of #N>
### Severity: <critical | major | minor | N/A>
### Affected area: <file / component / feature>
### Duplicate check: <no duplicates found | possible duplicate: #N — <title>>
### Labels: <label-1>, <label-2>

### Summary
<2–3 sentences: what's broken or being requested, evidence, conclusion>
```

**Severity guide:**
- `critical` — data loss, security vulnerability, or complete feature outage in production
- `major` — feature partially broken, workaround exists but is painful
- `minor` — cosmetic issue, edge case, or low-impact behavior
- `N/A` — feature request, question, or docs gap
- `unknown` — when the verdict is `needs more info` or `duplicate`, use this (reproduction insufficient to assess, or defers to the original issue's severity)

**Checkpoint:**
```
  [1] Confirmed
  [2] Not a bug
  [3] Needs more info
  [4] Duplicate
  [5] Dig deeper into a specific area
  [6] I have corrections (describe them)
  [7] Stop here
```

Wait for the user's selection. If `[5]`, investigate the specified area and re-present the triage verdict with the same menu. If `[6]`, incorporate the corrections, re-classify if needed, and re-present the triage verdict with the same menu. If `[7]`, stop with no changes made.

## Key Principle

Read-only — no code changes, no GitHub writes.
