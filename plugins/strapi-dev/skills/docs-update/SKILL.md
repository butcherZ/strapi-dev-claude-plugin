---
name: docs-update
description: Use after a bug fix or during a PR review to check if project documentation or Strapi documentation needs updating based on the code changes.
---

# Documentation Update Check

Check if the changes require updates to internal project docs or Strapi documentation.

## Input

**When run via bugfix orchestrator:** Has context of what code changed from `strapi-dev:bugfix-fix`.

**When run via review orchestrator:** Receives the PR diff content directly as context from `strapi-dev:review-start`.

**When run standalone:** Must be on a branch with changes. Claude will read the diff to understand what changed.

## Process

### Step 1: Get the Change Diff

```bash
git diff $(git merge-base main HEAD)...HEAD --name-only   # files changed
git diff $(git merge-base main HEAD)...HEAD               # full diff
```

If the base branch is not `main`, detect it:
```bash
git log --oneline --decorate | head -20  # find the base branch
```

Analyze the changed code for documentation-relevant changes:
- API endpoint behavior changes (request/response format, status codes, errors)
- Configuration option changes (new options, changed defaults, removed options)
- Error message changes (new errors, changed error text)
- CLI flag or command changes
- Environment variable changes
- Breaking changes in public interfaces

### Step 2: Check Internal Project Docs

Search the current repo for documentation that references the changed behavior:
- Use Grep to search `README.md`, `docs/`, `CONTRIBUTING.md`, and inline JSDoc/docstrings
- Check if any references are now outdated due to the fix

### Step 3: Check Strapi Documentation

Search the `strapi/documentation` repo for references to the changed behavior:

```bash
gh api search/code -f q="<search-term> repo:strapi/documentation" --jq '.items[] | "\(.path): \(.text_matches[0].fragment)"'
```

If the search scope is too broad, ask the user which doc pages might be affected.

### Step 4: Propose Updates or Skip

**If no docs are affected:**
- Report: "No documentation updates needed, moving on."
- Proceed to next phase without pausing.

**If internal docs are affected:**
- Present the specific changes needed with before/after
- **Checkpoint:** "These project docs reference the changed behavior. Approve the updates?"
- Apply changes and commit to the same branch

**If Strapi docs are affected:**
- Fetch the relevant file content from `strapi/documentation`:
  ```bash
  gh api repos/strapi/documentation/contents/<path> --jq '.content' | base64 -d
  ```
- Present the proposed change with before/after
- **Checkpoint:** "This Strapi doc page needs updating. Here's the proposed change for you to submit:"
- Output the change — do NOT auto-create a PR on the docs repo
- The user will submit the Strapi docs change manually

## Key Principle

Only touch docs that reference behavior that actually changed. No speculative doc improvements.
