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

#### A. Retrieve Jules Changes (Smart Branch Selection)
Determine if Jules pushed a remote branch for this session.
1. **Fetch Remote:** `git fetch origin`
2. **Check Branch:** Check if `origin/jules/<session_id>` exists.
   - **Case 1: Remote Branch Exists:**
     - Checkout the branch: `git checkout jules/<session_id>`
     - Ensure it is up to date with `main`: `git merge origin/main`
   - **Case 2: No Remote Branch:**
     - Create a fresh branch from main: `git checkout main; git pull; git checkout -b jules/<session_id>`
     - Apply changes using Jules: `jules remote pull --session <session_id> --apply`
     - **Conflict Recovery:** If `apply` fails, read the diff (`jules remote pull --session <session_id>`) and manually apply the changes using your `replace` tool, skipping sections that refer to missing files or outdated contexts.

#### B. Deduplication Check
Before proceeding, verify if the changes are already present in the target branch:
- Run `git log -n 20 --oneline` to see if a commit with the session ID or a similar description already exists.
- If the changes are already merged, skip this session.

#### C. Local Code Review & Security Audit
Review the code changes introduced by Jules:
1. Run the `lint_command` from `.jules-merger.yml` (e.g., `dotnet format analyzers`, `npm run lint`).
2. Check for hardcoded secrets, glaring logic errors, or style violations.
3. If minor issues are found, fix them locally using your tools.

#### D. Pre-Merge Test Verification (Auto-Fix Protocol)
Run the project's test suite:
1. **Environment Setup:** If `requirements.txt`, `package.json`, or `pyproject.toml` exists, ensure dependencies are installed (e.g., `pip install -r requirements.txt`).
2. **Run Tests:** Run the `test_command` from `.jules-merger.yml`.
3. **Handle Failures:**
   - **If Tests Pass:** Proceed to the next step.
   - **If Tests Fail:**
     - Analyze the failure output.
     - Attempt to fix the broken test or implementation locally (up to `auto_fix_attempts`).
     - If still failing, proceed but mark the PR as a `Draft`.

#### E. Commit and Push Changes
Commit any local fixes and push the branch:
```bash
git add -A
git commit -m "fix(jules): merge and test fixes for session <session_id>"
git push origin jules/<session_id>
```

#### F. Publish the Pull Request
Create the PR using the GitHub MCP tool `mcp_github_create_pull_request`. 
- **Head Branch:** `jules/<session_id>`
- **Base Branch:** `target_branch`
- **Title:** e.g., "🧹 Code Health Improvement (Jules <session_id>)"
- **Body:** Include Jules's description and a "Merger Report" section documenting test results and any manual fixes applied.

#### G. Merge Protocol
- **If tests passed and review is clean:** Merge the PR locally and push.
- **If tests failed or issues were flagged:** Leave the PR open and notify the user.

### 4. Final Report
Once all sessions are processed, provide a summary of:
- Which sessions were successfully merged.
- Which sessions were pushed as PRs but require human review (and why).
- Which sessions were skipped or re-delegated due to severe conflicts.
