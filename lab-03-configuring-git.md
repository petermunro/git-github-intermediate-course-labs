# Lab 3: Configuring Git

__Objective__: To configure Git for professional workflows using global and local configuration

## Prerequisites

- Completion of Labs 1-2 or equivalent Git knowledge
- Git installed on your machine
- A terminal or command prompt

## Step-by-Step Exercise

### Part 1: Essential Configuration

1. **View Current Configuration**

    Check what settings you already have. Use `git config --list` to see all configuration.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # View all configuration
    git config --list

    # View with file locations
    git config --list --show-origin

    # Check specific settings
    git config user.name
    git config user.email
    ```

    The `--show-origin` flag shows where each setting comes from.
    </details>

2. **Configure Your Identity**

    Set your name and email globally using `git config --global`. These appear in every commit you make.

    You can also set your identity locally using `git config --local`. These appear in every commit you make in the repository.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Set your name
    git config --global user.name "Your Full Name"

    # Set your email
    git config --global user.email "your.email@example.com"

    # Verify the settings
    git config user.name
    git config user.email
    ```

    These settings are stored in `~/.gitconfig`.
    </details>

3. **Set Your Default Editor**

    Configure which editor Git opens for commit messages using `git config --global core.editor`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # For VS Code
    git config --global core.editor "code --wait"

    # For Vim
    git config --global core.editor "vim"

    # For Nano
    git config --global core.editor "nano"

    # Verify
    git config core.editor
    ```
    </details>

4. **Configure Line Endings**

    Set line ending behaviour appropriate for your operating system using `core.autocrlf`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # On Windows
    git config --global core.autocrlf true

    # On macOS or Linux
    git config --global core.autocrlf input

    # Verify
    git config core.autocrlf
    ```

    **What this does:**
    - `true` (Windows): Converts LF to CRLF on checkout, CRLF to LF on commit
    - `input` (macOS/Linux): Converts CRLF to LF on commit, no conversion on checkout
    </details>

5. **Set Default Branch Name**

    Configure the default branch name for new repositories to `main` using `init.defaultBranch`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Set default branch to 'main'
    git config --global init.defaultBranch main

    # Verify by creating a test repository
    mkdir test-default
    cd test-default
    git init
    git branch --show-current
    cd ..
    rm -rf test-default
    ```
    </details>

### Part 2: Useful Aliases

6. **Create Common Aliases**

    Set up shortcuts for frequently used commands using `git config --global alias.<name>`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Short status
    git config --global alias.st "status -s"

    # Pretty log
    git config --global alias.lg "log --graph --oneline --all --decorate"

    # View staged changes
    git config --global alias.staged "diff --staged"

    # List all aliases
    git config --global alias.aliases "config --get-regexp ^alias\."

    # Test your aliases
    git aliases
    ```

    Now you can use `git st` instead of `git status -s`.
    </details>

7. **Create a Global .gitignore File**

    Set up a global gitignore for files you never want to track (OS files, editor files).

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create the global gitignore file
    cat > ~/.gitignore_global << 'EOF'
    # OS files
    .DS_Store
    Thumbs.db

    # Editor files
    .vscode/
    .idea/
    *.swp

    # Temporary files
    *.tmp
    *.log
    EOF

    # Configure Git to use it
    git config --global core.excludesfile ~/.gitignore_global

    # Verify
    git config core.excludesfile
    ```

    These files will be ignored in all your repositories.
    </details>

### Part 3: Local Configuration

8. **Override Global Settings Locally**

    Create a repository and set repository-specific configuration using `git config --local`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create a test repository
    mkdir git-config-lab
    cd git-config-lab
    git init

    # Set local (repository-specific) email
    git config --local user.email "project.specific@example.com"

    # Check effective configuration
    echo "Local email:"
    git config user.email

    echo "Global email:"
    git config --global user.email

    # View where settings come from
    git config --list --show-origin | grep user.email
    ```

    Local configuration overrides global configuration!
    </details>

9. **View Configuration Levels**

    Understand the three configuration levels: system, global, and local.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # View only global settings
    git config --global --list

    # View only local settings (must be in a repository)
    git config --local --list

    # View all settings with origin
    git config --list --show-origin
    ```

    **Priority:** local > global > system (local wins)
    </details>

## Challenge Exercises

### Challenge 1: Create Useful Team Aliases

Create aliases for common team workflows: viewing recent branches, showing who changed a file, and viewing the last commit.

<details>
<summary>Reveal Solution</summary>

```bash
# Recent branches
git config --global alias.recent "branch --sort=-committerdate"

# Who changed a file
git config --global alias.who "blame -w -C -C -C"

# Show last commit
git config --global alias.last "log -1 HEAD --stat"

# Test them
git recent
git last
```
</details>

### Challenge 2: Conditional Configuration

Set up different email addresses for work and personal projects using conditional includes.

<details>
<summary>Hint</summary>

Create separate config files (`~/.gitconfig-work` and `~/.gitconfig-personal`), then add conditional includes to your main `~/.gitconfig`:

```
[includeIf "gitdir:~/work/"]
    path = ~/.gitconfig-work
[includeIf "gitdir:~/personal/"]
    path = ~/.gitconfig-personal
```
</details>

## Reflection Questions

1. When would you use `--global` versus `--local` configuration?
2. Why is a global .gitignore file useful?
3. How do Git aliases improve your workflow?
4. What happens when local and global settings conflict?

## What You've Learned

- How to view your current Git configuration
- Setting your identity with `user.name` and `user.email`
- Configuring your default editor
- Setting line ending behaviour
- Creating aliases for common commands
- Setting up a global .gitignore file
- Understanding configuration levels (global vs local)
- Overriding global settings with local configuration

## Clean Up

```bash
cd ..
rm -rf git-config-lab
```
