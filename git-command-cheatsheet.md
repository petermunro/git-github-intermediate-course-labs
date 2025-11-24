# Git Command Cheat Sheet

A task-oriented reference guide for Git commands covered in the Intermediate Git and GitHub course.

---

## Module 1: Git Fundamentals Review

### To check the status of your repository:
```bash
git status
```

### To configure your identity:
```bash
# Set your name
git config --global user.name "Your Name"

# Set your email
git config --global user.email "your.email@example.com"

# Verify your configuration
git config user.name
git config user.email
```

### To add files to the staging area:
```bash
# Stage a specific file
git add index.html

# Stage all changes in current directory
git add .

# Stage all changes in the repository
git add -A

# Interactively stage changes
git add -p
```

### To commit changes:
```bash
# Commit with inline message
git commit -m "Fix navigation bug"

# Open editor for detailed message
git commit

# Stage all tracked files and commit
git commit -a -m "Update all styles"
```

### To view commit history:
```bash
# View full commit history
git log

# Compact one-line format
git log --oneline

# View last 5 commits
git log -5

# View commits with file changes
git log --stat

# Visual branch graph
git log --graph --oneline --all
```

### To view changes and differences:
```bash
# Changes not yet staged
git diff

# Changes staged for commit
git diff --staged

# Changes between two commits
git diff abc123 def456

# Changes in a specific file
git diff index.html
```

### To list branches:
```bash
git branch
```

### To create a branch:
```bash
# Create a new branch
git branch feature-login

# Create and switch in one command
git checkout -b feature-payment

# Modern alternative: create and switch
git switch -c new-feature
```

### To switch to a branch:
```bash
# Switch to a branch (traditional)
git checkout feature-login

# Modern alternative
git switch feature-payment
```

### To merge branches:
```bash
# Switch to the branch you want to merge INTO
git checkout main

# Merge the feature branch
git merge feature-login
```

### To view configured remotes:
```bash
git remote -v
```

### To add a remote repository:
```bash
git remote add origin https://github.com/user/repo.git
```

### To fetch changes from remote:
```bash
# Downloads changes but doesn't merge
git fetch origin
```

### To pull changes from remote:
```bash
# Fetch and merge in one command
git pull origin main
```

### To push changes to remote:
```bash
git push origin main
```

### To clone a repository:
```bash
# Clone via HTTPS
git clone https://github.com/user/repository.git

# Clone via SSH
git clone git@github.com:user/repository.git

# Clone into a specific directory
git clone https://github.com/user/repo.git my-folder

# Clone a specific branch
git clone -b develop https://github.com/user/repo.git
```

### To ignore files:
Create a `.gitignore` file in your repository root with patterns for files to ignore:
```bash
# Example .gitignore patterns
node_modules/
.env
dist/
*.pyc
.DS_Store
```

---

## Module 2: Git's Object Model and Internal Structure

### To examine Git objects:
```bash
# View object contents
git cat-file -p <object-hash>

# View object type
git cat-file -t <object-hash>

# View HEAD commit
git cat-file -p HEAD

# View tree from commit
git cat-file -p HEAD^{tree}

# View specific file from commit
git cat-file -p HEAD:README.md
```

### To view the staging area:
```bash
# List files in staging area with hashes
git ls-files -s
git ls-files --stage
```

### To create object hashes:
```bash
# Generate hash for content
echo "Hello, Git!" | git hash-object --stdin
```

### To find the common ancestor of branches:
```bash
# Find merge base between two branches
git merge-base main feature-branch
```

### To view the reflog (safety net):
```bash
# View reflog for current branch
git reflog

# View reflog for specific branch
git reflog show main
```

---

## Module 3: Configuring Git

### To view Git configuration:
```bash
# View all configuration
git config --list

# View config with file locations
git config --list --show-origin

# View specific setting
git config user.name

# View system-level config
git config --system --list

# View global config
git config --global --list

# View local (repository) config
git config --local --list
```

### To set configuration values:
```bash
# Set global configuration
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set local (repository-specific) configuration
git config --local user.email "work@company.com"

# Remove a configuration value
git config --global --unset user.signingkey

# Edit configuration file directly
git config --global --edit
```

### To configure your editor:
```bash
# Set default editor
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"          # Vim
git config --global core.editor "nano"         # Nano
```

### To configure diff and merge tools:
```bash
# Configure diff tool
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# Configure merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Use diff/merge tools
git difftool
git mergetool
```

### To configure line endings:
```bash
# On Windows
git config --global core.autocrlf true

# On macOS/Linux
git config --global core.autocrlf input
```

### To create Git aliases:
```bash
# Create aliases for common commands
git config --global alias.st "status -s"
git config --global alias.lg "log --graph --oneline --all --decorate"
git config --global alias.staged "diff --staged"
git config --global alias.unstaged "diff"
git config --global alias.last "log -1 HEAD"
```

### To configure pull and push behaviour:
```bash
# Rebase on pull
git config --global pull.rebase true

# Push only current branch
git config --global push.default simple
```

### To enable automatic stash on rebase:
```bash
git config --global rebase.autoStash true
```

### To enable rerere (reuse recorded resolution):
```bash
git config --global rerere.enabled true
```

### To configure credential storage:
```bash
# macOS: Use Keychain
git config --global credential.helper osxkeychain

# Windows: Use Windows Credential Manager
git config --global credential.helper wincred

# Linux: Cache for 1 hour
git config --global credential.helper 'cache --timeout=3600'
```

### To set up a global gitignore:
```bash
# Configure Git to use global gitignore
git config --global core.excludesfile ~/.gitignore_global
```

---

## Module 4: Understanding Merge Strategies

### To force a merge commit (no fast-forward):
```bash
git merge --no-ff feature-branch
```

### To specify merge strategy:
```bash
# Use recursive strategy (default)
git merge -s recursive feature-branch

# Use ours strategy (ignore their changes)
git merge -s ours obsolete-branch

# Use octopus strategy (merge multiple branches)
git merge feature-a feature-b feature-c
```

### To use merge strategy options:
```bash
# Prefer our changes in conflicts
git merge -X ours feature-branch

# Prefer their changes in conflicts
git merge -X theirs feature-branch

# Ignore whitespace changes
git merge -X ignore-space-change feature-branch
```

### To preview a merge:
```bash
# Preview merge without executing
git merge --no-commit --no-ff feature-branch
# Review, then either:
git merge --abort  # Cancel
# or
git commit  # Complete the merge
```

### To abort a merge:
```bash
git merge --abort
```

### To configure default merge behaviour:
```bash
# Always create merge commits
git config --global merge.ff false

# Only allow fast-forward merges
git config --global merge.ff only

# Allow fast-forward when possible (default)
git config --global merge.ff true
```

### To view merge history:
```bash
# Show only merge commits
git log --merges --oneline

# Show first-parent history
git log --first-parent

# Show which branches are merged
git branch --merged
git branch --no-merged
```

---

## Module 5: Conflict Resolution

### To view conflicted files:
```bash
# List files with conflicts
git diff --name-only --diff-filter=U

# Show conflict details
git diff
```

### To view the three stages of a conflicted file:
```bash
# View base version (common ancestor)
git show :1:config.js

# View our version (current branch)
git show :2:config.js

# View their version (incoming branch)
git show :3:config.js
```

### To accept one side of a conflict:
```bash
# Accept our version
git checkout --ours config.js
git add config.js

# Accept their version
git checkout --theirs config.js
git add config.js
```

### To use a merge tool:
```bash
# Launch configured merge tool
git mergetool
```

### To work with rerere (reuse recorded resolution):
```bash
# View rerere status
git rerere status

# View recorded resolution
git rerere diff

# Forget a recorded resolution
git rerere forget config.js

# Clear all recorded resolutions
git rerere clear
```

---

## Module 6: Cherry-Picking and Selective Changes

### To cherry-pick commits:
```bash
# Cherry-pick a specific commit
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456 ghi789

# Cherry-pick a range of commits
git cherry-pick abc123..def456
git cherry-pick abc123^..def456  # Include first commit
```

### To cherry-pick with options:
```bash
# Cherry-pick without committing
git cherry-pick -n abc123
git cherry-pick --no-commit abc123

# Edit the commit message
git cherry-pick -e abc123
git cherry-pick --edit abc123

# Add note about original commit
git cherry-pick -x abc123

# Cherry-pick merge commit (specify parent)
git cherry-pick -m 1 abc123
```

### To handle cherry-pick conflicts:
```bash
# Continue after resolving conflicts
git cherry-pick --continue

# Skip this commit
git cherry-pick --skip

# Abort cherry-pick
git cherry-pick --abort
```

### To create patches:
```bash
# Create patch file for a commit
git format-patch -1 abc123

# Apply patch
git apply 0001-commit-message.patch

# Apply patch and commit
git am 0001-commit-message.patch
```

---

## Module 7: Introduction to Rebase

### To rebase branches:
```bash
# Rebase current branch onto main
git rebase main

# Rebase specific branch onto main
git rebase main feature-branch
```

### To handle rebase conflicts:
```bash
# Continue after resolving conflicts
git rebase --continue

# Skip current commit
git rebase --skip

# Abort rebase
git rebase --abort
```

### To force push after rebasing:
```bash
# Safer force push (recommended)
git push --force-with-lease

# Traditional force push
git push --force
```

---

## Module 8: Interactive Rebase and History Editing

### To start interactive rebase:
```bash
# Rebase last 5 commits interactively
git rebase -i HEAD~5

# Rebase since a specific commit
git rebase -i abc123

# Rebase onto main interactively
git rebase -i main

# Interactive rebase with autosquash
git rebase -i --autosquash main
```

### To create fixup commits:
```bash
# Create commit that will be auto-squashed
git commit --fixup=abc123
```

### To enable autosquash by default:
```bash
git config --global rebase.autoSquash true
```

### To rebase onto a different base:
```bash
# Move branch from old-base to new-base
git rebase --onto new-base old-base feature
```

---

## Module 9: Branch Management Strategies

### To list branches with details:
```bash
# List with last commit
git branch -v

# List all branches (local + remote)
git branch -a

# List remote branches only
git branch -r

# List with tracking info
git branch -vv
```

### To sort branches:
```bash
# Sort by last commit date
git branch --sort=-committerdate

# Show branches with commit details
git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short) %(committerdate:relative)'
```

### To set upstream tracking:
```bash
# Set upstream for current branch
git branch --set-upstream-to=origin/feature-branch

# Set upstream when pushing
git push -u origin feature-branch
```

### To delete branches:
```bash
# Delete merged branch (safe)
git branch -d feature-branch

# Force delete unmerged branch
git branch -D experimental-feature

# Delete remote branch
git push origin --delete feature-branch
```

### To prune stale remote references:
```bash
# Fetch and prune
git fetch --prune

# Prune without fetching
git remote prune origin

# Set automatic pruning
git config --global fetch.prune true
```

### To work with multiple remotes:
```bash
# Add another remote
git remote add upstream https://github.com/original/repo.git

# Fetch from specific remote
git fetch upstream

# Create local branch from upstream
git checkout -b feature-x upstream/feature-x
```

---

## Module 12: Stashing and Context Switching

### To stash changes:
```bash
# Basic stash
git stash

# Stash with message
git stash push -m "WIP: authentication refactor"

# Stash including untracked files
git stash -u
git stash push -u -m "Message"

# Stash everything including ignored files
git stash -a

# Stash specific files only
git stash push -m "Config changes" config.js settings.json

# Interactive stashing
git stash push -p
```

### To view stashes:
```bash
# List all stashes
git stash list

# Show stash contents
git stash show

# Show with full diff
git stash show -p

# Show specific stash
git stash show stash@{1}
git stash show -p stash@{1}
```

### To apply stashes:
```bash
# Apply most recent stash (keeps in list)
git stash apply

# Apply specific stash
git stash apply stash@{1}

# Apply and remove from list (pop)
git stash pop

# Pop specific stash
git stash pop stash@{1}
```

### To manage stashes:
```bash
# Drop most recent stash
git stash drop

# Drop specific stash
git stash drop stash@{1}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch new-branch-name
git stash branch experiment stash@{2}
```

---

## Module 13: Git Worktrees for Parallel Development

### To create worktrees:
```bash
# Create worktree in specified directory
git worktree add ../hotfix

# Create worktree with specific branch
git worktree add ../feature-auth feature-auth

# Create new branch in worktree
git worktree add -b new-feature ../new-feature

# Create from remote branch
git worktree add ../staging origin/staging
```

### To manage worktrees:
```bash
# List all worktrees
git worktree list

# List with detailed info
git worktree list --porcelain

# Remove worktree
git worktree remove ../project-hotfix

# Force remove
git worktree remove --force ../project-hotfix

# Clean up stale worktree metadata
git worktree prune

# Dry run prune
git worktree prune --dry-run
```

### To lock/unlock worktrees:
```bash
# Lock a worktree
git worktree lock ../project-important

# Unlock a worktree
git worktree unlock ../project-important
```

### To repair worktrees:
```bash
# Repair after moving worktree
git worktree repair
```

---

## Module 14: Submodules and Subtrees

### To work with submodules:
```bash
# Add submodule
git submodule add https://github.com/user/library.git lib/auth

# Clone with submodules
git clone --recurse-submodules https://github.com/user/project.git

# Initialize and update submodules
git submodule init
git submodule update

# Update all submodules
git submodule update --remote

# Update recursively
git submodule update --init --recursive

# View submodule status
git submodule status

# Remove submodule
git submodule deinit lib/auth
git rm lib/auth
```

### To work with subtrees:
```bash
# Add remote for subtree
git remote add auth-lib https://github.com/user/library.git

# Fetch subtree
git fetch auth-lib

# Add as subtree
git subtree add --prefix=lib/auth auth-lib main --squash

# Pull updates from subtree
git subtree pull --prefix=lib/auth auth-lib main --squash

# Push changes back to subtree
git subtree push --prefix=lib/auth auth-lib feature-branch
```

---

## Module 19: Git Hooks for Local Automation

### To configure hooks path:
```bash
# Use custom hooks directory
git config core.hooksPath .githooks
```

### Common hook files (in .git/hooks/):
- `pre-commit` - Runs before commit is created
- `prepare-commit-msg` - Modifies commit message before editor
- `commit-msg` - Validates commit message
- `post-commit` - Runs after commit is created
- `pre-push` - Runs before push to remote
- `post-merge` - Runs after successful merge

---

## Module 23: Advanced Git Recovery and Debugging

### To view and use the reflog:
```bash
# View reflog
git reflog

# View reflog for specific branch
git reflog show main

# View with dates
git reflog --date=iso

# View last N entries
git reflog -10
```

### To reset to previous states:
```bash
# Soft reset (keep changes staged)
git reset --soft HEAD~1

# Mixed reset (keep changes unstaged)
git reset HEAD~1

# Hard reset (discard all changes)
git reset --hard HEAD~1

# Reset to specific reflog entry
git reset --hard HEAD@{2}
```

### To revert commits safely:
```bash
# Revert last commit
git revert HEAD

# Revert specific commit
git revert abc123

# Revert multiple commits
git revert HEAD~3..HEAD

# Revert merge commit
git revert -m 1 <merge-commit>
```

### To use git bisect for debugging:
```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark known good commit
git bisect good v1.0.0

# Mark current as good or bad
git bisect good
git bisect bad

# End bisect
git bisect reset

# Automated bisect with test script
git bisect start HEAD v1.0.0
git bisect run npm test
```

### To use git blame:
```bash
# Show who last modified each line
git blame file.js

# Show specific line range
git blame -L 10,20 file.js

# Ignore whitespace changes
git blame -w file.js

# Show email addresses
git blame -e file.js
```

### To search commit history:
```bash
# Search commit messages
git log --grep="authentication"

# Search by author
git log --author="Jane"

# Search code changes
git log -S"function login"

# Search by date
git log --since="2 weeks ago"
git log --until="2024-01-01"

# Combine filters
git log --author="Jane" --since="1 month ago" --grep="fix"
```

### To find and recover lost content:
```bash
# Find dangling commits
git fsck --lost-found

# Show unreachable objects
git fsck --unreachable

# Verify repository integrity
git fsck

# Full verification
git fsck --full

# Aggressive garbage collection
git gc --aggressive
```

### To recover deleted branches or commits:
```bash
# Find in reflog and recreate branch
git branch recovered-branch <commit-hash>

# Or cherry-pick the commit
git cherry-pick <commit-hash>
```

---
