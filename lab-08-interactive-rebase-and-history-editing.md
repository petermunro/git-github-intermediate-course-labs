# Lab 8: Interactive Rebase and History Editing

__Objective__: To master interactive rebase for editing, combining, and reorganising commits

## Prerequisites

- Completion of Labs 1-7 or equivalent Git knowledge
- Understanding of basic rebasing
- Git installed on your machine
- A text editor configured for Git

## Step-by-Step Exercise

### Part 1: Basic Interactive Rebase

1. **Create a Repository with Messy Commits**

    Set up a repository with various types of commits. Use `git init` to create it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir interactive-rebase-lab
    cd interactive-rebase-lab
    git init

    # Initial commit
    echo "# Interactive Rebase Lab" > README.md
    git add README.md
    git commit -m "Initial commit"

    # Create messy commits
    echo "function auth() { /* TODO */ }" > auth.js
    git add auth.js
    git commit -m "Add authentication"

    echo "function auth() { return true; }" > auth.js
    git add auth.js
    git commit -m "WIP"

    echo "function validate() { return true; }" > validation.js
    git add validation.js
    git commit -m "Add validation"

    # View history
    git log --oneline
    ```
    </details>

2. **Start Interactive Rebase**

    Open the interactive rebase editor using `git rebase -i HEAD~3`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Rebase the last 3 commits
    git rebase -i HEAD~3

    # Git opens your editor with:
    # pick abc123 Add authentication
    # pick def456 WIP
    # pick ghi789 Add validation
    ```

    Just observe the format, then close without changes.
    </details>

3. **Reword a Commit Message**

    Fix a commit message using `reword`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Start interactive rebase
    git rebase -i HEAD~3

    # Change "WIP" line from:
    # pick def456 WIP
    # to:
    # reword def456 WIP

    # Save and close

    # Git opens editor for that commit
    # Change to: "Implement authentication logic"
    # Save and close

    # View result
    git log --oneline
    ```
    </details>

### Part 2: Combining Commits

4. **Squash Commits Together**

    Combine related commits using `squash`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create more commits
    echo "function logout() { /* logout */ }" >> auth.js
    git add auth.js
    git commit -m "Add logout"

    echo "function logout() { return true; }" >> auth.js
    git add auth.js
    git commit -m "WIP logout"

    # Interactive rebase
    git rebase -i HEAD~2

    # Change to:
    # pick abc123 Add logout
    # squash def456 WIP logout

    # Save and close
    # Edit combined commit message
    # Save and close

    # View result
    git log --oneline
    ```

    Multiple commits are now combined!
    </details>

5. **Use Fixup Instead of Squash**

    Combine commits whilst discarding unwanted messages using `fixup`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create commits
    echo "function getUser() { return user; }" >> auth.js
    git add auth.js
    git commit -m "Add getUser"

    echo "function getUser() { return currentUser; }" >> auth.js
    git add auth.js
    git commit -m "fix"

    # Interactive rebase
    git rebase -i HEAD~2

    # Change to:
    # pick abc123 Add getUser
    # fixup def456 fix

    # Save and close

    # View result - single clean commit
    git log --oneline -1
    ```

    **Difference:** `fixup` discards messages automatically; `squash` lets you edit them.
    </details>

### Part 3: Advanced Operations

6. **Drop Unwanted Commits**

    Remove commits from history using `drop`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create debug commit
    echo "console.log('DEBUG');" >> auth.js
    git add auth.js
    git commit -m "Add debug logging"

    # Create useful commit
    echo "function isAuthenticated() { return true; }" >> auth.js
    git add auth.js
    git commit -m "Add isAuthenticated"

    # Interactive rebase
    git rebase -i HEAD~2

    # Change to:
    # drop abc123 Add debug logging
    # pick def456 Add isAuthenticated

    # Save and close

    # Verify debug commit is gone
    git log --oneline
    ```
    </details>

7. **Reorder Commits**

    Change the sequence of commits for better logical flow.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create commits in wrong order
    echo "function sendEmail() { /* send */ }" > email.js
    git add email.js
    git commit -m "Add email function"

    echo "const emailConfig = {};" > config.js
    git add config.js
    git commit -m "Add email configuration"

    # Reorder: configuration should come first
    git rebase -i HEAD~2

    # Reorder lines:
    # pick def456 Add email configuration
    # pick abc123 Add email function

    # Save and close

    # View logical order
    git log --oneline
    ```
    </details>

8. **Edit a Commit**

    Pause during rebase to modify a commit using `edit`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create incomplete commit
    echo "function parse() { return data; }" > parser.js
    git add parser.js
    git commit -m "Add parser"

    # Interactive rebase
    git rebase -i HEAD~1

    # Change to:
    # edit abc123 Add parser

    # Save and close

    # Rebase pauses. Add error handling:
    echo "function parse(data) { if (!data) throw new Error('No data'); return data; }" > parser.js

    # Amend commit
    git add parser.js
    git commit --amend --no-edit

    # Continue
    git rebase --continue
    ```
    </details>

9. **Use Autosquash**

    Automate squashing with fixup commits using `git commit --fixup`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create initial commit
    echo "function calculate() { return 0; }" > calc.js
    git add calc.js
    git commit -m "Add calculate function"

    # Note the hash
    HASH=$(git rev-parse HEAD)

    # Create fixup commit
    echo "function calculate(a, b) { return a + b; }" > calc.js
    git add calc.js
    git commit --fixup=$HASH

    # View commits
    git log --oneline

    # Rebase with autosquash
    git rebase -i --autosquash HEAD~2

    # Git automatically arranges the fixup
    # Just save and close

    # View cleaned history
    git log --oneline
    ```

    **Tip:** Enable by default with `git config --global rebase.autoSquash true`
    </details>

## Challenge Exercises

### Challenge 1: Clean Up Feature Branch

Create a branch with 8 commits including WIP commits, typos, and debug code. Use interactive rebase to create 3 clean commits.

<details>
<summary>Hint</summary>

Use `squash`, `fixup`, `reword`, `drop`, and `reorder` to transform messy history into professional commits.
</details>

### Challenge 2: Split a Commit

Create one commit that does three things. Split it into three separate commits using `edit` and `git reset HEAD~1`.

<details>
<summary>Hint</summary>

Mark the commit as `edit`, let rebase pause, use `git reset HEAD~1` to unstage, then create separate commits.
</details>

## Reflection Questions

1. How does interactive rebase help prepare code for review?
2. When should you use `squash` versus `fixup`?
3. Why is it important to only use interactive rebase on private branches?
4. How can autosquash improve your workflow?

## What You've Learned

- Starting and using interactive rebase
- Rewording commit messages
- Squashing related commits together
- Using fixup to combine commits without editing messages
- Dropping unwanted commits
- Reordering commits for logical progression
- Editing commits to add forgotten changes
- Using autosquash for efficient workflows
- Best practices for cleaning up history before sharing

## Clean Up

```bash
cd ..
rm -rf interactive-rebase-lab
```
