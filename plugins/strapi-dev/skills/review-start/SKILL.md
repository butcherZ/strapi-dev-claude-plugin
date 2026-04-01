---
name: review-start
description: Use when reviewing a pull request. Orchestrates code analysis, breaking change detection, alternative approaches, bug hunting, and docs impact check with checkpoints between each phase.
argument-hint: <pr-url>
---

# PR Review Workflow

Review a pull request through 5 phases with user checkpoints between each. Strapi monorepo focused.

## Input

The user provides: $ARGUMENTS

Accepts one of:
- **GitHub PR URL** (e.g., `https://github.com/strapi/strapi/pull/123`)
- **Nothing** — auto-detects the current branch's PR

If no input is provided and no PR is found for the current branch, ask the user for a PR URL or number.

## PR Metadata Fetch

Fetch the PR context needed for all phases:

**If PR URL provided:**
```bash
gh pr view <URL> --json title,body,number,baseRefName,headRefName,files
gh pr diff <URL>
```

**If auto-detecting from current branch:**
```bash
gh pr view --json title,body,number,baseRefName,headRefName,files
gh pr diff
```

Extract and store:
- PR title and description (body)
- PR number
- Base and head branch names
- List of changed files
- Full diff content

If `gh` fails, ask the user to paste the PR description and provide the branch name manually.

## Workflow

Execute each phase in order. Use the Skill tool to invoke each phase. After each phase, pause at the checkpoint and wait for user input before proceeding.

**IMPORTANT:** Maintain all context gathered from previous phases and pass it forward to each subsequent phase. Each phase builds on the information from the ones before it.

### Phase 0: Smoke Test

Use the Skill tool: `skill: "strapi-dev:review-smoke"`

Pass the PR number and any available diff context.

**Checkpoint:** Present the smoke test results above, then show:

```
  [1] All good — continue to code analysis
  [2] Investigate a test failure (specify which)
  [3] Skip and continue anyway
  [4] Stop here
```

Wait for the user's selection. If `[2]`, ask review-smoke to investigate the specified failure. If `[3]`, note skipped tests in the running context and continue. If `[4]`, stop the review.

### Phase 1: Code vs Description

Use the Skill tool: `skill: "strapi-dev:review-analyze", args: "pass:1 <PR-metadata>"`

Pass the PR title, description, changed files list, and diff.

**Checkpoint:** Present the code vs description analysis above, then show:

```
  [1] Looks right — check for breaking changes
  [2] I have corrections (describe them)
  [3] Dig deeper into a specific file
  [4] Stop here
```

Wait for the user's selection. If `[2]`, incorporate corrections and re-present analysis with the same menu. If `[3]`, investigate the specified file and re-present. If `[4]`, stop the review.

### Phase 2: Breaking Changes

Use the Skill tool: `skill: "strapi-dev:review-analyze", args: "pass:2 <PR-metadata + Phase 1 findings>"`

**Checkpoint:** Present the breaking change analysis above, then show:

```
  [1] Continue to alternatives
  [2] Dig deeper into a specific area
  [3] I think you missed something (describe it)
  [4] Stop here
```

Wait for the user's selection. If `[2]`, investigate the specified area and re-present with the same menu. If `[3]`, investigate what they described and re-present. If `[4]`, stop the review.

### Phase 3: Alternative Approaches

Use the Skill tool: `skill: "strapi-dev:review-analyze", args: "pass:3 <PR-metadata + Phase 1-2 findings>"`

**Checkpoint:** Present the alternative approaches above, then show:

```
  [1] Continue to bug hunting
  [2] Explore an alternative further (specify which)
  [3] I disagree with the recommendation (describe why)
  [4] Stop here
```

Wait for the user's selection. If `[2]`, explore the specified alternative in more depth and re-present. If `[3]`, discuss the disagreement and revise the recommendation if warranted, then re-present. If `[4]`, stop the review.

### Phase 4: Manual Testing

Use the Skill tool: `skill: "strapi-dev:review-manual-test"`

Pass the PR number and the full PR body so the skill can parse testing instructions without re-fetching.

**Checkpoint:** Present the manual test results above, then show:

```
  [1] Looks good — continue to bug hunting
  [2] Re-run a specific step (specify)
  [3] A step failed — investigate (specify)
  [4] Stop here
```

Wait for the user's selection. If `[2]` or `[3]`, pass the detail to review-manual-test and re-present with the same menu. If `[4]`, stop, leaving all Bruno files in place.

### Phase 5: Bug Hunting

Use the Skill tool: `skill: "strapi-dev:review-bugs"`

Pass all accumulated context: PR metadata, analysis findings, breaking change assessment.

**Checkpoint:** Present the bug report above, then show:

```
  [1] Continue to docs check
  [2] Investigate another area for bugs (specify)
  [3] Dispute a reported bug (specify which)
  [4] Stop here
```

Wait for the user's selection. If `[2]`, investigate the specified area and update the bug report, then re-present with the same menu. If `[3]`, re-examine the disputed bug and remove it from the report if unconfirmed, then re-present. If `[4]`, stop the review.

### Phase 6: Documentation Impact

Use the Skill tool: `skill: "strapi-dev:docs-update"`

Pass the PR diff content as context so the skill can analyze without needing a local branch diff.

`docs-update` handles its own numbered checkpoint menus internally. Wait for the skill to complete before presenting the final summary. No separate Phase 5 checkpoint is needed here.

## Summary

After all phases complete, present a consolidated summary:

```
## PR Review Summary: #<number> — <title>

### Smoke Test: <passed / failed / skipped>
<one-line summary>

### Code Match: <match / partial match / mismatch>
<one-line summary>

### Breaking Changes: <none / low risk / medium risk / high risk>
<one-line summary>

### Alternatives: <current approach is best / consider alternative X>
<one-line summary>

### Manual Tests: <passed / partial / skipped — no instructions found>
<one-line summary>

### Bugs Found: <N bugs found / no bugs found>
<one-line summary per bug>

### Docs Impact: <no updates needed / updates needed>
<one-line summary>
```

## Error Handling

When any phase fails:
1. **Report clearly** — what failed, why, what was attempted
2. **Ask the user** — options: retry, skip phase, or abort review
3. **Never silently continue** past a failure

Specific fallbacks:
- **`gh pr view` fails** → ask user to paste PR description manually
- **`gh pr diff` fails** → ask user to provide the diff or ensure the branch is checked out locally
- **Test infrastructure unavailable** (Phase 4) → report suspected bugs without proof, flag reduced confidence

## Key Principles

- **Read-only** — no commits, no pushes, no PRs created
- **Exception:** Phase 4 may create test files and apply temporary fixes for verification, all left unstaged
- **Checkpoints are mandatory** — never skip user approval between phases
- **Strapi-specific** — export tracing, package boundary checks, user-facing API analysis

## Working Directory

Before starting, resolve the Strapi monorepo path:

1. **Check Claude's memory** for a previously saved Strapi repo path (search for "strapi repo path" or "strapi monorepo")
2. **If found** → use that path as the base for all file operations
3. **If not found** → ask the user: "Where is your Strapi monorepo checked out? (e.g., `/path/to/strapi`)"
4. **Save the user's answer to Claude's memory** so they are never asked again
