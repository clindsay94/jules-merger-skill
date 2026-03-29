---
name: jules-merger
description: "Autonomously process, pull, publish, and merge completed Jules sessions with built-in code review, test verification, and state recovery. Use when the user asks to merge Jules sessions, orchestrate Jules PRs, or process completed Jules tasks."
license: Apache-2.0
metadata:
  author: gemini-cli
  version: "1.2"
---

# Jules Session Merger Orchestrator

This skill provides an automated, resilient workflow to process completed Jules sessions, apply them to local repositories, verify the code, and manage Pull Requests.

## Workflow Instructions

When invoked, follow this exact workflow autonomously:

### Step 0: Pre-Flight & Configuration
1. **Clean Tree Check:** Ensure the working directory is clean (`git status`). If it is not, stash or discard changes before proceeding.
2. **State Recovery:** Consult `RECOVERY.md` to prune stray branches or resolve locked states (`git rebase --abort`, `git merge --abort`).
3. **Load Configuration:** Look for `.jules-merger.yml` in the target repository root. 
   - If found, parse it to define `target_branch`, `test_command`, and `lint_command`.
   - If NOT found, default to `main` and attempt to infer the test command.

### 1. List All Jules Sessions
Run the command to list all Jules sessions and identify those with a `Completed` status:
```bash
jules remote list --session
```

### 2. Check Existing Pull Requests
Before processing a session, use the GitHub MCP tools or `gh` CLI to list existing Pull Requests in the target repository to avoid duplicating work.
```bash
# Using gh CLI
gh pr list --state all
```

### 3. Process Each Session (The Resilient Pipeline)
For each unresolved `Completed` session, perform the following steps:

#### A. Fetch and Checkout Jules Branch
Jules automatically creates and pushes a branch for its session on the remote repository. Fetch and check out this existing branch:
```bash
git fetch origin
git checkout jules/<session_id>
```

#### B. Local Code Review & Security Audit
Review the code changes introduced by Jules:
1. Run the `lint_command` from `.jules-merger.yml` (e.g., `dotnet format analyzers`, `npm run lint`).
2. Check for hardcoded secrets, glaring logic errors, or style violations.
3. If minor issues are found, fix them locally using your tools.
4. If major architectural flaws or security vulnerabilities are found, DO NOT auto-fix. Note the issues for the PR description, and flag the upcoming PR as a Draft.

#### C. Pre-Merge Test Verification (Auto-Fix Protocol)
Run the project's test suite using the `test_command` from `.jules-merger.yml`:
- **If Tests Pass:** Proceed to the next step.
- **If Tests Fail (or are missing):**
  1. Analyze the test failure output.
  2. Attempt to fix the broken test or the buggy implementation locally using your own tools.
  3. Re-run the tests.
  4. If the tests remain broken after a reasonable attempt (up to `auto_fix_attempts`), proceed but mark the upcoming PR as a `Draft` or add a "Needs Human Review" label.

#### D. Commit and Push Local Fixes
If you made any local changes during the review or testing phase, commit and push them to the Jules branch:
```bash
git add -A
git commit -m "fix(jules): local test and review fixes"
git push origin jules/<session_id>
```

#### E. Publish the Pull Request
Create the PR using the GitHub MCP tool `mcp_github_create_pull_request`. 
- **Head Branch:** `jules/<session_id>`
- **Base Branch:** The configured `target_branch` (defaults to `main`)
- **Title:** e.g., "🧹 Code Health Improvement (Jules <session_id>)"
- **Body:** Include Jules's original description. *Crucially, if tests are still failing or you found security warnings during your local review, explicitly document them in the PR body.*

#### F. Merge Protocol
- **If tests passed and review is clean:** Merge the PR locally and push (`git checkout <target_branch>; git merge jules/<session_id>; git push origin <target_branch>`).
- **If tests failed or review flagged issues:** DO NOT merge. Leave the PR open for the human user to review, notify the user, and move on to the next session.

### 4. Final Report
Once all sessions are processed, provide a summary of:
- Which sessions were successfully merged.
- Which sessions were pushed as PRs but require human review (and why).
- Which sessions were skipped or re-delegated due to severe conflicts.
