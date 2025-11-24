# Lab 13: Git Worktrees for Parallel Development

__Objective__: To practise using Git worktrees for working on multiple branches simultaneously without context switching

## Prerequisites

- Completion of Labs 1-12 or equivalent Git knowledge
- Understanding of branching and working directories
- Git installed on your machine
- A terminal or command prompt

## About This Lab

In Module 13, you learnt about Git worktrees - a powerful feature that lets you have multiple branches checked out simultaneously in separate directories. Key concepts:

- **`git worktree add <path> <branch>`** - Create a worktree
- **`git worktree list`** - View all worktrees
- **`git worktree remove <path>`** - Delete a worktree
- **Shared repository** - All worktrees share the same `.git` database

Worktrees are useful when you need to:
- Review a pull request whilst working on a feature
- Run tests on one branch whilst developing another
- Compare different branches side by side
- Handle urgent fixes without stashing

## Step-by-Step Exercise

### Part 1: Creating and Using Worktrees

1. **Create the Main Repository**

    Create a new repository called `worktree-lab` and make an initial commit.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir worktree-lab
    cd worktree-lab
    git init

    # Create initial files
    echo "# Project" > README.md
    echo "v1.0.0" > version.txt
    git add .
    git commit -m "Initial commit"
    ```
    </details>

2. **Create Your First Worktree**

    Create a worktree for a hotfix in a separate directory using `git worktree add ../worktree-hotfix main`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create worktree for hotfix
    git worktree add ../worktree-hotfix main

    # List all worktrees
    git worktree list

    # Navigate to the new worktree
    cd ../worktree-hotfix
    pwd
    git branch

    # Make a hotfix
    echo "v1.0.1" > version.txt
    git add version.txt
    git commit -m "Hotfix: Update version"

    # Return to main worktree
    cd ../worktree-lab
    git log --oneline
    ```

    The hotfix commit is visible in the main repository - they share the same Git database!
    </details>

3. **Create Worktree with New Branch**

    Create a worktree and a new feature branch in one command using `git worktree add -b <branch> <path>`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create worktree with new branch
    git worktree add -b feature/auth ../worktree-auth

    # Check what was created
    git worktree list

    # Work in the new worktree
    cd ../worktree-auth
    git branch  # On feature/auth

    # Add authentication module
    echo "function login() { }" > auth.js
    git add auth.js
    git commit -m "Add authentication module"

    cd ../worktree-lab
    ```

    The `-b` flag creates the branch and worktree together.
    </details>

### Part 2: Parallel Development Workflow

4. **Work on Multiple Features Simultaneously**

    Create two feature worktrees and work on both without switching contexts.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create two feature worktrees
    git worktree add -b feature/dashboard ../worktree-dashboard
    git worktree add -b feature/reports ../worktree-reports

    # View all worktrees
    git worktree list

    # Work on dashboard
    cd ../worktree-dashboard
    echo "function renderDashboard() { }" > dashboard.js
    git add dashboard.js
    git commit -m "Add dashboard"

    # Work on reports (no context switch needed!)
    cd ../worktree-reports
    echo "function generateReport() { }" > reports.js
    git add reports.js
    git commit -m "Add reports"

    # Back to main
    cd ../worktree-lab
    ```

    Both features developed in parallel without stashing or branch switching.
    </details>

5. **Review Pull Request Whilst Developing**

    Simulate reviewing someone's code whilst your feature work remains untouched.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create a PR branch to review
    git checkout -b review/new-feature
    echo "function newFeature() { }" > feature.js
    git add feature.js
    git commit -m "Colleague's new feature"
    git checkout main

    # Create worktree for review
    git worktree add ../worktree-review review/new-feature

    # Review in separate directory
    cd ../worktree-review
    cat feature.js
    # Run tests, check code quality, etc.

    # Meanwhile, your work in other worktrees is untouched
    cd ../worktree-dashboard
    cat dashboard.js  # Your work is still here

    cd ../worktree-lab
    ```

    Perfect for code reviews without disrupting your current work.
    </details>

6. **Compare Branches Side by Side**

    Use worktrees to compare different versions running simultaneously.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create worktrees for comparison
    git worktree add ../worktree-main-test main
    git worktree add ../worktree-dashboard-test feature/dashboard

    # Compare files
    echo "=== Main version ==="
    cat ../worktree-main-test/version.txt

    echo "=== Dashboard version ==="
    ls ../worktree-dashboard-test/

    # In real scenarios, you could run both versions:
    # Terminal 1: cd ../worktree-main-test && npm start
    # Terminal 2: cd ../worktree-dashboard-test && npm start -p 3001

    # Clean up test worktrees
    git worktree remove ../worktree-main-test
    git worktree remove ../worktree-dashboard-test
    ```
    </details>

### Part 3: Managing Worktrees

7. **Remove Worktrees Properly**

    Clean up worktrees when finished using `git worktree remove`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # List current worktrees
    git worktree list

    # Remove the review worktree
    git worktree remove ../worktree-review

    # Verify it's gone
    git worktree list
    ls ../ | grep worktree
    ```

    The directory is deleted and removed from Git's worktree list.
    </details>

8. **Handle Worktree Limitations**

    Understand that you can't check out the same branch in multiple worktrees.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Try to create another worktree on main
    git worktree add ../worktree-main-again main
    # Error: 'main' is already checked out

    # Solution: Create a new branch from main
    git worktree add -b main-test ../worktree-main-test main

    # Or use a different existing branch
    git worktree list

    # Clean up
    git worktree remove ../worktree-main-test
    git branch -d main-test
    ```

    Each branch can only be checked out in one worktree at a time.
    </details>

9. **Implement Hotfix Workflow with Worktrees**

    Use worktrees to handle urgent fixes without stashing.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # You're working on dashboard (with uncommitted changes)
    cd ../worktree-dashboard
    echo "// Work in progress" >> dashboard.js
    # Don't commit yet

    # Urgent bug report! No need to stash
    cd ../worktree-lab

    # Create hotfix worktree
    git worktree add -b hotfix/security ../worktree-hotfix main

    cd ../worktree-hotfix
    echo "function sanitizeInput() { }" > security.js
    git add security.js
    git commit -m "Fix XSS vulnerability"

    # Merge to main
    git checkout main
    git merge hotfix/security --no-ff

    # Return to dashboard work
    cd ../worktree-dashboard
    cat dashboard.js  # Your uncommitted work is still here!

    # Clean up
    cd ../worktree-lab
    git worktree remove ../worktree-hotfix
    git branch -d hotfix/security
    ```

    No stashing required - both workspaces stay active.
    </details>

## Challenge Exercises

### Challenge 1: Multi-Branch Development

Set up a realistic development scenario:
1. Create worktrees for: main, develop, and two feature branches
2. Make commits in each worktree
3. Merge features to develop
4. Clean up completed worktrees

<details>
<summary>Hint</summary>

Use `git worktree add -b <branch> <path> <start-point>` to create branches from specific starting points. Keep track of all worktrees with `git worktree list`.
</details>

### Challenge 2: Testing Multiple Versions

Create worktrees for:
1. Production (main branch)
2. Staging (develop branch)
3. Feature under development

Simulate running all three simultaneously for testing.

<details>
<summary>Hint</summary>

Create three worktrees with different branch names and paths. In a real scenario, you'd run build/test commands in each directory simultaneously in separate terminals.
</details>

## Reflection Questions

After completing this lab, consider:

1. **Worktrees vs Stashing:** When are worktrees better than stashing?
2. **Disk Space:** What are the disk space implications of multiple worktrees?
3. **Collaboration:** How do worktrees compare to cloning the repository multiple times?
4. **Use Cases:** What are the best use cases for worktrees in your workflow?

## Clean Up

If you want to remove all lab directories:

```bash
cd ..
rm -rf worktree-*
```

## Key Commands Summary

```bash
# Create worktrees
git worktree add <path> <branch>           # Create from existing branch
git worktree add -b <new-branch> <path>    # Create with new branch

# List and inspect
git worktree list                          # List all worktrees
git worktree list --porcelain              # Detailed output

# Remove worktrees
git worktree remove <path>                 # Remove worktree
git worktree remove --force <path>         # Force remove

# Maintenance
git worktree prune                         # Clean stale references
```

## What You've Learnt

- Creating worktrees for parallel development
- Working on multiple branches simultaneously
- Creating worktrees with new branches
- Reviewing code without context switching
- Comparing different branches side by side
- Removing worktrees properly
- Understanding worktree limitations
- Implementing hotfix workflows without stashing
- Managing multiple active worktrees
- When worktrees are better than cloning

Worktrees enable true parallel development without the overhead of context switching!
