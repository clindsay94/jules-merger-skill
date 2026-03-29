---
name: jules-merger
description: "Autonomously process, pull, publish, and merge completed Jules sessions with built-in code review and test verification. Use when the user asks to merge Jules sessions, orchestrate Jules PRs, or process completed Jules tasks."
license: Apache-2.0
metadata:
  author: gemini-cli
  version: "1.1"
---

# Jules Session Merger Orchestrator

This skill provides an automated, resilient workflow to process completed Jules sessions, apply them to local repositories, verify the code, and manage Pull Requests.

## Workflow Instructions

When invoked, follow this exact workflow autonomously:

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

#### A. Navigate & Branch
Update the local repository and create a new branch:
```bash
git checkout main
git pull
git checkout -b jules/<session_id>
```

#### B. Pull and Apply Jules Changes
```bash
jules remote pull --session <session_id> --apply
```
**Conflict Resolution Protocol:** 
If the patch fails to apply (e.g., due to an outdated baseline):
1. Safely revert (`git reset --hard; git clean -fd`).
2. Read the failed patch file locally to understand what Jules was trying to do.
3. Attempt to manually apply the intended changes using your own editing tools (`replace`, `write_file`).
4. If the conflict is too complex, submit a new Jules task (`jules new`) with the current file context and skip this session for now.

#### C. Local Code Review & Security Audit
Before committing, review the uncommitted diff:
```bash
git diff
```
1. Check for hardcoded secrets, glaring logic errors, or style violations.
2. If minor issues are found, fix them locally using your tools.
3. If major architectural flaws are found, stage what you can, but note the issues for the PR description.

#### D. Pre-Merge Test Verification (Auto-Fix Protocol)
Run the project's test suite (e.g., `dotnet test`, `pytest`, `npm test`):
- **If Tests Pass:** Proceed to commit.
- **If Tests Fail (or are missing):** DO NOT discard the branch. 
  1. Analyze the test failure output.
  2. Attempt to fix the broken test or the buggy implementation locally using your own tools.
  3. Re-run the tests. If you successfully fix them, proceed.
  4. If the tests remain broken after a reasonable attempt, proceed to commit but mark the upcoming PR as a `Draft` or add a "Needs Human Review" label.

#### E. Commit and Push Changes
```bash
git add -A
git commit -m "Jules session <session_id>"
git push -u origin jules/<session_id>
```

#### F. Publish the Pull Request
Create the PR using the GitHub MCP tool `mcp_github_create_pull_request`. 
- **Title:** e.g., "🧹 Code Health Improvement (Jules <session_id>)"
- **Body:** Include Jules's original description. *Crucially, if tests are still failing or you found security warnings during your local review, explicitly document them in the PR body.*

#### G. Merge Protocol
- **If tests passed and review is clean:** Merge the PR locally and push (`git checkout main; git merge jules/<session_id>; git push origin main`).
- **If tests failed or review flagged issues:** DO NOT merge. Leave the PR open for the human user to review, notify the user, and move on to the next session.

### 4. Final Report
Once all sessions are processed, provide a summary of:
- Which sessions were successfully merged.
- Which sessions were pushed as PRs but require human review (and why).
- Which sessions were skipped or re-delegated due to severe conflicts.
