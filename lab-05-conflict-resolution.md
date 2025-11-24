# Lab 5: Conflict Resolution

__Objective__: To understand and practise resolving merge conflicts through hands-on exercises

## Prerequisites

- Completion of Labs 1-4 or equivalent Git knowledge
- Understanding of branching and merging
- Git installed on your machine
- A terminal or command prompt

## Step-by-Step Exercise

### Part 1: Understanding Conflicts

1. **Create a Test Repository**

    Set up a fresh repository for practising conflict resolution. Use `git init` to create it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir conflict-resolution-lab
    cd conflict-resolution-lab
    git init
    ```
    </details>

2. **Create a Simple Conflict**

    We'll create a file on main, then modify the file (same lines) on a different branch.

    1. Create a file on main.
    2. Modify the file (same lines) on a different branch.
    3. Observe that the branches have diverged.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create initial file on main
    echo "Welcome to our application" > README.md
    git add README.md
    git commit -m "Initial README"

    # Create feature branch and modify the file
    git checkout -b feature-docs
    echo "Welcome to our amazing application" > README.md
    echo "This application helps users manage tasks." >> README.md
    git add README.md
    git commit -m "Enhance README documentation"

    # Switch to main and make conflicting change
    git checkout main
    echo "Welcome to our application" > README.md
    echo "This app is for project management." >> README.md
    git add README.md
    git commit -m "Add description to README"

    # View the divergent history
    git log --oneline --graph --all
    ```
    </details>

    Both branches modified the same file differently.

3. **Trigger the Conflict**

    Attempt to merge and observe the conflict using `git merge`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Attempt merge
    git merge feature-docs

    # Git reports: "CONFLICT (content): Merge conflict in README.md"

    # Check status
    git status

    # View the conflicted file
    cat README.md
    ```
    </details>

    You'll see conflict markers showing both versions.

4. **Understand Conflict Markers**

    Examine the conflict markers in the file using `cat`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat README.md
    ```

    **Understanding the markers:**
    - `<<<<<<< HEAD` - Your current branch's version (main)
    - `=======` - Separator between versions
    - `>>>>>>> feature-docs` - The incoming branch's version
    </details>

### Part 2: Resolving Conflicts

5. **Manually Resolve the Conflict**

    Edit the file to resolve the conflict, then use `git add` to mark it resolved.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Edit to combine both descriptions
    cat > README.md << 'EOF'
    Welcome to our amazing application
    This application helps users manage tasks and projects.
    EOF

    # Mark as resolved
    git add README.md

    # Check status
    git status  # Shows "All conflicts fixed but you are still merging"

    # Complete the merge
    git commit
    ```

    The conflict is now resolved!
    </details>

6. **Use --ours to Accept Our Version**

    When you want to keep your version entirely, use `git checkout --ours`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create another conflict
    git checkout -b feature-config
    echo "port=8080" > config.txt
    git add config.txt
    git commit -m "Set port to 8080"

    git checkout main
    echo "port=3000" > config.txt
    git add config.txt
    git commit -m "Set port to 3000"

    # Merge (will conflict)
    git merge feature-config

    # Accept our version (main's)
    git checkout --ours config.txt
    git add config.txt
    git commit -m "Merge feature-config, keeping main's port"

    # Verify
    cat config.txt  # Shows port=3000
    ```
    </details>

7. **Use --theirs to Accept Their Version**

    When you want the incoming version, use `git checkout --theirs`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create another conflict
    git checkout -b feature-api
    echo "api_version=2.0" > api.txt
    git add api.txt
    git commit -m "Update to API version 2.0"

    git checkout main
    echo "api_version=1.5" > api.txt
    git add api.txt
    git commit -m "Set API version to 1.5"

    # Merge (will conflict)
    git merge feature-api

    # Accept their version (feature-api's)
    git checkout --theirs api.txt
    git add api.txt
    git commit -m "Merge feature-api, using newer version"

    # Verify
    cat api.txt  # Shows api_version=2.0
    ```
    </details>

### Part 3: Advanced Conflict Resolution

8. **Abort a Merge**

    Practise cancelling a merge when you need to reconsider using `git merge --abort`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create a conflict
    git checkout -b feature-test
    echo "test=true" > test.txt
    git add test.txt
    git commit -m "Enable testing"

    git checkout main
    echo "test=false" > test.txt
    git add test.txt
    git commit -m "Disable testing"

    # Start merge
    git merge feature-test

    # Check status
    git status

    # Decide to abort
    git merge --abort

    # Verify clean state
    git status
    ```
    </details>

9. **Resolve Multiple File Conflicts**

    Resolve conflicts across multiple files using a combination of `--ours` and `--theirs`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create multiple conflicting files
    git checkout -b feature-multi
    echo "auth=enabled" > auth.txt
    echo "logging=verbose" > logging.txt
    git add auth.txt logging.txt
    git commit -m "Configure auth and logging"

    git checkout main
    echo "auth=disabled" > auth.txt
    echo "logging=minimal" > logging.txt
    git add auth.txt logging.txt
    git commit -m "Different auth and logging config"

    # Merge (multiple conflicts)
    git merge feature-multi

    # Check which files have conflicts
    git diff --name-only --diff-filter=U

    # Resolve differently for each file
    git checkout --theirs auth.txt
    git checkout --ours logging.txt
    git add auth.txt logging.txt

    # Complete merge
    git commit -m "Merge feature-multi with selective resolutions"
    ```
    </details>

## Challenge Exercises

### Challenge 1: Three-Way Conflict

Create three branches that all modify the same file differently. Merge them sequentially into main and resolve each conflict.

<details>
<summary>Hint</summary>

Create branches feature-a, feature-b, and feature-c from the same base. Each modifies the same file. Merge them one by one, resolving conflicts at each step.
</details>

### Challenge 2: Delete/Modify Conflict

Create a scenario where one branch deletes a file whilst another modifies it. Resolve the conflict by deciding whether to keep or delete the file.

<details>
<summary>Hint</summary>

Create a file, branch off, delete it in one branch, modify it in another. When merging, use `git add` to keep it or `git rm` to delete it.
</details>

## Reflection Questions

1. Why might you use `--ours` or `--theirs` instead of manually resolving?
2. When should you abort a merge rather than resolving conflicts?
3. What strategies can help prevent conflicts in the first place?
4. Why is it important to test code after resolving conflicts?

## What You've Learned

- How merge conflicts occur when the same lines are modified
- Understanding conflict markers (HEAD, separator, branch name)
- Manually resolving conflicts by editing files
- Using `--ours` and `--theirs` for whole-file resolution
- Aborting merges with `git merge --abort`
- Resolving multiple file conflicts selectively
- Best practices for conflict resolution

## Clean Up

```bash
cd ..
rm -rf conflict-resolution-lab
```
