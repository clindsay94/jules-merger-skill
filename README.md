# Jules Session Merger Orchestrator

A specialized AI Agent skill to autonomously process, pull, publish, and merge completed Google Jules sessions.

## 🚀 The Problem: The Jules Backlog
Google's asynchronous coding agent, Jules, is incredibly powerful for automating background tasks, refactoring, and adding tests. However, as Jules completes multiple parallel sessions across various repositories, the backlog of completed tasks waiting for human review can quickly pile up. 

Manually navigating to the Jules UI, clicking through each session, publishing the Pull Request, and then merging it into your local/remote codebase takes valuable time and context-switching.

## 💡 The Solution
The **jules-merger** skill acts as a robotic orchestrator for your Jules sessions. When invoked, your local AI assistant will run a resilient pipeline to integrate the work:
1. **Pre-Flight & Configure:** Checks for a clean working tree, attempts recovery from interrupted states, and loads repository-specific configurations (`.jules-merger.yml`).
2. **Identify:** Finds all `Completed` Jules sessions via the CLI.
3. **Retrieve:** Automatically locates the associated local repository, checks for an existing remote branch (`origin/jules/<session_id>`), and checks it out. If no remote branch exists, it falls back to a fresh branch and a manual `jules remote pull --apply`.
4. **Review & Audit:** Reviews the code changes introduced by Jules for security flaws, logic errors, or style violations against the main branch, running static analysis tools if configured, and attempts to fix minor issues locally.
5. **Verify & Auto-Fix:** Ensures the environment is ready (installing dependencies if needed) and runs the project's explicit test suite. If tests fail, the agent will attempt to diagnose and fix the broken code or test locally up to a configured threshold. 
6. **Publish:** Publishes a Pull Request to GitHub using Conventional Commits. If unresolved issues remain, the PR is marked as a Draft or flagged for human review.
7. **Merge:** If the tests pass and the review is clean, the approved changes are merged directly into your target branch.

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

## ⚙️ Configuration (`.jules-merger.yml`)

The skill relies on repository-specific configuration to avoid guessing critical commands. Create a `.jules-merger.yml` file in the root of your target repositories (see `jules-merger.schema.json` for details):

```yaml
# Example: .jules-merger.yml
target_branch: "main"
test_command: "dotnet test src/RemEx.Tests/RemEx.Tests.csproj"
lint_command: "dotnet format"
auto_fix_attempts: 3
require_human_review_for:
  - "Security"
  - "Architecture"
```

## 🤖 The Resilient Pipeline

The core of this skill is its ability to handle edge cases without immediately giving up and dropping the workload back on your plate:

* **State Recovery:** Consults `RECOVERY.md` to safely prune stray branches, abort broken rebases, and clean the working tree if the agent was previously interrupted mid-run.
* **Conflict Resolution:** The agent uses a multi-layered approach to conflicts. It first attempts to merge/rebase the existing Jules branch. If the branch doesn't exist or patching fails, it reads the raw diff and uses its own context-aware editing tools to surgically apply the changes, ignoring "corrupt" sections that refer to missing files. If the conflict is too complex, it will automatically submit a *new* Jules task with the updated file context.
* **Environment Management:** Automatically detects `requirements.txt`, `package.json`, or `pyproject.toml` and ensures the agent's local environment is prepared (e.g., `pip install -r requirements.txt`) before running tests.
* **Test Auto-Fixing:** Throwing away a branch just because a test fails is wasteful. The agent will read the test failure output and attempt to fix the implementation or the test itself. It will only escalate to you if the fix requires significant architectural changes.
* **Intelligent Escalation:** The agent knows the difference between a clean patch and a risky one. Clean patches are merged silently. Risky or broken patches are staged, pushed, and presented to you as Draft Pull Requests with detailed notes on what went wrong.

## 🧪 Testing & Dry-Runs

Because this agent executes arbitrary git commands and code modifications, a safe testing protocol is provided. See `TESTING.md` for instructions on using the `--dry-run` flag and setting up a mock sandbox repository.
