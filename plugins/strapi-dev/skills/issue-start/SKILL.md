---
name: issue-start
description: Use when triaging a GitHub issue or Linear ticket end-to-end. Fetches issue context, runs triage analysis with a verdict checkpoint, drafts a response comment for human review, and optionally hands off to the bugfix workflow.
argument-hint: <issue-url-or-ticket-id>
---

# Issue Review Workflow

Triage a GitHub issue or Linear ticket: investigate, produce a verdict, draft a response comment for the user to post manually, and optionally start the fix workflow.

## Input

The user provides: $ARGUMENTS

Accepts one of:
- **GitHub issue URL** (e.g., `https://github.com/strapi/strapi/issues/456`)
- **Linear ticket ID** (e.g., `CMS-123`)
- **Plain text description**

If no input is provided, ask the user for an issue URL, ticket ID, or description.

## Issue Metadata Fetch

Fetch issue context upfront so `issue-analyze` does not need to re-fetch.

**If GitHub issue URL:**
```bash
gh issue view <number> --repo <owner/repo> --json title,body,labels,comments
```

**If Linear ticket ID:**
- Use `get_issue` from Linear MCP → title, description, priority, assignee, labels
- Use `list_comments` → discussion context

**If plain text:**
- Use the provided text directly as the issue summary
- Labels: none

**If fetch fails:**
- Report: "Could not fetch issue from [source]. Error: [message]"
- Ask user to paste the content manually and continue

Store: title, body, labels (array of strings), comments, source URL or ticket ID.

## Workflow

### Phase 1: Triage Analysis

Use the Skill tool: `skill: "strapi-dev:issue-analyze"`, passing the pre-fetched context (title, body, labels, comments, source URL or ticket ID) as arguments.

`issue-analyze` handles its own checkpoint menu. After the user selects a verdict, handle it below.

### Phase 2: Handle Verdict

`issue-analyze` handles its own checkpoint and returns a resolved verdict. Act on the verdict returned:

- **`confirmed`** → Phase 2a: Fix Handoff
- **`not a bug`** → Phase 2b: Draft Response (not a bug)
- **`needs more info`** → Phase 2b: Draft Response (needs more info)
- **`duplicate`** → Phase 2b: Draft Response (duplicate)
- **`feature request`** → Phase 2b: Draft Response (feature request)
- **`question`** → Phase 2b: Draft Response (question)
- **`docs gap`** → Phase 2b: Draft Response (docs gap)
- **`stop`** → Stop the workflow cleanly. No further output.

### Phase 2a: Fix Handoff

Present:
```
The issue is confirmed. Start the fix workflow?

  [1] Yes — hand off to bugfix workflow
  [2] No — stop here
```

If `[1]`, use the Skill tool: `skill: "strapi-dev:bugfix-start"`, passing the original ticket reference (issue URL or Linear ID).

If `[2]`, stop cleanly.

### Phase 2b: Draft Response

Draft a comment appropriate for the verdict. Do not post it — display it for the user to copy and post manually.

**Not a bug:**
```
Thanks for the report! After investigating, this appears to be expected behavior because [reason from triage analysis].

[One paragraph explaining the relevant code or design decision and why the current behavior is correct.]

If you believe this is still a problem, please share [specific additional context that would help re-evaluate — e.g., a minimal reproduction, your Strapi version, or the exact configuration you're using].
```

**Needs more info:**
```
Thanks for the report! To investigate further, could you please provide:

- [Specific piece of missing information — e.g., Strapi version and Node version]
- [Reproduction steps if not already included]
- [Error message or full stack trace if applicable]
- [Any other context specific to this issue based on triage findings]

Once we have this, we'll be able to dig in.
```

**Duplicate:**
```
Thanks for the report! This looks like a duplicate of #N — [title].

You can track progress on the fix there. If you have any additional context or a different reproduction case not covered in that issue, feel free to add it there.
```

**Feature request:**
```
Thanks for the suggestion! We've noted this as a feature request. 

[One paragraph summarizing the requested capability and its potential value, based on triage findings.]

We'll consider this for a future release. Feel free to add any additional context or use cases that would help us prioritize it.
```

**Question:**
```
Thanks for reaching out! [Answer or explanation based on triage analysis.]

If this doesn't fully address your question, please let us know and we'll dig in further.
```

**Docs gap:**
```
Thanks for pointing this out! You're right that the documentation [description of the gap from triage].

We'll update the docs to clarify this. If you'd like to contribute a fix, the docs live in [docs location if known].
```

Present the drafted comment, then show:
```
  [1] Looks good
  [2] Adjust wording
  [3] Skip — don't draft a comment
```

If `[1]`, stop cleanly. The workflow is complete.

Wait for the user's selection. If `[2]`, ask what to adjust, apply changes, and re-present with the same menu. If `[3]`, stop cleanly.

The comment is for the user to copy and post manually. Never call `gh issue comment`, `gh issue close`, or any GitHub write API.

## Error Handling

| Failure | Response |
|---------|----------|
| GitHub/Linear fetch fails | Report error, ask user to paste content manually, continue |
| `bugfix-start` unavailable | Report the error and stop |

## Key Principle

All GitHub writes go through human review. Claude never posts, closes, or labels issues.

Work from: /Users/ziyi/Work/claude-bugfix-workflow
