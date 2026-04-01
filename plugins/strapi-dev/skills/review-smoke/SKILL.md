---
name: review-smoke
description: Use when checking out a PR branch and running automated tests for affected packages before review begins.
argument-hint: <pr-url-or-number>
---

# PR Smoke Test

Checkout a PR branch locally and run automated tests for the affected packages.

## Input

The user provides: $ARGUMENTS

Accepts one of:
- **PR URL** (e.g., `https://github.com/strapi/strapi/pull/123`)
- **PR number** (e.g., `123`)
- **Nothing** — uses PR context already provided by the orchestrator

## Process

### Step 1: Resolve Strapi Repo Path

1. Check Claude's memory for a previously saved Strapi repo path (search for "strapi repo path" or "strapi monorepo")
2. If found → use that path as the base for all operations
3. If not found → ask: "Where is your Strapi monorepo checked out? (e.g., `/path/to/strapi`)"
4. Save the user's answer to memory

### Step 2: Checkout the PR Branch

```bash
cd <strapi-repo-path>
gh pr checkout <pr-number>
```

If `gh pr checkout` fails, report the error clearly and ask the user to check out the branch manually, then confirm when done.

### Step 3: Identify Affected Packages

From the PR diff's changed file paths, determine which packages are touched:
- Files under `packages/core/content-manager/` → `@strapi/content-manager`
- Files under `packages/core/strapi/` → `@strapi/strapi`
- Files under `packages/utils/` → `@strapi/utils`
- Files under `packages/plugins/<name>/` → `@strapi/<name>`

List the affected packages before running tests.

### Step 4: Run Tests

For each affected package, run its tests from the Strapi repo root:

```bash
# Unit tests for affected packages (replace with actual package path)
yarn test:unit --testPathPattern="<package-relative-path>"

# API/integration tests if the changes touch controllers, routes, or services
yarn test:api --testPathPattern="<feature-area>"
```

If a test command fails to start (missing env vars, database not running), report the failure clearly and ask the user for setup instructions or permission to skip.

### Step 5: Report Results

Present results in this format:

```
## Smoke Test Results

**Branch:** <branch-name>
**Packages tested:** <comma-separated list>

| Package | Tests | Status |
|---------|-------|--------|
| @strapi/content-manager | 42 passed | PASS |
| @strapi/utils | 3 failed | FAIL |

### Failures
- <test name> (<file:line>)
  <error message>
```

## Checkpoint

```
  [1] All good — continue to code analysis
  [2] Investigate a test failure (specify which)
  [3] Skip remaining tests and continue anyway
  [4] Stop here
```

Wait for the user's selection. If `[2]`, investigate the specified failure, update the results table, and re-present with the same menu. If `[3]`, note in the summary that tests were skipped and proceed. If `[4]`, stop.

## Working Directory

Before starting, resolve the Strapi monorepo path:

1. Check Claude's memory for a previously saved Strapi repo path
2. If found → use that path
3. If not found → ask the user and save the answer to memory
