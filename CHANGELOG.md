# Changelog

## [2.4.0] — 2026-04-24

### Changed

- **PR Review Workflow: Manual Testing is now optional and runs after Fix Proposals**
  Previously, Manual Testing (Phase 4) gated Fix Proposals (Phase 6). If bugs were caught statically in earlier phases, users still had to complete a potentially lengthy manual test run before review comments could be generated.
  The new phase order is: Smoke Test → Code Analysis → Breaking Changes → Alternatives → **Bug Hunting** → **Fix Proposals** → **Manual Testing (optional)** → Docs Impact.
  The Manual Testing checkpoint now includes a "Skip — static analysis was sufficient" option.

---

## [2.3.0] — 2026-04-21

### Added

- **`review-comments` skill (Phase 6 of PR Review Workflow)**
  Generates ready-to-post GitHub review comments by synthesizing findings from all earlier review phases plus a fresh read of the diff. Groups comments by file with friendly, reasoning-forward tone and concrete code snippets where helpful.

---

## [2.2.0] — 2026-04-08

### Added

- **`review-manual-test` skill (Phase 4 of PR Review Workflow)**
  Follows the PR's "How to test" instructions end-to-end: executes Admin UI steps via Playwright MCP and generates a Bruno API collection for HTTP request steps.

---

## [2.1.0] — 2026-03-25

### Added

- **`review-bugs` skill (Phase 5 of PR Review Workflow)**
  Hunts for bugs in changed code, proves each one with a failing test, applies a minimal fix, and leaves all files unstaged for review. Never reports a potential bug without a failing test.

---

## [2.0.0] — 2026-03-10

### Added

- **Issue Triage Workflow** — `issue-start` → `issue-analyze` with optional handoff to the bug fix workflow
- **`docs-update` skill** — detects documentation impact from a diff and drafts MDX updates using the Strapi docs Router + Drafter pipeline

### Changed

- PR Review Workflow expanded to include `review-analyze` (three-pass: code match, breaking changes, alternatives)

---

## [1.0.0] — 2026-02-14

### Added

- Initial release: Bug Fix Workflow (`bugfix-start` → `bugfix-analyze` → `bugfix-fix` → `obsidian` → `pr`) and PR Review Workflow (`review-start` → `review-smoke`)
