---
name: bugfix-fix
description: Use when a bug has been analyzed and you need to find the root cause, propose fix options, and implement the chosen fix with tests. Follows TDD and systematic debugging.
---

# Bug Fix

Find the root cause, propose fix options, and implement the chosen fix with tests.

## Input

**When run via orchestrator:** Receives analysis context from `strapi-dev:bugfix-analyze` (bug summary, affected code, reproduction result, initial hypothesis).

**When run standalone:** Ask the user to describe:
1. What is the bug? (symptoms, error messages)
2. Where in the codebase? (files, functions, feature area)

Claude will explore the codebase to build context before proceeding.

## Process

### Step 1: Root Cause Identification

Apply systematic debugging — evidence-based, no guessing:
- Start from the reproduction or the initial hypothesis from the analyze phase
- Form a hypothesis, then verify it by reading code and running tests
- Narrow down: trace the execution path to find the exact root cause
- Gather evidence at each step — do not skip ahead
- Present the root cause with file/line references:

```
## Root Cause

The root cause is [X] because [Y].

**Evidence:**
- `path/to/file.ts:123` — [what this code does wrong]
- [trace of how the bug manifests from this code]

**Why this causes the reported symptom:**
[Explanation connecting root cause to user-visible bug]
```

### Step 2: Propose Fixes

Propose **2-3 fix options** (when applicable — simple bugs may only have one reasonable fix):

```
## Fix Options

### Option A: [Name] (Recommended)
- **Change:** [what to modify]
- **Scope:** [files affected]
- **Risk:** [low/medium/high — what could break]
- **Trade-off:** [any downsides]

### Option B: [Name]
- **Change:** [what to modify]
- **Scope:** [files affected]
- **Risk:** [low/medium/high]
- **Trade-off:** [any downsides]

**Recommendation:** Option A because [reasoning].
```

**Checkpoint 1:** "Which approach do you want to go with?"

### Step 3: Implement the Fix

Follow TDD:

1. **Write a failing test** that reproduces the bug
   - The test should fail before the fix and pass after
   - Run it to confirm it fails

2. **Implement the minimal fix**
   - Only change what's necessary to fix the bug
   - Do not refactor surrounding code
   - Do not add unrelated improvements

3. **Run all tests**
   - Run the new test to confirm it passes
   - Run existing tests in the affected area to check for regressions
   - If tests fail, investigate and fix before proceeding

4. **Commit the changes**
   - Stage only the relevant files
   - Commit to the current `fix/<slug>` branch

**Checkpoint 2:** "Fix implemented and tests pass. Here's a summary of changes. Ready to check docs?"

## Error Handling

- **Tests fail after fix:** Report the failures. Do not proceed to docs/PR phases. Ask user how to proceed.
- **Root cause unclear:** Present what's known, flag uncertainty, ask user for additional context.

## Key Principles

- **TDD:** Bug-reproducing test first, then the fix makes it pass
- **Minimal changes:** Fix the bug, don't refactor the neighborhood
- **Evidence-based:** No guessing — trace the code, verify the hypothesis
