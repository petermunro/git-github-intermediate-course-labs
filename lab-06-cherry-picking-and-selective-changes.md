# Lab 6: Cherry-Picking and Selective Changes

__Objective__: To understand and practise cherry-picking commits for selective integration

## Prerequisites

- Completion of Labs 1-5 or equivalent Git knowledge
- Understanding of branching and merging
- Git installed on your machine
- A terminal or command prompt

## Step-by-Step Exercise

### Part 1: Basic Cherry-Picking

1. **Create a Test Repository**

    Set up a repository with a main branch. Use `git init` to create it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir cherry-pick-lab
    cd cherry-pick-lab
    git init

    # Create initial commit
    echo "# Cherry Pick Lab" > README.md
    git add README.md
    git commit -m "Initial commit"
    ```
    </details>

2. **Create a Feature Branch with Multiple Commits**

    Create a feature branch and make several distinct commits on it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create feature branch
    git checkout -b feature-development

    # Commit 1: Add authentication
    echo "function authenticate() { return true; }" > auth.js
    git add auth.js
    git commit -m "Add authentication function"

    # Commit 2: Add validation
    echo "function validate(data) { return data !== null; }" > validation.js
    git add validation.js
    git commit -m "Add validation function"

    # Commit 3: Add bug fix
    echo "function authenticate(user) { return user !== null; }" > auth.js
    git add auth.js
    git commit -m "Fix authentication bug"

    # View the commits
    git log --oneline
    ```
    </details>

3. **Cherry-Pick a Single Commit**

    Select one specific commit to apply to main using `git cherry-pick`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Switch to main
    git checkout main

    # View commits on feature branch
    git log --oneline feature-development

    # Cherry-pick the bug fix commit (use the actual hash)
    git cherry-pick <hash-of-bug-fix>

    # Verify
    git log --oneline
    ls  # Should show auth.js but not validation.js
    ```

    Main now has the bug fix but not the other commits!
    </details>

4. **Cherry-Pick with the -x Flag**

    Use `-x` to record where the commit came from.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # View commits
    git log --oneline feature-development

    # Cherry-pick validation commit with -x
    git cherry-pick -x <hash-of-validation>

    # View the commit message
    git log -1
    ```

    The `-x` flag adds a "(cherry picked from commit ...)" note.
    </details>

### Part 2: Advanced Cherry-Picking

5. **Cherry-Pick Multiple Commits**

    Select several commits to apply at once.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create more commits on feature branch
    git checkout feature-development
    echo "function log(msg) { console.log(msg); }" > logger.js
    git add logger.js
    git commit -m "Add logging function"

    # Return to main
    git checkout main

    # Cherry-pick multiple commits (use actual hashes)
    git cherry-pick <hash1> <hash2>

    # View result
    git log --oneline
    ```
    </details>

6. **Handle Cherry-Pick Conflicts**

    Resolve conflicts that occur during cherry-picking.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create conflicting change on main
    echo "function authenticate(user, password) { return true; }" > auth.js
    git add auth.js
    git commit -m "Update authentication signature"

    # Try to cherry-pick a commit that modifies auth.js
    git cherry-pick <hash-that-changes-auth>

    # Conflict occurs - resolve it
    echo "function authenticate(user, password) { return user !== null; }" > auth.js

    # Mark resolved and continue
    git add auth.js
    git cherry-pick --continue
    ```
    </details>

7. **Cherry-Pick Without Committing**

    Use `-n` to apply changes without creating a commit.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Cherry-pick without committing
    git cherry-pick -n <commit-hash>

    # Changes are staged but not committed
    git status

    # You can modify before committing
    echo "// Additional changes" >> validation.js
    git add validation.js
    git commit -m "Add validation with enhancements"
    ```

    **Use case:** Combine multiple cherry-picks into one commit.
    </details>

### Part 3: Practical Scenarios

8. **Abort a Cherry-Pick**

    Practise cancelling a cherry-pick operation using `git cherry-pick --abort`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Start a cherry-pick that conflicts
    git cherry-pick <conflicting-hash>

    # Check status
    git status

    # Decide to abort
    git cherry-pick --abort

    # Verify clean state
    git status
    ```
    </details>

9. **Simulate a Hotfix Workflow**

    Apply a critical fix to multiple release branches.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create release branches
    git checkout -b release-1.0 main
    echo "Release 1.0" > release-notes.txt
    git add release-notes.txt
    git commit -m "Release 1.0"

    git checkout main
    git checkout -b release-2.0
    echo "Release 2.0" > release-notes.txt
    git add release-notes.txt
    git commit -m "Release 2.0"

    # Create critical fix on main
    git checkout main
    echo "function fixCriticalBug() { /* fix */ }" > bugfix.js
    git add bugfix.js
    git commit -m "Fix critical security bug"

    # Cherry-pick to both releases
    git checkout release-1.0
    git cherry-pick -x main

    git checkout release-2.0
    git cherry-pick -x main

    # Verify fix is on all branches
    git log --oneline --all --graph
    ```

    **Real-world scenario:** Applying hotfixes to multiple versions.
    </details>

## Challenge Exercises

### Challenge 1: Selective Integration

You have a feature branch with 10 commits. Cherry-pick only commits 3, 5, and 7 to main.

<details>
<summary>Hint</summary>

Create a branch with 10 commits, use `git log --oneline` to find the hashes, then `git cherry-pick <hash3> <hash5> <hash7>`.
</details>

### Challenge 2: Combining Cherry-Picks

Cherry-pick three separate commits without committing each one (`-n` flag), then create a single combined commit.

<details>
<summary>Hint</summary>

Use `git cherry-pick -n` for each commit, then `git commit` once with a message explaining all three changes.
</details>

## Reflection Questions

1. When would you cherry-pick instead of merging an entire branch?
2. Why is the `-x` flag important for tracking?
3. What problems can arise from cherry-picking too frequently?
4. How does cherry-picking differ from rebasing?

## What You've Learned

- How to cherry-pick individual commits across branches
- Using the `-x` flag for traceability
- Cherry-picking multiple commits at once
- Resolving conflicts during cherry-pick operations
- Cherry-picking without committing for combined changes
- Aborting cherry-pick operations when needed
- Applying hotfixes to multiple release branches
- When cherry-picking is appropriate versus merging

## Clean Up

```bash
cd ..
rm -rf cherry-pick-lab
```
