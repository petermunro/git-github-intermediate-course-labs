# Lab 9: Branch Management Strategies

__Objective__: To practise effective branch management, naming conventions, and maintenance workflows

## Prerequisites

- Completion of Labs 1-8 or equivalent Git knowledge
- Understanding of branching and merging
- Git installed on your machine
- A terminal or command prompt

## Step-by-Step Exercise

### Part 1: Branch Organisation

1. **Create a Test Repository**

    Set up a repository with long-lived branches. Use `git init` to create it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir branch-management-lab
    cd branch-management-lab
    git init

    # Create initial commit on main
    echo "# Branch Management Lab" > README.md
    git add README.md
    git commit -m "Initial commit"

    # Create develop branch
    git checkout -b develop
    echo "Development branch" >> README.md
    git add README.md
    git commit -m "Initialise develop branch"

    # View branches
    git branch -a
    ```

    **Long-lived branches:** `main` and `develop` exist permanently.
    </details>

2. **Use Naming Conventions**

    Create feature branches with consistent, descriptive names.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Feature branches with prefixes
    git checkout develop

    git checkout -b feature/user-authentication
    echo "auth.js" > auth.js
    git add auth.js
    git commit -m "Add authentication module"

    git checkout develop
    git checkout -b fix/login-redirect
    echo "fix.txt" > fix.txt
    git add fix.txt
    git commit -m "Fix login redirect issue"

    # View all branches
    git branch
    ```

    **Naming conventions:** `feature/`, `fix/`, `hotfix/`
    </details>

3. **Identify Merged Branches**

    Find which branches have been integrated using `git branch --merged`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Merge a feature
    git checkout develop
    git merge feature/user-authentication

    # Check merged branches
    git branch --merged

    # Check unmerged branches
    git branch --no-merged
    ```

    Merged branches are safe to delete.
    </details>

### Part 2: Branch Cleanup

4. **Delete Merged Branches**

    Remove branches that have been integrated using `git branch -d`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Delete merged branch (safe)
    git branch -d feature/user-authentication

    # Try to delete unmerged branch (will fail)
    git branch -d fix/login-redirect

    # Force delete if certain
    git branch -D fix/login-redirect

    # View remaining branches
    git branch
    ```

    **Safety:** `-d` for safe deletion, `-D` to force.
    </details>

5. **Sort Branches by Activity**

    Find recently active branches using `git branch --sort`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create branches with different ages
    git checkout -b feature/old-feature develop
    echo "old" > old.txt
    git add old.txt
    git commit -m "Old feature"

    git checkout -b feature/new-feature develop
    echo "new" > new.txt
    git add new.txt
    git commit -m "New feature"

    # Sort by last commit date
    git branch --sort=-committerdate

    # View with dates
    git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short) %(committerdate:relative)'
    ```

    **Use case:** Identify stale branches for cleanup.
    </details>

6. **Create Branches with Issue Numbers**

    Link branches to tracking systems.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Feature with issue number
    git checkout -b feature/123-user-profile develop
    echo "profile.js" > profile.js
    git add profile.js
    git commit -m "Add user profile (closes #123)"

    # Bug fix with issue number
    git checkout -b fix/456-session-timeout develop
    echo "session.js" > session.js
    git add session.js
    git commit -m "Fix session timeout (fixes #456)"

    # View branches
    git branch
    ```

    **Pattern:** `<type>/<issue-number>-<description>`
    </details>

### Part 3: Branch Recovery

7. **Recover a Deleted Branch**

    Use reflog to restore an accidentally deleted branch.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create branch
    git checkout -b feature/important develop
    echo "important work" > important.txt
    git add important.txt
    git commit -m "Important work"

    # Note the commit hash
    git log --oneline -1

    # Delete the branch
    git checkout develop
    git branch -D feature/important

    # Recover using reflog
    git reflog | grep "important"

    # Recreate branch
    git branch feature/recovered <commit-hash>

    # Verify
    git checkout feature/recovered
    cat important.txt
    ```

    **The reflog saves deleted branches!**
    </details>

## Challenge Exercises

### Challenge 1: Implement Branch Workflow

Create a repository with `main` and `develop` branches, 3 feature branches with proper naming, merge them to develop, then create a release.

<details>
<summary>Hint</summary>

Use `feature/` prefix for branches, merge to develop with `--no-ff`, then merge develop to main for the release.
</details>

### Challenge 2: Bulk Cleanup

Create 10 feature branches, merge 5 of them, then identify and delete only the merged branches using `git branch --merged`.

<details>
<summary>Hint</summary>

Use `git branch --merged | grep -v "main\|develop"` to filter, then pipe to `xargs git branch -d`.
</details>

## Reflection Questions

1. What are the benefits of consistent branch naming conventions?
2. How often should you clean up merged branches?
3. Why distinguish between long-lived and short-lived branches?
4. What strategies can prevent branch proliferation?

## What You've Learned

- Distinguishing between long-lived and short-lived branches
- Implementing consistent branch naming conventions
- Identifying merged and unmerged branches
- Safely deleting local branches
- Sorting branches by activity
- Linking branches to issue tracking systems
- Recovering accidentally deleted branches using reflog
- Best practices for branch management

## Clean Up

```bash
cd ..
rm -rf branch-management-lab
```
