# Lab 4: Understanding Merge Strategies

__Objective__: To understand and practise different Git merge strategies through hands-on exercises

## Prerequisites

- Completion of Labs 1-3 or equivalent Git knowledge
- Understanding of basic branching and merging
- Git installed on your machine
- A terminal or command prompt

## Step-by-Step Exercise

### Part 1: Fast-Forward Merges

1. **Create a Test Repository**

    Set up a fresh repository for experimenting with merge strategies. Use `git init` to create it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir merge-strategies-lab
    cd merge-strategies-lab
    git init
    ```
    </details>

2. **Create a Fast-Forward Merge Scenario**

    Create an initial commit on main.

    Next, create a feature branch from main, make a commit on the feature branch, and observe that main hasn't changed.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create initial commit on main
    echo "# Merge Strategies Lab" > README.md
    git add README.md
    git commit -m "Initial commit"

    # Create and switch to feature branch
    git checkout -b feature-add-content
    echo "This is new content" > content.txt
    git add content.txt
    git commit -m "Add content file"

    # View the state using git log --oneline --graph --all
    git log --oneline --graph --all
    ```

    Main hasn't changed since the feature branch was created.
    </details>

3. **Perform a Fast-Forward Merge**

    Switch to main and merge the feature branch using `git merge`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Switch to main
    git checkout main

    # Merge feature branch (fast-forward)
    git merge feature-add-content

    # Output shows: "Fast-forward"

    # View the result
    git log --oneline --graph --all
    ```

    No merge commit was created—main simply moved forward!
    </details>

### Part 2: Three-Way Merges

4. **Create Divergent Branches**

    We'll make commits on both main and a feature branch so they diverge.

    1. Create a feature branch from main.
    2. Make a commit on the feature branch.
    3. Make a commit on main.
    4. Observe that the branches have diverged.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create another feature branch
    git checkout -b feature-experiment
    echo "Experimental feature" > experiment.txt
    git add experiment.txt
    git commit -m "Add experimental feature"

    # Return to main and add a commit
    git checkout main
    echo "Main branch update" >> README.md
    git add README.md
    git commit -m "Update README on main"

    # View the divergent history
    git log --oneline --graph --all
    ```

    The branches have now diverged.
    </details>

5. **Perform a Merge**

    Merge the divergent branches using `git merge`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Merge feature-experiment into main
    git checkout main
    git merge feature-experiment

    # Git creates a merge commit (editor opens)
    # Save and close

    # View the result
    git log --oneline --graph --all
    ```

    </details>

    A merge commit now connects both lines of development.

    How can you view the merge commit? Take a look at it. What do you notice?


### Part 3: Controlling Merge Behaviour

6. **Force a Merge Commit with --no-ff**

    Create a merge commit even when fast-forward is possible using `git merge --no-ff`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create new branch from main
    git checkout -b feature-docs
    echo "# Documentation" > DOCS.md
    git add DOCS.md
    git commit -m "Add documentation"

    # Switch to main (unchanged)
    git checkout main

    # Force merge commit with --no-ff
    git merge --no-ff feature-docs -m "Merge feature-docs"

    # View the history
    git log --oneline --graph --all
    ```
    </details>

    A merge commit was created even though fast-forward was possible.

    **Why use --no-ff?** It preserves the context that this was a feature branch.

7. **Check Which Branches Are Merged**

    Identify which branches have been integrated using `git branch --merged`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Show branches merged into current branch
    git branch --merged

    # Show branches not yet merged
    git branch --no-merged
    ```
    </details>

    Merged branches are typically safe to delete.

### Part 4: Merge Strategy Options

8. **Use the -X ours Strategy Option**

    When conflicts occur, automatically favour our version using `git merge -X ours`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create conflicting scenario
    git checkout -b feature-conflict
    echo "Feature version" > conflict.txt
    git add conflict.txt
    git commit -m "Add conflict file from feature"

    # Create same file on main with different content
    git checkout main
    echo "Main version" > conflict.txt
    git add conflict.txt
    git commit -m "Add conflict file from main"

    # Merge favouring our version
    git merge -X ours feature-conflict

    # Check the result
    cat conflict.txt  # Shows "Main version"
    ```
    </details>

    **Note:** `-X ours` is a strategy option, not the name of a strategy.

9. **Abort a Merge**

    Practise cancelling a merge operation using `git merge --abort`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create another conflict
    git checkout -b to-be-aborted
    echo "content" > abort-test.txt
    git add abort-test.txt
    git commit -m "Branch to be aborted"

    git checkout main
    echo "conflict" > abort-test.txt
    git add abort-test.txt
    git commit -m "Conflicting change"

    # Start merge (will conflict)
    git merge to-be-aborted

    # Check status
    git status

    # Abort the merge
    git merge --abort

    # Verify clean state
    git status
    ```
    </details>

## Challenge Exercises

### Challenge 1: Compare Merge vs No-FF

Create two identical scenarios. In the first, use a regular merge. In the second, use `--no-ff`. Compare the resulting histories using `git log --graph`.

<details>
<summary>Hint</summary>

Create two branches from the same point on main. Merge one normally and one with `--no-ff`. Use `git log --oneline --graph --all` to compare.
</details>

### Challenge 2: Simulate a Release Workflow

Create a workflow that simulates merging multiple features into a develop branch, then merging develop into main with `--no-ff`.

<details>
<summary>Hint</summary>

Create a develop branch, create and merge two feature branches into it, then merge develop into main with `--no-ff` to mark the release.
</details>

## Reflection Questions

1. When would you want to use `--no-ff` versus allowing fast-forward merges?
2. What are the trade-offs between fast-forward merges (linear history) and merge commits (preserved context)?
3. What is the difference between `-X ours` and `-s ours`?

## What You've Learned

- The difference between fast-forward and three-way merges
- How to force merge commits with `--no-ff`
- When and how to use strategy options like `-X ours`
- How to abort merges that haven't been completed
- How to identify which branches have been merged
- The trade-offs between different merge approaches

## Clean Up

```bash
cd ..
rm -rf merge-strategies-lab
```
