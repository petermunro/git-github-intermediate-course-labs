# Lab 12: Stashing and Context Switching

__Objective__: To practise using Git stash for managing uncommitted changes and efficiently switching between tasks

## Prerequisites

- Completion of Labs 1-11 or equivalent Git knowledge
- Understanding of branching and working directory concepts
- Git installed on your machine
- A terminal or command prompt

## About This Lab

In Module 12, you learnt about Git stash - a tool for temporarily saving uncommitted work. Key concepts:

- **`git stash`** - Save current uncommitted changes
- **`git stash pop`** - Apply and remove most recent stash
- **`git stash apply`** - Apply but keep in stash list
- **`git stash list`** - View all stashes
- **`git stash -u`** - Include untracked files

Stashing is essential when you need to:
- Switch contexts urgently
- Pull changes with a dirty working directory
- Test something on a clean state

## Step-by-Step Exercise

### Part 1: Basic Stash Workflow

1. **Create a Practice Repository**

    Create a new repository called `stashing-lab` and initialise Git.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir stashing-lab
    cd stashing-lab
    git init
    ```
    </details>

2. **Create Initial Project Files**

    Create `app.js` with a simple function, then commit it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    echo "function login() { return true; }" > app.js
    git add app.js
    git commit -m "Initial commit"
    ```
    </details>

3. **Make Uncommitted Changes and Stash**

    Start working on a new feature by adding a function to `app.js`. Then stash your work using `git stash`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Start working on a feature
    echo "function logout() { /* TODO */ }" >> app.js

    # Check status - file is modified
    git status

    # Urgent task comes in - stash your work
    git stash

    # Working directory is now clean
    git status
    ```

    The output shows "working tree clean" - your changes are safely stored.
    </details>

4. **View Your Stashes**

    Use `git stash list` to see all stashed changes.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    git stash list
    ```

    Output: `stash@{0}: WIP on main: abc1234 Initial commit`

    Each stash has an index (0, 1, 2...) and shows the branch and commit where it was created.
    </details>

5. **Restore Stashed Changes**

    Apply your stashed changes back to the working directory using `git stash pop`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Restore and remove from stash list
    git stash pop

    # Verify the changes are back
    cat app.js

    # Check stash list is now empty
    git stash list
    ```

    `pop` applies the stash and removes it from the list.
    </details>

### Part 2: Working with Multiple Stashes

6. **Create Multiple Stashes with Messages**

    Create two different stashes with descriptive messages using `git stash push -m "message"`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Work on feature 1
    echo "function feature1() { }" >> app.js
    git stash push -m "Feature 1: Add user feature"

    # Work on feature 2
    echo "function feature2() { }" >> app.js
    git stash push -m "Feature 2: Add admin feature"

    # List stashes
    git stash list
    ```

    Output shows both stashes with your custom messages.
    </details>

7. **Apply a Specific Stash**

    Apply the first stash (feature 1) without removing it, using `git stash apply stash@{1}`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Apply the older stash (stash@{1})
    git stash apply stash@{1}

    # Verify it's still in the list
    git stash list

    # See what was applied
    cat app.js
    ```

    Using `apply` instead of `pop` keeps the stash in the list for reuse.
    </details>

8. **View Stash Contents Before Applying**

    Use `git stash show -p stash@{0}` to preview a stash before applying it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Show summary
    git stash show stash@{0}

    # Show full diff
    git stash show -p stash@{0}
    ```

    The `-p` flag shows the actual changes, helping you decide if you want to apply it.
    </details>

### Part 3: Advanced Stashing

9. **Stash Untracked Files**

    Create a new file (untracked) and stash it using the `-u` flag.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Reset working directory
    git reset --hard HEAD

    # Create new untracked file
    echo "function newFeature() { }" > feature.js

    # Regular stash won't save untracked files
    git stash
    ls  # feature.js still there

    # Stash with untracked files
    git stash push -u -m "New feature with untracked file"

    # Now feature.js is gone
    ls
    ```

    Without `-u`, untracked files are ignored by stash.
    </details>

10. **Stash Specific Files**

    Make changes to multiple files but stash only one. Use `git stash push <file>`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Pop previous stash first
    git stash pop

    # Modify multiple files
    echo "// Change 1" >> app.js
    echo "// Change 2" >> feature.js

    # Stash only app.js
    git stash push -m "Just app.js changes" app.js

    # Check status
    git status  # feature.js still modified

    # Verify with diff
    git diff
    ```

    This is useful when you have unrelated changes and want to stash them separately.
    </details>

11. **Create Branch from Stash**

    Turn a stash into a new branch using `git stash branch <branchname>`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Clean working directory
    git reset --hard HEAD

    # Create some experimental work
    echo "function experiment() { }" >> app.js
    git stash push -m "Experimental feature"

    # Later, promote it to a proper branch
    git stash branch feature/experimental stash@{0}

    # This creates the branch and applies the stash
    git status
    git branch
    ```

    This is perfect for when stashed work deserves its own feature branch.
    </details>

12. **Handle Stash Conflicts**

    Create a conflict scenario when applying a stash.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Ensure we're on main
    git checkout main

    # Create a file
    echo "Version 1" > config.txt
    git add config.txt
    git commit -m "Add config"

    # Modify and stash
    echo "Stashed version" > config.txt
    git stash

    # Modify the same file differently
    echo "Different version" > config.txt
    git add config.txt
    git commit -m "Update config"

    # Try to apply stash - conflict!
    git stash pop
    # CONFLICT message appears

    # Resolve the conflict
    cat config.txt  # Shows conflict markers

    # Fix it manually
    echo "Resolved version" > config.txt
    git add config.txt

    # Drop the stash (since pop failed, it's still there)
    git stash drop
    ```

    When `pop` creates a conflict, the stash remains in the list until you drop it manually.
    </details>

## Challenge Exercises

### Challenge 1: Emergency Context Switch

Simulate a real emergency scenario:
1. Create a feature branch and make changes (don't commit)
2. Emergency bug report arrives
3. Stash your work
4. Create hotfix branch from main
5. Fix bug and merge
6. Return to feature work

<details>
<summary>Hint</summary>

Use `git stash push -u -m "descriptive message"` before switching branches. After handling the emergency, use `git checkout <feature-branch>` followed by `git stash pop` to restore your work.
</details>

### Challenge 2: Multiple Stash Management

Create a workflow with 3 different stashes:
1. Feature work stash
2. Bug fix stash
3. Experimental work stash

Practise applying them in different orders and cleaning up completed ones.

<details>
<summary>Hint</summary>

Use `git stash list` to track them all. Use `git stash drop stash@{n}` to remove completed stashes. Consider using clear naming conventions in your stash messages.
</details>

## Reflection Questions

After completing this lab, consider:

1. **When to Stash:** When should you use stash instead of committing work-in-progress?
2. **Stash vs Branch:** How does stashing compare to creating a temporary branch?
3. **Organisation:** How can descriptive messages improve stash management?
4. **Cleanup:** Why is it important to regularly clean up old stashes?

## Clean Up

If you want to remove the lab directory:

```bash
cd ..
rm -rf stashing-lab
```

## Key Commands Summary

```bash
# Basic stashing
git stash                           # Stash tracked changes
git stash push -m "message"         # Stash with message
git stash push -u -m "message"      # Include untracked files

# Viewing stashes
git stash list                      # List all stashes
git stash show stash@{0}            # Show stash summary
git stash show -p stash@{0}         # Show stash diff

# Applying stashes
git stash pop                       # Apply and remove latest
git stash apply stash@{0}           # Apply but keep in list

# Managing stashes
git stash drop stash@{0}            # Remove specific stash
git stash clear                     # Remove all stashes
git stash branch <name> stash@{0}   # Create branch from stash

# Partial stashing
git stash push <file>               # Stash specific file
```

## What You've Learnt

- Using Git stash to save uncommitted work
- Stashing with descriptive messages
- Understanding `pop` vs `apply`
- Including untracked files with `-u`
- Stashing specific files for targeted saves
- Viewing stash contents before applying
- Creating branches from stashed work
- Handling stash conflicts
- Managing multiple stashes effectively
- When to use stash vs commit vs branch
- Cleaning up old stashes

Stashing is essential for efficient context switching in your daily workflow!
