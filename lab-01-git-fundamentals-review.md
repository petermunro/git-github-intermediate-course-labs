# Lab 1: Git Fundamentals Review

__Objective__: To reinforce understanding of Git fundamentals through practical exercises

## Prerequisites

- Git installed on your machine
- A terminal or command prompt
- A text editor

## Step-by-Step Exercise

### Part 1: Basic Git Workflow

1. **Create a Git Repository**

    Create a new directory called `git-fundamentals-lab` and initialise it as a Git repository. Use `git init` to initialise it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir git-fundamentals-lab
    cd git-fundamentals-lab
    git init
    ```

    Verify with: `git status`
    </details>

2. **Configure Your Identity**

    Set your name and email address globally. Use `git config --global user.name` and `git config --global user.email`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "your.email@example.com"
    ```

    Verify: `git config user.name` and `git config user.email`
    </details>

3. **Create, Stage, and Commit Files**

    Create a file called `README.md` with the text "# Git Fundamentals Lab". Stage it using `git add`, then commit it with message "Initial commit".

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    echo "# Git Fundamentals Lab" > README.md
    git add README.md
    git commit -m "Initial commit"
    ```

    Use `git status` after each step to see how the state changes.
    </details>

### Part 2: Working with Changes

4. **Make and View Changes**

    Modify `README.md` to add a second line "This is a lab exercise". Use `git diff` to see the unstaged changes before staging them.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    echo "This is a lab exercise" >> README.md

    # View unstaged changes
    git diff

    # Stage the changes
    git add README.md

    # View staged changes
    git diff --staged
    ```

    Notice how `git diff` shows different things before and after staging!
    </details>

5. **Commit Changes and View History**

    Commit your staged changes with message "Add description to README". Then use `git log --oneline` to view your commit history.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    git commit -m "Add description to README"

    # View commit history
    git log --oneline
    ```

    You should see both commits with their hashes and messages.
    </details>

### Part 3: Branching and Merging

6. **Create and Switch to a Branch**

    Create a new branch called `feature-greeting` and switch to it. Use `git checkout -b feature-greeting` to do both in one command.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    git checkout -b feature-greeting

    # Verify you're on the new branch
    git branch
    ```

    The asterisk (*) shows your current branch.
    </details>

7. **Make Changes on the Branch**

    Create a file called `greeting.js` with the content `function greet(name) { return "Hello, " + name; }`. Stage and commit it with message "Add greeting function".

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    echo 'function greet(name) { return "Hello, " + name; }' > greeting.js
    git add greeting.js
    git commit -m "Add greeting function"
    ```

    Use `ls` to see the file is present on this branch.
    </details>

8. **Switch Branches and Observe Differences**

    Switch back to the `main` branch using `git checkout main`. Notice that `greeting.js` disappears. Then switch back to `feature-greeting` and see it return.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Switch to main
    git checkout main
    ls    # greeting.js is not here!

    # Switch back to feature
    git checkout feature-greeting
    ls    # greeting.js is back!
    ```

    Git changes your working directory to match the branch!
    </details>

9. **Merge Your Branch**

    Switch to `main` and merge the `feature-greeting` branch into it. Use `git merge feature-greeting`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    git checkout main
    git merge feature-greeting
    ```

    You'll see "Fast-forward" - this means Git just moved the `main` pointer forward.

    Verify: `ls` should now show `greeting.js` on main.
    </details>

### Part 4: Ignoring Files

10. **Create a .gitignore File**

    Create a file called `.env` with some dummy content. Then create a `.gitignore` file to ignore it. Use `git status` to verify the `.env` file doesn't appear.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create a file we don't want to track
    echo "SECRET_KEY=abc123" > .env

    # Create .gitignore
    echo ".env" > .gitignore

    # Check status
    git status
    ```

    `.env` shouldn't appear in the status, but `.gitignore` should!

    Commit the `.gitignore` file:
    ```bash
    git add .gitignore
    git commit -m "Add .gitignore"
    ```
    </details>

## Challenge Exercises

### Challenge 1: Amend a Commit

After your last commit, realise you forgot to add a comment to `greeting.js`. Add a comment line, stage it, then use `git commit --amend` to modify the last commit instead of creating a new one.

<details>
<summary>Hint</summary>

```bash
echo "// Greets a person by name" > greeting-comment.js
cat greeting.js >> greeting-comment.js
mv greeting-comment.js greeting.js

git add greeting.js
git commit --amend
```

The commit hash will change because you've modified the commit!
</details>

### Challenge 2: View Different Log Formats

Explore different ways to view your commit history: `git log`, `git log --oneline`, `git log --graph --all`, `git log --stat`.

## Reflection Questions

1. What's the difference between `git diff` and `git diff --staged`?
2. Why does `greeting.js` disappear when you switch to `main` (before the merge)?
3. What happens to files when you switch branches?
4. Why do we commit `.gitignore` but not `.env`?

## What You've Learned

- How to initialise a repository and configure your identity
- The basic workflow: modify → stage → commit
- How to view changes with `git diff` and history with `git log`
- How to create branches and switch between them
- How branches isolate work until you merge them
- How to ignore files you don't want to track

## Clean Up

```bash
cd ..
rm -rf git-fundamentals-lab
```
