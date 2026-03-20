---
name: review-analyze
description: Use when analyzing a PR for code correctness, breaking changes, or alternative approaches. Supports three analysis passes — code vs description, breaking changes (Strapi-specific), and alternatives.
argument-hint: <pr-url>
---

# PR Analysis

Analyze a pull request across three dimensions: code vs description match, breaking changes, and alternative approaches.

## Input

The user provides: $ARGUMENTS

**When run via orchestrator:** Receives a pass indicator and PR context as arguments.
- `pass:1` — code vs description match
- `pass:2` — breaking change analysis
- `pass:3` — alternative approaches

**When run standalone:** Accepts a PR URL or auto-detects from the current branch. Runs all three passes sequentially with checkpoints between each.

### Standalone PR Fetch

If no pass indicator is provided, fetch the PR metadata:

```bash
gh pr view [URL] --json title,body,number,baseRefName,headRefName,files
gh pr diff [URL]
```

If `gh` fails, ask the user to paste the PR description and provide the branch name.

## Pass 1: Code vs Description Match

Read the PR description and title. Read every changed file in the diff. For each change, explain in simple terms what the code does and why.

Compare against the stated intent:

- **Missing:** things described in the PR but not implemented in the code
- **Extra:** things implemented in the code but not mentioned in the PR description

### Process

1. Parse the PR description for stated goals and changes
2. Read each changed file using the Read tool (use the diff to identify file paths)
3. For each file, describe the change in plain language
4. Cross-reference code changes with PR description
5. Flag any discrepancies

### Output Format

```
## Code vs Description

### What the PR says it does
<summary of PR description>

### What the code actually does
<file-by-file explanation in simple terms>

### Gaps
- **Missing:** <things described but not implemented>
- **Extra:** <things implemented but not described>
- **Accurate:** <things that match between description and code>
```

**Checkpoint (standalone only):** "Does this match your read of the PR?"

---

## Pass 2: Breaking Change Analysis (Strapi-specific)

For each changed file, determine whether the change could affect Strapi users.

### Process

1. **Trace exports:** For each changed file, check if its exports are re-exported through the package's `index.ts` or entry point
   ```bash
   # Find the package this file belongs to
   # Check the package's main entry point for re-exports
   ```

2. **Check package boundaries:** Trace re-exports across Strapi packages
   - Check if `@strapi/strapi` re-exports from `@strapi/core`
   - Check if `@strapi/utils` exports are used by other packages
   - Check internal packages like `@strapi/content-manager`, `@strapi/content-type-builder`

3. **Check user-reachable paths:** Can a Strapi user's code call or reach this through:
   - Custom controllers, services, or middlewares
   - Lifecycle hooks (`beforeCreate`, `afterUpdate`, etc.)
   - Custom policies
   - Plugin extensions
   - Direct `strapi.` API access (e.g., `strapi.documents`, `strapi.db`)

4. **Check API behavior:** Does this change alter:
   - REST API or GraphQL response format, status codes, or error messages
   - Admin API behavior
   - Content API filtering, sorting, or population behavior

5. **Check data layer:** Does this change affect:
   - Database schema or migrations
   - Query behavior (different SQL generated, different results returned)
   - Entity/document lifecycle

### Output Format

```
## Breaking Change Analysis

### Changed files and their exposure
| File | Exported? | Reachable by user code? | How? |
|------|-----------|------------------------|------|

### Risk assessment
**Risk level:** <high / medium / low / none>
<explanation of why>

### User-facing behavior changes
- <change 1 and its impact>
- <change 2 and its impact>
- (or "No user-facing behavior changes detected")
```

**Checkpoint (standalone only):** "Any areas you want me to dig deeper into?"

---

## Pass 3: Alternative Approaches

Identify the problem the PR is solving and propose 2-3 alternative ways to solve it.

### Process

1. Extract the problem statement from the PR description and code changes
2. Understand the constraints (why was this approach chosen?)
3. Propose alternatives — focus on:
   - Simpler approaches with smaller scope
   - Approaches that don't touch public APIs
   - Approaches that avoid adding new parameters/options
4. For each alternative, assess: scope, risk, simplicity
5. Explicitly consider: "Is this change necessary at all? Could it be done more simply?"

### Output Format

```
## Alternative Approaches

### Problem being solved
<one-liner>

### The PR's approach
<summary>
- **Scope:** <files/areas touched>
- **Pros:** <advantages>
- **Cons:** <disadvantages>

### Alternative A: <name>
- **Change:** <what to modify>
- **Scope:** <files affected>
- **Pros:** <why it might be better>
- **Cons:** <why it might be worse>

### Alternative B: <name>
- **Change:** <what to modify>
- **Scope:** <files affected>
- **Pros:** <why it might be better>
- **Cons:** <why it might be worse>

### Recommendation
<which approach is best and why — could be the PR's current approach>
```

**Checkpoint (standalone only):** "Thoughts on these alternatives?"

## Key Principle

This skill is **read-only** — no code changes, only analysis.
