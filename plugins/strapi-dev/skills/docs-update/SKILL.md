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

**When run via orchestrator:** Use the diff passed in from the orchestrator instead of running git commands. Only run git commands in standalone mode.

### Step 2: Detect Doc-Relevant Changes

Analyze the diff for changes that have documentation impact:
- API endpoint behavior changes (request/response format, status codes, errors)
- Configuration option changes (new options, changed defaults, removed options)
- Error message changes
- CLI flag or command changes
- Environment variable changes
- Breaking changes in public interfaces

**If nothing doc-relevant:** Exit silently — "No documentation updates needed, moving on." Do not proceed to Steps 3–6.

### Step 3: Check Internal Project Docs

Search the current repo for documentation that references the changed behavior:
- Use Grep to search `README.md`, `docs/`, `CONTRIBUTING.md`, and inline JSDoc/docstrings
- Check if any references are now outdated due to the fix

**If internal docs are affected:**

Present the proposed changes with before/after for each affected file.

**Checkpoint:**

```
Found <N> internal doc file(s) that reference changed behavior.

Proposed changes:
<summary of each file and what would change>

  [1] Approve — apply the updates
  [2] Adjust the proposed changes (describe what to change)
  [3] Skip docs update
  [4] Stop here
```

- `[1]` → Apply the changes to internal docs and commit to the same branch
- `[2]` → Incorporate the adjustment, re-present with the same menu
- `[3]` → Skip internal docs; proceed to Step 4
- `[4]` → Stop the workflow. Leave all changes as-is (unstaged/unsaved).

**If no internal docs are affected:** Proceed to Step 4 silently.

### Step 4: Router Phase

**Purpose:** Determine where in the Strapi documentation the change belongs and what action is needed.

**GitHub MCP calls:**
```
github:get_file_contents(owner: "strapi", repo: "documentation", path: "agents/prompts/router.md")
github:get_file_contents(owner: "strapi", repo: "documentation", path: "docusaurus/sidebars.js")
github:get_file_contents(owner: "strapi", repo: "documentation", path: "docusaurus/static/llms.txt")
```

**Note on llms.txt size:** If `llms.txt` is too large to use in full alongside `sidebars.js`, the Router prompt, and the diff, filter it to entries relevant to the feature area of the diff (e.g., if the diff touches relations code, keep only entries mentioning "relations", "document service", "content API"). Inform the user if truncation was applied: "Note: llms.txt was filtered to <N> relevant entries due to size."

**Input to Router:**
- Source material: the diff (changed files + code changes)
- `sidebars.js`: navigation hierarchy
- `llms.txt`: page index (full or filtered)

**Process:** Follow the Router prompt exactly as fetched. Do not override its placement decisions.

**Output:** Router YAML report with targets, priorities, and actions.

**Zero-target exit:** If the Router returns an empty targets list, report "Router determined no Strapi documentation changes are needed for this diff." and exit. Do not proceed to Steps 5–6.

**ask_user handling:** If any target in the Router output has `ask_user` set, surface that question to the user and wait for their answer before presenting the Router checkpoint. Do not guess.

**Checkpoint:**

```
Router identified <N> Strapi doc page(s) to update:

<For each target:>
- `<page path>` — <action> (<priority>)
  <one-line summary of what changes>

  [1] Approve — proceed to drafting
  [2] Remove a target (specify which)
  [3] Adjust a target (specify which and how)
  [4] Stop here
```

- `[1]` → Approved. Proceed to Step 5.
- `[2]` → Remove the specified target from the list. Re-present the updated list with the same menu.
- `[3]` → Apply the specified adjustment to the target (user-directed override to Router output — do not re-run the Router). Re-present the updated list with the same menu.
- `[4]` → Stop the workflow. Leave all changes as-is.

### Step 5: Drafter Phase

**Purpose:** Write style-compliant, publication-ready MDX content for each target page.

**Fetch timing:** Fetch `drafter.md` after the Router checkpoint passes — do not fetch it before.

**GitHub MCP calls:**
```
github:get_file_contents(owner: "strapi", repo: "documentation", path: "agents/prompts/drafter.md")
  → fetch once, reuse for all targets

github:get_file_contents(owner: "strapi", repo: "documentation", path: "docusaurus/docs/<target-path>")
  → fetch only for targets where action is NOT create_page (page must already exist)
  → skip this call for create_page targets — the page does not exist yet

github:get_file_contents(owner: "strapi", repo: "documentation", path: "<template-path>")
github:get_file_contents(owner: "strapi", repo: "documentation", path: "<guide-path>")
  → fetch only if specified in Router output for the target's doc type
```

**Mode selection** (follow Drafter prompt's rule):
- `create_page` or `add_section` → **Compose**
- `update_section`, `update_text`, `add_row` → **Patch**
- `add_link`, `add_mention`, `add_tip` → **Micro-edit**

**Process:** Follow the Drafter prompt exactly as fetched. Apply all writing rules, output format, and quality checklist from the Drafter prompt.

**Output:** Structured MDX content per target wrapped in the Drafter's output envelope (`<!-- drafter:mode=... -->`).

**Checkpoint:**

```
Here are the proposed Strapi doc changes (<N> file(s)):

<For each target:>
### `<page path>` — <mode>
<drafted content>

---

  [1] Approve — present for submission
  [2] Regenerate a specific change (specify which)
  [3] Edit a change manually (specify which)
  [4] Stop here
```

- `[1]` → Approved. Proceed to Step 6.
- `[2]` → Re-run the Drafter for the specified target from scratch. Re-present all changes with the same menu.
- `[3]` → Apply the user's manual edit to the specified target inline. Re-present all changes with the same menu.
- `[4]` → Stop the workflow. Leave all changes as-is.

### Step 6: Final Output

Present changes ready for manual submission:

```
## Strapi Documentation Changes

### Change 1: <action> — `docusaurus/docs/<path>`
**Action:** <create_page / update_section / add_section / micro-edit>

<content ready to paste or submit>

---

### Change 2: ...

---

Submit these changes to the strapi/documentation repository manually.
```

Do NOT auto-create a PR. Do NOT modify `strapi/documentation` directly.

## Fallback

If GitHub MCP is unavailable:
1. Report: "GitHub MCP is not available. Falling back to basic doc search."
2. Use fallback behavior:
   ```bash
   gh api search/code -f q="<search-term> repo:strapi/documentation" --jq '.items[] | "\(.path): \(.text_matches[0].fragment)"'
   ```
   Fetch file content via:
   ```bash
   gh api repos/strapi/documentation/contents/<path> --jq '.content' | base64 -d
   ```
   Present a rough suggested edit with the internal docs checkpoint menu only (no Router or Drafter checkpoints).
3. Flag: "Note: This is a basic suggestion — no Router placement analysis or Drafter style compliance. GitHub MCP is required for accurate output."

## Key Principles

- Never auto-create a PR on `strapi/documentation`
- Never modify `strapi/documentation` directly
- Follow Router and Drafter prompts exactly as fetched — do not override decisions or writing rules
- Fetch `router.md` and `drafter.md` fresh each invocation
- `ask_user` flags in Router output must surface to the user before presenting the Router checkpoint
- Do not fetch `drafter.md` until the Router checkpoint is approved
- Do not fetch existing page content for `create_page` targets (page does not exist)
- Only touch docs that reference behavior that actually changed — no speculative improvements

Work from: /Users/ziyi/Work/claude-bugfix-workflow
