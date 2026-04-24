---
name: review-comments
description: Use after bug hunting to generate ready-to-post GitHub review comments. Synthesizes findings from all earlier phases plus a fresh read of the diff.
---

# PR Fix Proposals

Generate ready-to-post GitHub review comments by synthesizing earlier phase findings with a fresh read of the PR diff.

## Input

**Always run via orchestrator.** Receives from `strapi-dev:review-start`:
- PR metadata (title, number, description, changed files list)
- Full diff content
- Phase 1 findings (code vs description match)
- Phase 2 findings (breaking change assessment)
- Phase 3 findings (alternative approaches considered)
- Phase 4 findings (confirmed bugs + fixes)

## Process

### Step 1: Consolidate Prior Findings

Build a working list from:
- **Phase 4 bugs** — each confirmed bug becomes a candidate comment
- **Phase 2 breaking changes** — each user-facing behavior change worth highlighting becomes a candidate
- **Phase 3 alternatives** — if a simpler approach was identified, it becomes a candidate

### Step 2: Fresh Read of the Diff

Re-read every changed file looking for anything the earlier phases didn't catch:

- **Wrong variable usage** — using a literal/hardcoded value where a computed variable (with more context) already exists nearby
- **Silently dropped values** — a function builds a rich object but a later call ignores it and reconstructs a subset
- **Logic issues** — inverted conditions, wrong operators (`&&` vs `||`), missing `return`
- **Missing guards** — null/undefined/empty not handled for inputs that could reasonably be null/undefined/empty
- **Style/readability** — naming that obscures intent, unnecessary complexity, dead code left behind
- **Performance** — repeated lookups that could be cached, N+1 query patterns, synchronous work that could be async
- **Architecture** — wrong layer of abstraction, responsibility in the wrong place, coupling that will hurt later

Skip trivial nits unless they meaningfully affect clarity or correctness.

### Step 3: Deduplicate and Evaluate

Merge Step 1 and Step 2 candidates. Remove duplicates. Each remaining item becomes one comment.

Discard anything that reads as nitpicking without clear benefit.

### Step 4: Generate Comments

Write each comment in the format below, grouped by file.

## Output Format

```
---
### Comment N
**File:** `<relative/path/to/file.ts>`
**Line:** <line number>

<Friendly, first-person explanation — lead with WHY this matters before proposing
WHAT to change. Be specific: name the variable, explain what it drops, show the
consequence. Conversational tone. Emojis welcome.>

```ts
// proposed replacement (include when a concrete snippet makes the suggestion
// clearer; omit for pure style/architecture observations)
```
---
```

**Tone guide:**
- "Shouldn't this use `findPageParams` instead of the hardcoded object? `findPageParams` includes the `hasPublishedVersion` pass-through — that gets silently dropped here."
- "I think this condition is inverted — when `isPublished` is true we'd be skipping the publish step, not running it."
- "This could be `null` if the content type has no lifecycle hooks — worth a guard here."

End with: `**N comments generated**`

**If no comments remain after deduplication**, output:

```
**0 comments generated** — diff reviewed; no actionable suggestions beyond what earlier phases already covered.
```

## Checkpoint

```
  [1] Looks good — continue to docs check
  [2] Generate more suggestions for a specific area (specify)
  [3] Remove a suggestion (specify which)
  [4] Stop here
```

- `[2]` — re-read the specified area, append any new suggestions, re-present
- `[3]` — remove the specified comment number, re-present the updated list
- `[4]` — stop; comments remain displayed in terminal for manual copy-paste

## Key Rules

- **Display only** — never post comments to GitHub
- **Friendly reasoning** — always explain why before proposing what
- **Specific citations** — name the exact variable, line, or behavior
- **No duplication** — if Phase 4 already proved a bug with a test, reference it; don't re-explain the test
