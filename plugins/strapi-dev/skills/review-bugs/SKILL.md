---
name: review-bugs
description: Use when hunting for bugs in a PR's changed code. Identifies potential bugs, proves them with failing tests, then verifies fixes. All changes left unstaged for user review.
---

# PR Bug Hunting

Identify potential bugs in a PR's code changes, prove them with failing tests, and verify fixes.

## Input

**When run via orchestrator:** Receives accumulated context from `strapi-dev:review-start` — PR metadata, diff, analysis findings from earlier phases.

**When run standalone:** Ask the user:
1. Which PR or branch to review? (PR URL or ensure branch is checked out)
2. Any specific areas of concern?

If standalone, fetch the PR diff:
```bash
gh pr diff [URL]
```

## Process

### Step 1: Identify Potential Bugs

Read every changed file in the PR diff. Look for:

- **Edge cases not handled:** null values, empty arrays, missing fields, undefined properties
- **Off-by-one errors:** wrong loop bounds, incorrect array indexing
- **Logic errors:** wrong operators (`&&` vs `||`), inverted conditions, missing `return`
- **Async issues:** missing `await`, unhandled promise rejections, race conditions
- **Type issues:** incorrect type narrowing, type assertions hiding real problems
- **Error handling:** swallowed errors, wrong error types thrown, missing error cases
- **Strapi-specific:**
  - Missing sanitization of user input
  - Wrong service method calls (e.g., `findOne` vs `findFirst`)
  - Lifecycle hook ordering issues
  - Missing permission checks
  - Incorrect content-type handling (single type vs collection type)
  - Draft/publish state not considered

Build a list of suspected bugs with:
- Location (file:line)
- What might go wrong
- Under what conditions

### Step 2: Prove Each Bug

For each suspected bug:

1. **Find the nearest test file:**
   - Look in the same directory for `__tests__/`, `tests/`, or `*.test.*` files
   - Look for API test files in `tests/api/` matching the feature area
   - Match the existing test convention (jest, mocha, etc.)

2. **Write a failing test:**
   - The test should exercise the exact code path where the bug exists
   - Use the same test patterns as nearby existing tests
   - The test name should describe the bug: `it('should handle null values in ...')`

3. **Run the test:**
   ```bash
   # Use the project's test runner — check package.json scripts
   # For Strapi API tests, typically:
   yarn test:api -- --testPathPattern="<test-file>"
   # Or for unit tests:
   yarn test:unit -- --testPathPattern="<test-file>"
   ```

4. **Evaluate the result:**
   - **Test FAILS** → bug is confirmed. Continue to Step 3.
   - **Test PASSES** → bug was not real. Delete the test file, remove from the report. Do not mention it.

### Step 3: Fix and Verify Each Confirmed Bug

For each confirmed bug:

1. **Apply the minimal fix** — change only what's necessary to make the test pass
2. **Re-run the exact same test** — confirm it now passes
3. **Record:**
   - The bug description
   - The test that proves it
   - The fix that resolves it
   - Before/after test output

**IMPORTANT:** Do NOT commit any changes. Leave all files (tests + fixes) unstaged. The user decides what to keep.

### Step 4: Present Report

**Output format when bugs are found:**

```
## Bug Report

### Bug 1: <short description>
**Location:** <file:line>
**What happens:** <description of the incorrect behavior>
**Conditions:** <when/how this bug triggers>
**Test file:** <path to the test file>
**Test result before fix:** FAIL — <error message>
**Fix:** <description of what to change, with file:line>
**Test result after fix:** PASS

### Bug 2: <short description>
...

### Files created/modified (all unstaged)
- <path> — test file
- <path> — fix applied
```

**Output format when no bugs are found:**

```
## Bug Report

No bugs found. Edge cases tested:
- <case 1> — passed
- <case 2> — passed
- <case 3> — passed

All suspected issues were verified with tests and none failed.
```

## Error Handling

- **Test runner not found or fails to start:** Report the issue, output the test code for the user to run manually
- **Test infrastructure requires setup** (database, env vars): Ask the user for setup instructions, or report suspected bugs without proof and flag reduced confidence
- **Cannot determine test conventions:** Ask the user where tests go and what runner to use

## Key Rules

- **Never report a bug without a failing test** — suspected-but-unproven bugs are not bugs
- **Never claim a fix works without the test passing** — run the test, show the output
- **Do not commit anything** — all changes left unstaged for user review
- **Delete disproven bugs** — if a test passes (no bug), remove the test file and don't mention it
- **Minimal fixes only** — fix the specific bug, don't refactor surrounding code

Work from: /Users/ziyi/Work/claude-bugfix-workflow
