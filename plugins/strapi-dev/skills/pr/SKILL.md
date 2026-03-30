---
name: pr
description: Use after a bug fix is complete to generate a PR description following the repository's PR template and create the pull request via gh CLI.
---

# PR Creation

Generate a PR description from the repo's template and create the pull request.

## Input

**When run via orchestrator:** Has full context from all previous phases.

**When run standalone:** Must be on a branch with changes. Claude will:
1. Read the diff (`git diff main...HEAD`)
2. Read the PR template from `.github/PULL_REQUEST_TEMPLATE.md`
3. Ask the user for any missing context (ticket ID, bug description)

## Process

### Step 1: Read PR Template

Look for the PR template:
```bash
cat .github/PULL_REQUEST_TEMPLATE.md
```

If not found, check `.github/pull_request_template.md` (lowercase). If still not found, ask the user where it is.

### Step 2: Detect Base Branch

```bash
git remote show origin | grep "HEAD branch"
```

Use this as the `--base` for PR creation. If detection fails, ask the user.

### Step 3: Generate PR Description

For the Strapi PR template, fill sections as follows:

**"What does it do?"**
- Summary of the technical changes from the fix phase
- Keep concise — what was changed, not the full analysis

**"Why is it needed?"**
- Bug summary from the analyze phase
- Brief root cause explanation

**"How to test it?"**
- Reproduction steps from the analyze phase
- Test commands from the fix phase
- Expected behavior after the fix

**"Related issue(s)/PR(s)"**
- Linear ticket with magic word: `fix CMS-123` (auto-transitions ticket on merge)
- GitHub issue with magic word: `Fix #45` (auto-closes issue on merge)
- Related PRs if any were identified during analysis

### Step 4: Generate PR Title

Format: `fix(<area>): <concise description>`

Examples:
- `fix(relations): sync bidirectional relations on republish`
- `fix(query-params): handle keyword collision with user attributes`

Keep under 70 characters.

### Step 5: Present for Review

Show the complete PR (title + description) to the user.

**Checkpoint:** Present the PR title and description above, then show:

```
  [1] Create the PR
  [2] Edit the title
  [3] Edit the description
  [4] Stop here
```

Wait for the user's selection. If `[2]`, ask for the new title, update, and re-present with the same menu. If `[3]`, ask what to change in the description, apply, and re-present. If `[4]`, stop leaving the branch pushed but no PR created.

### Step 6: Create PR

```bash
git push -u origin HEAD
gh pr create --base <base-branch> --title "<title>" --body "<description>"
```

### Step 7: Strapi Docs Reminder

If the `strapi-dev:docs-update` phase identified Strapi documentation changes, remind the user:

"Don't forget the Strapi docs change. Here's what to submit to `strapi/documentation`:"
- Show the proposed file path and change

## Error Handling

- **`gh pr create` fails** (no remote, auth issues):
  - Attempt to push the branch first: `git push -u origin HEAD`
  - Retry PR creation
  - If still failing, output the PR title and description for the user to create manually

## Key Principle

The PR description follows the team's template exactly — no improvisation on format.
