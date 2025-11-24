# Lab 7: Introduction to Rebase

__Objective__: To understand and practise rebasing for creating linear commit history

## Prerequisites

- Completion of Labs 1-6 or equivalent Git knowledge
- Understanding of branching and merging
- Git installed on your machine
- A terminal or command prompt

## Step-by-Step Exercise

### Part 1: Basic Rebasing

1. **Create a Test Repository**

    Set up a repository with initial commits on main. Use `git init` to create it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir rebase-lab
    cd rebase-lab
    git init

    # Create initial commits
    echo "# Rebase Lab" > README.md
    git add README.md
    git commit -m "Initial commit"

    echo "Project description" >> README.md
    git add README.md
    git commit -m "Add project description"
    ```
    </details>

2. **Create Divergent Branches**

    Make commits on both main and a feature branch so they diverge.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create feature branch
    git checkout -b feature-user-auth

    # Commits on feature branch
    echo "function login() { /* login */ }" > auth.js
    git add auth.js
    git commit -m "Add login function"

    echo "function logout() { /* logout */ }" >> auth.js
    git add auth.js
    git commit -m "Add logout function"

    # Switch to main and make commits
    git checkout main
    echo "## Installation" >> README.md
    git add README.md
    git commit -m "Add installation section"

    echo "## Usage" >> README.md
    git add README.md
    git commit -m "Add usage section"

    # View divergent history
    git log --oneline --graph --all
    ```

    Main and feature-user-auth have diverged.
    </details>

3. **Perform Your First Rebase**

    Rebase the feature branch onto main using `git rebase main`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Switch to feature branch
    git checkout feature-user-auth

    # Rebase onto main
    git rebase main

    # View the result
    git log --oneline --graph --all
    ```

    The feature commits now appear after main commits in a linear history!
    </details>

4. **Understand What Happened**

    Examine how rebase changed the commit hashes.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # View the reflog
    git reflog

    # View the graph
    git log --oneline --graph --all
    ```

    **Key insight:** Rebase created new commits with the same changes but different hashes!
    </details>

### Part 2: Handling Conflicts

5. **Create a Rebase Conflict**

    Set up a scenario where rebase requires conflict resolution.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create new feature branch
    git checkout main
    git checkout -b feature-config

    echo "timeout=30" > config.txt
    git add config.txt
    git commit -m "Set timeout to 30"

    # Create conflicting change on main
    git checkout main
    echo "timeout=60" > config.txt
    git add config.txt
    git commit -m "Set timeout to 60"

    # Try to rebase
    git checkout feature-config
    git rebase main

    # Conflict occurs
    cat config.txt
    ```
    </details>

6. **Resolve the Conflict**

    Fix the conflict and continue the rebase using `git rebase --continue`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Resolve conflict
    echo "timeout=45" > config.txt

    # Mark as resolved
    git add config.txt

    # Continue rebase
    git rebase --continue

    # View result
    git log --oneline --graph --all
    ```

    **Important:** Rebase may require resolving conflicts for each commit being replayed.
    </details>

7. **Abort a Rebase**

    Practise cancelling a rebase operation using `git rebase --abort`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create another conflict scenario
    git checkout main
    git checkout -b feature-test

    echo "test=true" > test.txt
    git add test.txt
    git commit -m "Enable tests"

    git checkout main
    echo "test=false" > test.txt
    git add test.txt
    git commit -m "Disable tests"

    # Start rebase
    git checkout feature-test
    git rebase main

    # Conflict occurs - abort it
    git rebase --abort

    # Verify we're back to original state
    git status
    ```
    </details>

### Part 3: Comparing Rebase and Merge

8. **Compare Rebase vs Merge**

    Create two scenarios to see the difference in history.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Scenario 1: Using merge
    git checkout main
    git checkout -b test-merge
    echo "merge content" > merge.txt
    git add merge.txt
    git commit -m "Merge branch content"

    git checkout main
    echo "main content" > main-file.txt
    git add main-file.txt
    git commit -m "Main branch content"

    # Merge
    git merge test-merge

    # View history
    git log --oneline --graph

    # Scenario 2: Using rebase
    git checkout -b test-rebase HEAD~2
    echo "rebase content" > rebase.txt
    git add rebase.txt
    git commit -m "Rebase branch content"

    # Rebase
    git rebase main

    # View history
    git log --oneline --graph test-rebase
    ```

    **Observation:** Merge creates a merge commit; rebase creates linear history.
    </details>

9. **Use Reflog to Recover**

    Practise using reflog to undo a rebase.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create a branch
    git checkout -b recovery-test main
    echo "original" > original.txt
    git add original.txt
    git commit -m "Original commit"

    # Note current commit
    git log --oneline -1

    # Make changes on main
    git checkout main
    echo "new content" > newfile.txt
    git add newfile.txt
    git commit -m "New main content"

    # Rebase recovery-test
    git checkout recovery-test
    git rebase main

    # Use reflog to undo
    git reflog
    git reset --hard HEAD@{1}

    # Verify recovery
    git log --oneline
    ```

    **The reflog is your safety net!**
    </details>

## Challenge Exercises

### Challenge 1: Multi-Commit Rebase

Create a feature branch with 5 commits. Advance main by 3 commits. Rebase the feature branch onto the updated main.

<details>
<summary>Hint</summary>

Each commit will be replayed in order on top of main. Use `git log --oneline --graph --all` to see the result.
</details>

### Challenge 2: Compare Histories

Create two identical branches. Merge one into main and rebase the other onto main. Use `git log --graph` to compare the resulting histories.

<details>
<summary>Hint</summary>

The merged branch will show a merge commit. The rebased branch will show a linear history.
</details>

## Reflection Questions

1. What are the main differences between merge and rebase?
2. Why is it dangerous to rebase commits that have been pushed to a shared branch?
3. When would you prefer rebasing over merging?
4. How does rebasing affect commit hashes, and why does this matter?

## What You've Learned

- The fundamental concept of rebasing commits
- How rebase creates a linear history
- The difference between rebase and merge
- Resolving conflicts during rebase operations
- Aborting a rebase when necessary
- Understanding that rebase creates new commits with new hashes
- Using reflog to recover from rebase mistakes
- The golden rule: never rebase public/shared commits

## Clean Up

```bash
cd ..
rm -rf rebase-lab
```
