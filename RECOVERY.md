# Agent Recovery Protocol

This document provides explicit instructions for the `jules-merger` agent to follow if it encounters an interrupted state, a crashed pipeline, or an unclean working tree upon initialization.

## 1. Stray Branch Cleanup
If the agent crashes or is interrupted mid-session, stray `jules/*` branches may be left behind.
- **Check for stray branches:** `git branch --list "jules/*"`
- **Cleanup Strategy:** If a branch exists but the corresponding Jules session is no longer marked as `Completed` (or was already merged), delete the branch safely:
  ```bash
  git checkout target_branch
  git branch -D jules/<session_id>
  ```

## 2. Lockfile & Mid-State Resolution
An interrupted agent might leave the repository in the middle of a merge, rebase, or with uncommitted files.
Before starting any new session processing, the agent MUST execute these safety checks:
- **Abort stale rebases:**
  ```bash
  if [ -d ".git/rebase-merge" ] || [ -d ".git/rebase-apply" ]; then
    git rebase --abort
  fi
  ```
- **Abort stale merges:**
  ```bash
  if [ -f ".git/MERGE_HEAD" ]; then
    git merge --abort
  fi
  ```
- **Clean the working tree:**
  If uncommitted changes exist that are not part of the active step, stash or discard them.
  ```bash
  git reset --hard
  git clean -fd
  ```

## 3. State Telemetry
To prevent catastrophic context loss, the agent should maintain a localized state.
- **Write State:** Before pulling a patch, write the current session ID to `.jules-merger.state`.
- **Read State:** On initialization, check if `.jules-merger.state` exists. If it does, and the session is still pending, resume the pipeline for that session.
- **Clear State:** Upon successful PR creation or merge, delete `.jules-merger.state`.
