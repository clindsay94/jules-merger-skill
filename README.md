# Jules Session Merger Orchestrator

A specialized AI Agent skill to autonomously process, pull, publish, and merge completed Google Jules sessions.

## 🚀 The Problem: The Jules Backlog
Google's asynchronous coding agent, Jules, is incredibly powerful for automating background tasks, refactoring, and adding tests. However, as Jules completes multiple parallel sessions across various repositories, the backlog of completed tasks waiting for human review can quickly pile up. 

Manually navigating to the Jules UI, clicking through each session, publishing the Pull Request, and then merging it into your local/remote codebase takes valuable time and context-switching.

## 💡 The Solution
The **jules-merger** skill acts as a robotic orchestrator for your Jules sessions. When invoked, your local AI assistant will:
1. Identify all `Completed` Jules sessions via the CLI.
2. Automatically locate the associated local repository.
3. Create a clean branch, pull the patch, and commit the changes.
4. Publish a Pull Request to GitHub.
5. Merge the approved changes directly into your `main` branch.

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

The agent will take over, read the session states, and batch process the integrations autonomously.

## 🤖 Advanced Evaluation Workflows 

If you want to ensure the code Jules wrote is strictly evaluated before merging, consider chaining this skill with additional workflows:

1. **Pre-Merge Test Verification:** The agent can be instructed to run local CI/CD steps (e.g., `dotnet test`, `pytest`) after applying the patch. If the test suite fails, it automatically discards the branch and skips the merge.
2. **Automated Code Review:** Chain this skill with your `code-review` or `security-auditor` skills to have your local agent perform an independent security and best-practices audit on the Jules patch *before* publishing the PR.
3. **Conflict Resolution Delegation:** If a patch fails to apply cleanly due to an outdated baseline, the agent can be instructed to automatically re-prompt Jules with the latest `main` branch state to try again.
