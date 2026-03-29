# Jules Session Merger Orchestrator

A specialized AI Agent skill to autonomously process, pull, publish, and merge completed Google Jules sessions.

## 🚀 The Problem: The Jules Backlog
Google's asynchronous coding agent, Jules, is incredibly powerful for automating background tasks, refactoring, and adding tests. However, as Jules completes multiple parallel sessions across various repositories, the backlog of completed tasks waiting for human review can quickly pile up. 

Manually navigating to the Jules UI, clicking through each session, publishing the Pull Request, and then merging it into your local/remote codebase takes valuable time and context-switching.

## 💡 The Solution
The **jules-merger** skill acts as a robotic orchestrator for your Jules sessions. When invoked, your local AI assistant will run a resilient pipeline to integrate the work:
1. **Identify:** Find all `Completed` Jules sessions via the CLI.
2. **Retrieve:** Automatically locate the associated local repository, create a clean branch, and pull the patch.
3. **Review & Audit:** Review the uncommitted diff for security flaws, logic errors, or style violations, attempting to fix minor issues locally before committing.
4. **Verify & Auto-Fix:** Run the project's test suite. If tests fail, the agent will attempt to diagnose and fix the broken code or test locally. 
5. **Publish:** Publish a Pull Request to GitHub. If unresolved issues remain, the PR is marked as a Draft or flagged for human review.
6. **Merge:** If the tests pass and the review is clean, the approved changes are merged directly into your `main` branch.

## 📦 Installation

This skill is compatible with standard Agent `.skills` directories (e.g., Cline, Cursor, Gemini CLI, Copilot, etc.).

```bash
npx skills add clindsay94/jules-merger-skill
```

## 🛠️ Usage

To invoke the skill, simply ask your AI agent:

> "Use the jules-merger skill to process all completed Jules sessions."
> 
> "Run jules-merger on my SystemSage repo."

The agent will take over, read the session states, and batch process the integrations autonomously, providing a final report of what was merged, what needs human review, and what was skipped.

## 🤖 The Resilient Pipeline

The core of this skill is its ability to handle edge cases without immediately giving up and dropping the workload back on your plate:

* **Conflict Resolution:** If a patch fails to apply because the baseline code has changed, the agent will attempt to manually apply the intended changes using its own context-aware editing tools. If the conflict is too complex, it will automatically submit a *new* Jules task with the updated file context.
* **Test Auto-Fixing:** Throwing away a branch just because a test fails is wasteful. The agent will read the test failure output and attempt to fix the implementation or the test itself. It will only escalate to you if the fix requires significant architectural changes.
* **Intelligent Escalation:** The agent knows the difference between a clean patch and a risky one. Clean patches are merged silently. Risky or broken patches are staged, pushed, and presented to you as Draft Pull Requests with detailed notes on what went wrong.
