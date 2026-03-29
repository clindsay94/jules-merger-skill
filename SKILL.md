---
name: jules-merger
description: "Autonomously process, pull, publish, and merge completed Jules sessions. Use when the user asks to merge Jules sessions, orchestrate Jules PRs, or process completed Jules tasks."
license: Apache-2.0
metadata:
  author: gemini-cli
  version: "1.0"
---

# Jules Session Merger Orchestrator

This skill provides an automated workflow to process completed Jules sessions, apply them to local repositories, create Pull Requests on GitHub, and merge them into the main branch.

## Workflow Instructions

When invoked, follow this exact workflow autonomously:

### 1. List All Jules Sessions
Run the command to list all Jules sessions and identify those with a `Completed` status:
```bash
jules remote list --session
```

### 2. Identify Target Repositories
Parse the list to find the session IDs, descriptions, and repositories for the `Completed` sessions.
*Note: If there are many completed sessions, you can process them one by one or in batches.*

### 3. Check Existing Pull Requests
Before processing a session, use the GitHub MCP tools or `gh` CLI to list existing Pull Requests in the target repository to avoid duplicating work.
```bash
# Using gh CLI
gh pr list --state all
```

### 4. Process Each Session
For each unresolved `Completed` session, perform the following steps:

#### A. Navigate to the Local Repository
Change your working directory to the local path of the repository associated with the session.

#### B. Prepare the Branch
Update the local repository and create a new branch for the session:
```bash
git checkout main
git pull
git checkout -b jules/<session_id>
```

#### C. Pull and Apply Jules Changes
Use the `jules remote pull` command to download and apply the patch:
```bash
jules remote pull --session <session_id> --apply
```
*Note: If the patch fails to apply (e.g., due to conflicts with more recent changes), abort the current session processing, delete the branch (`git checkout main; git branch -D jules/<session_id>`), and move on to the next session.*

#### D. Commit and Push Changes
Stage all applied changes, commit them with a descriptive message, and push to the remote repository:
```bash
git add -A
git commit -m "Jules session <session_id>"
git push -u origin jules/<session_id>
```

#### E. Create a Pull Request
Use the GitHub MCP tool `mcp_github_create_pull_request` (or `gh pr create`) to create a PR. Set the title based on the Jules task description (e.g., "🧹 Code Health Improvement (Jules <session_id>)") and provide a descriptive body.

#### F. Merge the Pull Request Locally and Push
After creating the PR, merge the branch into `main` locally and push to origin:
```bash
git checkout main
git merge jules/<session_id>
git push origin main
```

### 5. Repeat and Report
Repeat Step 4 for all identified `Completed` sessions. Once all sessions are processed, provide the user with a summary report of the actions taken, including the repositories updated and the PR numbers created/merged.

## Error Handling
- **Patch Application Failure:** If `jules remote pull --apply` fails due to merge conflicts or outdated code, safely revert the changes (`git reset --hard`), switch back to `main`, delete the created branch, and notify the user that the session was skipped due to conflicts.
- **Repository Missing:** If the local repository cannot be found, ask the user for the correct path or clone the repository to a temporary workspace.
