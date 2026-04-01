---
name: review-manual-test
description: Use when following a PR's manual testing instructions. Executes Admin UI steps via Playwright MCP and creates a Bruno API collection for HTTP request steps.
argument-hint: <pr-url-or-number>
---

# PR Manual Testing

Follow a PR's "How to test" instructions. Execute Admin UI steps via Playwright MCP and create a Bruno API collection for HTTP request steps.

## Input

**When run via orchestrator:** Receives PR metadata (title, body, number) from `review-start`.

**When run standalone:** Ask the user for a PR URL or number, then fetch:
```bash
gh pr view <pr-number> --json title,body,number
```

## Process

First, resolve both configuration paths per the Working Directory section below.

### Step 1: Parse Testing Instructions

Scan the PR body for a section whose heading matches any of:
- How to test
- Testing
- Testing steps
- Steps to reproduce
- Verification
- Manual testing

If no such section is found, output:
```
No manual testing instructions found in this PR description.
Skipping manual test phase.
```
Then show the checkpoint and exit.

### Step 2: Classify Steps

For each step found in the testing section, classify it as one of:
- **UI** — involves the Admin panel, a browser, clicking, navigating, or filling forms
- **API** — involves an HTTP request (GET, POST, PUT, DELETE, PATCH) or references an API endpoint
- **Setup** — prerequisite action (create a content type, install a plugin, run a shell command)
- **Verify** — check a result (confirm something appears in the UI, verify response contains a field)

### Step 3: Execute Setup Steps

For each setup step:
- Shell commands → run via Bash tool
- Content type creation or admin UI setup → note as a manual prerequisite and list it for the user

### Step 4: Execute UI Steps via Playwright MCP

For each UI step, use the Playwright MCP tools to:
1. Navigate to the Admin panel (`http://localhost:1337/admin`)
2. Perform the described interaction (click a button, fill a form, submit)
3. Capture the result (screenshot or text confirmation)

If Playwright MCP is unavailable, output all UI steps as a numbered manual checklist:
```
## UI Steps (run manually — Playwright unavailable)
1. <step>
2. <step>
```

### Step 5: Create Bruno Collection for API Steps

Create the sub-folder for this PR:
```bash
mkdir -p <bruno-collection-path>/pr-<number>
```

For each API step, write a `.bru` file to `<bruno-collection-path>/pr-<number>/`. Use sequence numbers starting at 1.

Bruno `.bru` file format — GET request example:
```
meta {
  name: Get Articles
  type: http
  seq: 1
}

get {
  url: http://localhost:1337/api/articles
  body: none
  auth: none
}
```

Bruno `.bru` file format — POST with JSON body example:
```
meta {
  name: Create Article
  type: http
  seq: 2
}

post {
  url: http://localhost:1337/api/articles
  body: json
  auth: none
}

body:json {
  {
    "data": {
      "title": "Test Article"
    }
  }
}
```

Supported methods: `get`, `post`, `put`, `patch`, `delete`. Match the method from the PR's instructions.

After writing all `.bru` files, tell the user:
```
Bruno collection created: <bruno-collection-path>/pr-<number>/
Open Bruno and run the requests in sequence to execute the API steps.
```

### Step 6: Report Results

```
## Manual Test Results — PR #<number>

### Setup
- <step description> — done / manual action required

### UI Steps (Playwright)
- <step description> — passed / failed / manual
  <error or confirmation detail>

### API Steps (Bruno)
Collection: <bruno-collection-path>/pr-<number>/
Files created:
- <filename>.bru — <request name>
- <filename>.bru — <request name>
```

## Checkpoint

```
  [1] Looks good — continue to bug hunting
  [2] Re-run a specific step (specify which)
  [3] A step failed — investigate (specify which)
  [4] Stop here
```

Wait for the user's selection. If `[2]`, re-run the specified step, update the report, and re-present with the same menu. If `[3]`, investigate the specified failure — re-run it with verbose output, examine the error, and update the report — then re-present with the same menu. If `[4]`, stop, leaving all Bruno files in place.

## Working Directory

Before starting, resolve both paths from memory:

**Strapi repo path:**
1. Check Claude's memory for "strapi repo path"
2. If not found → ask: "Where is your Strapi monorepo checked out? (e.g., `/path/to/strapi`)"
3. Save to memory

**Bruno collection path:**
1. Check Claude's memory for "bruno collection path"
2. If not found → ask: "Where is your Bruno collection folder? (e.g., `/Users/you/api-testing`)"
3. Save to memory
