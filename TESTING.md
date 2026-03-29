# Testing & Dry-Run Guide

Because the `jules-merger` skill executes git operations and creates remote Pull Requests, it is highly recommended to sandbox and test its behavior before unleashing it on a production repository.

## Dry-Run Mode

You can instruct the agent to perform a dry-run by appending `--dry-run` to your prompt.

**Example Prompt:**
> "Use the jules-merger skill to process all completed Jules sessions with the --dry-run flag."

**Agent Behavior in Dry-Run:**
1. The agent will read sessions, verify configurations, and prepare local branches.
2. It will **NOT** execute `git push`.
3. It will **NOT** use MCP tools to create actual GitHub Pull Requests.
4. Instead, it will generate a `dry-run-output.txt` file detailing the exact CLI commands and PR payloads it *would* have executed.

## Sandbox Testing Environment

To safely test the agent's logic without a real Jules session, use the following bash script to generate a mock repository and a fake patch file.

### 1. Create the Sandbox
Save the following as `setup-sandbox.sh` and run it:

```bash
#!/bin/bash
mkdir jules-sandbox
cd jules-sandbox
git init
echo "# Mock Repo" > README.md
git add README.md
git commit -m "Initial commit"
git checkout -b main

# Create a mock .jules-merger.yml
cat <<EOF > .jules-merger.yml
target_branch: "main"
test_command: "echo 'Tests passed'"
lint_command: "echo 'Linting passed'"
auto_fix_attempts: 1
EOF
git add .jules-merger.yml
git commit -m "Add config"

# Create a fake patch file to simulate `jules remote pull --apply`
cat <<EOF > fake_patch.diff
diff --git a/README.md b/README.md
index 3f95e5e..9582531 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,3 @@
 # Mock Repo
+
+This line was added by a mock Jules session.
EOF

echo "Sandbox created at ./jules-sandbox"
```

### 2. Instruct the Agent
Navigate to the `jules-sandbox` directory and tell the agent:

> "Use the jules-merger skill. Assume there is a completed Jules session ID '999999'. Skip the 'jules remote list' and 'jules remote pull' commands. Instead, apply 'fake_patch.diff' using 'git apply fake_patch.diff'. Run through the rest of the resilient pipeline in --dry-run mode."
