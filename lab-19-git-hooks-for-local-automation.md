# Lab 19: Git Hooks for Local Automation

__Objective__: To practise creating and using Git hooks to automate tasks and enforce standards in your local workflow

## Prerequisites

- Completion of Labs 1-18 or equivalent Git knowledge
- Understanding of shell scripting basics
- Git installed on your machine
- A terminal or command prompt
- Text editor for creating hook scripts

## Important Note

Git hooks are local scripts that run automatically at specific points in your Git workflow. They are stored in `.git/hooks/` and are not tracked by Git (not pushed to remotes). This lab teaches you to create, test, and share hooks with your team.

## Step-by-Step Exercise

### Part 1: Understanding Hooks

1. **Explore the Hooks Directory**

    Discover where hooks live and examine sample hooks. Use `ls -la .git/hooks/` to see the hook directory.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir git-hooks-lab
    cd git-hooks-lab
    git init

    # Explore hooks directory
    ls -la .git/hooks/

    # Shows sample hooks:
    # pre-commit.sample, commit-msg.sample, pre-push.sample, etc.

    # View a sample hook
    cat .git/hooks/pre-commit.sample

    # Note: Sample hooks have .sample extension and won't execute
    # Remove .sample and make executable to activate
    ```

    **Location:** `.git/hooks/` - Not tracked by Git by default.
    </details>

2. **Create Your First Pre-Commit Hook**

    Build a hook that prevents committing console.log statements. Create `.git/hooks/pre-commit` and make it executable with `chmod +x`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create pre-commit hook
    cat > .git/hooks/pre-commit << 'EOF'
#!/bin/sh

echo "Running pre-commit checks..."

# Check for console.log statements
if git diff --cached | grep -E '^\+.*console\.log'; then
    echo "Error: console.log() found in staged files"
    echo "Please remove console.log statements before committing"
    exit 1
fi

echo "Pre-commit checks passed"
exit 0
EOF

    # Make it executable
    chmod +x .git/hooks/pre-commit

    # Test the hook
    mkdir src
    echo "function hello() { console.log('test'); }" > src/app.js

    git add src/app.js
    git commit -m "Add app.js"
    # Should fail with error about console.log

    # Fix the code
    echo "function hello() { return 'Hello'; }" > src/app.js

    git add src/app.js
    git commit -m "Add app.js"
    # Should succeed!

    git log --oneline -1
    ```

    **Result:** Hook prevents commits with console.log statements.
    </details>

### Part 2: Common Hook Types

3. **Create Commit-Msg Hook**

    Enforce commit message standards. Create `.git/hooks/commit-msg` that validates message length and format.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create commit-msg hook
    cat > .git/hooks/commit-msg << 'EOF'
#!/bin/sh

commit_msg_file=$1
commit_msg=$(cat "$commit_msg_file")

echo "Validating commit message..."

# Check minimum length
if [ ${#commit_msg} -lt 10 ]; then
    echo "Error: Commit message too short (minimum 10 characters)"
    echo "Current length: ${#commit_msg}"
    exit 1
fi

# Check for capital first letter
first_char=$(echo "$commit_msg" | cut -c1)
if ! echo "$first_char" | grep -q '[A-Z]'; then
    echo "Error: Commit message must start with capital letter"
    exit 1
fi

echo "Commit message validation passed"
exit 0
EOF

    chmod +x .git/hooks/commit-msg

    # Test various commit messages
    echo "test" >> src/app.js
    git add src/app.js

    # Test too short
    git commit -m "fix"
    # Should fail

    # Test no capital
    git commit -m "fixed the bug"
    # Should fail

    # Test good message
    git commit -m "Add error handling to app.js"
    # Should succeed

    git log --oneline -2
    ```

    **Standard:** Enforces commit message quality.
    </details>

4. **Create Pre-Push Hook**

    Run tests before pushing code. Create `.git/hooks/pre-push` that executes a test script.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # First, create a simple test file
    mkdir tests
    cat > tests/test.sh << 'EOF'
#!/bin/sh

echo "Running tests..."

# Simple test: check that src/app.js exists
if [ ! -f src/app.js ]; then
    echo "Test failed: src/app.js not found"
    exit 1
fi

# Simple test: check no console.log in src/app.js
if grep -q 'console\.log' src/app.js; then
    echo "Test failed: console.log found in src/app.js"
    exit 1
fi

echo "All tests passed"
exit 0
EOF

    chmod +x tests/test.sh

    # Create pre-push hook
    cat > .git/hooks/pre-push << 'EOF'
#!/bin/sh

echo "Running pre-push checks..."

# Run tests
if ! ./tests/test.sh; then
    echo "Tests failed - push aborted"
    exit 1
fi

echo "Pre-push checks passed"
exit 0
EOF

    chmod +x .git/hooks/pre-push

    # Test the hook (simulate push)
    # Note: Needs a remote to actually push
    echo "Pre-push hook created and ready to test"
    echo "It will run tests before any push operation"
    ```

    **Safety:** Tests run automatically before code is pushed.
    </details>

### Part 3: Sharing Hooks

5. **Share Hooks with Team**

    Make hooks available to all team members by storing them in a tracked directory.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Hooks in .git/hooks/ are not tracked by Git
    # Solution: Store hooks in repository and configure Git to use them

    # Create hooks directory in repository
    mkdir -p .githooks

    # Copy hooks to tracked directory
    cp .git/hooks/pre-commit .githooks/pre-commit
    cp .git/hooks/commit-msg .githooks/commit-msg
    cp .git/hooks/pre-push .githooks/pre-push

    # Make them executable
    chmod +x .githooks/*

    # Configure Git to use custom hooks directory
    # (Requires Git 2.9+)
    git config core.hooksPath .githooks

    # Create README for hooks
    cat > .githooks/README.md << 'EOF'
# Git Hooks

This directory contains Git hooks for the project.

## Installation

Configure Git to use these hooks:

```bash
git config core.hooksPath .githooks
```

## Available Hooks

### pre-commit
Runs before commits are created:
- Checks for console.log statements

### commit-msg
Validates commit messages:
- Minimum length (10 characters)
- Capital first letter

### pre-push
Runs before pushing to remote:
- Runs test suite

## Bypassing Hooks

Not recommended, but possible:

```bash
git commit --no-verify
git push --no-verify
```
EOF

    # Add to repository
    git add .githooks/
    git commit -m "Add shared Git hooks

Team members can install with:
  git config core.hooksPath .githooks"

    cat .githooks/README.md
    ```

    **Team workflow:** Hooks now shared and versioned with repository.
    </details>

6. **Create Configurable Hooks**

    Make hooks configurable for different team needs using a configuration file.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create hook configuration file
    cat > .githooks/config.sh << 'EOF'
#!/bin/bash

# Hook Configuration
# Customize behaviour of Git hooks

# Pre-commit settings
export CHECK_DEBUG_CODE=true
export CHECK_WHITESPACE=true
export MAX_FILE_SIZE=1048576  # 1MB in bytes

# Commit message settings
export MIN_COMMIT_LENGTH=10

# Pre-push settings
export RUN_TESTS_ON_PUSH=true
EOF

    # Create configurable pre-commit hook
    cat > .githooks/pre-commit << 'EOF'
#!/bin/bash

# Load configuration
if [ -f .githooks/config.sh ]; then
    source .githooks/config.sh
fi

echo "Running pre-commit checks..."

exit_code=0

# Check debug code
if [ "$CHECK_DEBUG_CODE" = true ]; then
    echo "  Checking for debug code..."
    if git diff --cached | grep -E '^\+.*(console\.log|debugger)'; then
        echo "    Error: Debug code found"
        exit_code=1
    fi
fi

# Check whitespace
if [ "$CHECK_WHITESPACE" = true ]; then
    echo "  Checking whitespace..."
    if ! git diff --cached --check > /dev/null 2>&1; then
        echo "    Error: Whitespace issues found"
        exit_code=1
    fi
fi

if [ $exit_code -eq 0 ]; then
    echo "Pre-commit checks passed"
fi

exit $exit_code
EOF

    chmod +x .githooks/pre-commit

    # Create local configuration example
    cat > .githooks/config.local.sh.example << 'EOF'
#!/bin/bash

# Local Hook Configuration
# Copy to .githooks/config.local.sh and customise

# Override settings for your local environment
# export CHECK_DEBUG_CODE=false
# export RUN_TESTS_ON_PUSH=false
EOF

    # Add to gitignore
    echo ".githooks/config.local.sh" >> .gitignore

    git add .githooks/config.sh .githooks/config.local.sh.example .gitignore
    git commit -m "Add configurable Git hooks"

    cat .githooks/config.sh
    ```

    **Flexibility:** Team can customise hooks without editing scripts.
    </details>

### Part 4: Advanced Hooks

7. **Create Prepare-Commit-Msg Hook**

    Automatically enhance commit messages with branch names. Create `.githooks/prepare-commit-msg`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create prepare-commit-msg hook
    cat > .githooks/prepare-commit-msg << 'EOF'
#!/bin/sh

commit_msg_file=$1
commit_source=$2

# Only modify message for regular commits
if [ "$commit_source" != "message" ] && [ -n "$commit_source" ]; then
    exit 0
fi

# Get current branch name
branch=$(git symbolic-ref --short HEAD)

# Skip for main/master branches
if [ "$branch" = "main" ] || [ "$branch" = "master" ]; then
    exit 0
fi

# Add branch name to commit message
current_msg=$(cat "$commit_msg_file")

# Check if branch name already in message
if ! echo "$current_msg" | grep -q "\[$branch\]"; then
    # Prepend branch name
    echo "[$branch] $current_msg" > "$commit_msg_file"
fi

exit 0
EOF

    chmod +x .githooks/prepare-commit-msg

    # Test the hook
    git checkout -b feature/user-auth

    echo "function auth() { return true; }" > src/auth.js
    git add src/auth.js
    git commit -m "Add authentication"

    # View commit message
    git log -1 --pretty=%B
    # Should show: "[feature/user-auth] Add authentication"

    git checkout main
    ```

    **Automation:** Branch name automatically added to commits.
    </details>

8. **Test Your Hooks**

    Create a testing framework for hooks to ensure they work correctly.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir -p .githooks/tests

    cat > .githooks/tests/test-pre-commit.sh << 'EOF'
#!/bin/bash

echo "Testing pre-commit hook..."

# Setup test repository
test_dir=$(mktemp -d)
cd "$test_dir"
git init > /dev/null 2>&1

# Copy hook
cp "$OLDPWD/.githooks/pre-commit" .git/hooks/
cp "$OLDPWD/.githooks/config.sh" ./ 2>/dev/null || true

# Test 1: Debug code should be blocked
echo "Test 1: Block debug code"
echo "console.log('test');" > test.js
git add test.js
if git commit -m "Test" 2>&1 | grep -q "Debug code"; then
    echo "  Pass: Correctly blocked debug code"
else
    echo "  Fail: Failed to block debug code"
fi

# Test 2: Clean code should pass
echo "Test 2: Allow clean code"
echo "// Clean code" > test.js
git add test.js
if git commit -m "Clean commit" > /dev/null 2>&1; then
    echo "  Pass: Clean code passed"
else
    echo "  Fail: Clean code was blocked"
fi

# Cleanup
cd "$OLDPWD"
rm -rf "$test_dir"

echo "Pre-commit hook tests complete"
EOF

    chmod +x .githooks/tests/test-pre-commit.sh

    # Run tests
    ./.githooks/tests/test-pre-commit.sh

    git add .githooks/tests/
    git commit -m "Add Git hooks testing framework"
    ```

    **Quality assurance:** Hooks are tested before deployment.
    </details>

## Challenge Exercises

### Challenge 1: Complete Hook Suite

Create a comprehensive set of hooks for a professional project:
- pre-commit: Linting, formatting, secrets detection
- commit-msg: Enforce conventional commits format
- pre-push: Full test suite
- All hooks should be configurable

<details>
<summary>Hint</summary>

Combine multiple checks in each hook, use configuration files for flexibility, add clear error messages, make hooks fast by only checking changed files, and provide documentation for team onboarding.
</details>

### Challenge 2: Hook Performance Optimisation

Create hooks that are fast even with large repositories:
1. Only check staged/changed files
2. Run checks in parallel where possible
3. Provide progress indicators
4. Allow skipping slow checks with configuration

<details>
<summary>Hint</summary>

Use `git diff --cached --name-only` to get only staged files, run independent checks with `&` for parallelisation, and show progress with echo statements.
</details>

## Reflection Questions

After completing this lab, consider:

1. What tasks are good candidates for Git hooks?
2. How do you balance automation with developer freedom?
3. When should hooks be warnings versus blocking errors?
4. How do you ensure hooks don't slow down development?
5. What's the difference between local hooks and CI checks?

## Clean Up

If you want to remove the lab directory:

```bash
cd ..
rm -rf git-hooks-lab
```

## Key Commands Reference

```bash
# Hook location
.git/hooks/

# Make hook executable
chmod +x .git/hooks/pre-commit

# Configure hooks directory
git config core.hooksPath .githooks

# Bypass hooks
git commit --no-verify
git push --no-verify

# Test hook manually
.git/hooks/pre-commit

# Available hooks
# pre-commit, prepare-commit-msg, commit-msg
# post-commit, pre-push
```

## What You've Learned

- Understanding Git hooks and their purposes
- Creating pre-commit hooks to enforce code quality
- Building commit-msg hooks to validate commit messages
- Implementing pre-push hooks to run tests before pushing
- Using prepare-commit-msg hooks to enhance messages automatically
- Sharing hooks with team members using tracked directories
- Making hooks configurable for different environments
- Testing hooks to ensure correct functionality
- Balancing automation with developer productivity
- Best practices for hook development and maintenance

Git hooks enable powerful local automation whilst maintaining developer flexibility and speed!
