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

### Phase 1: Code vs Description

Use the Skill tool: `skill: "strapi-dev:review-analyze", args: "pass:1 <PR-metadata>"`

Pass the PR title, description, changed files list, and diff.

**Checkpoint:** "Does this match your read of the PR? Any corrections before I check for breaking changes?"
- If user has corrections → incorporate, re-present
- If user says yes → proceed to Phase 2

### Phase 2: Breaking Changes

Use the Skill tool: `skill: "strapi-dev:review-analyze", args: "pass:2 <PR-metadata + Phase 1 findings>"`

**Checkpoint:** "Any areas you want me to dig deeper into before I look at alternatives?"
- If user wants deeper analysis → investigate specific areas, re-present
- If user says yes → proceed to Phase 3

### Phase 3: Alternative Approaches

Use the Skill tool: `skill: "strapi-dev:review-analyze", args: "pass:3 <PR-metadata + Phase 1-2 findings>"`

**Checkpoint:** "Thoughts on these alternatives? Ready to hunt for bugs?"
- If user wants to discuss → explore the alternatives further
- If user says yes → proceed to Phase 4

### Phase 4: Bug Hunting

Use the Skill tool: `skill: "strapi-dev:review-bugs"`

Pass all accumulated context: PR metadata, analysis findings, breaking change assessment.

**Checkpoint:** "Here are the bugs found (or none found). Want to proceed to check docs impact?"
- If user wants to investigate further → continue bug hunting
- If user says yes → proceed to Phase 5

### Phase 5: Documentation Impact

Use the Skill tool: `skill: "strapi-dev:docs-update"`

Pass the PR diff content as context so the skill can analyze without needing a local branch diff.

**If updates needed:** "These docs reference changed behavior. Here's what would need updating."
**If no updates needed:** "No documentation updates needed."

## Summary

After all phases complete, present a consolidated summary:

```
## PR Review Summary: #<number> — <title>

### Code Match: <match / partial match / mismatch>
<one-line summary>

### Breaking Changes: <none / low risk / medium risk / high risk>
<one-line summary>

### Alternatives: <current approach is best / consider alternative X>
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

Work from: /Users/ziyi/Work/claude-bugfix-workflow
