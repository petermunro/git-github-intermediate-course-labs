# Lab 15: GitHub Collaboration Fundamentals

__Objective__: To practise GitHub collaboration workflows including forking, pull requests, and managing remote repositories

## Prerequisites

- Completion of Labs 1-14 or equivalent Git knowledge
- A GitHub account (sign up at https://github.com if needed)
- Git configured with your GitHub credentials
- Understanding of branches and remote repositories
- A terminal or command prompt

## About This Lab

In Module 15, you learnt about GitHub collaboration features:

- **Pull Requests** - Propose changes for review
- **Forks** - Personal copy of someone else's repository
- **Remotes** - Connections to GitHub repositories
- **origin** vs **upstream** - Your fork vs original repository

Key commands:
- **`git remote add origin <url>`** - Connect to your GitHub repository
- **`git push -u origin <branch>`** - Push and set tracking
- **`git remote add upstream <url>`** - Connect to original repository (for forks)

## Step-by-Step Exercise

### Part 1: Basic GitHub Workflow

1. **Create Local Repository**

    Set up a project to push to GitHub.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir github-collab-lab
    cd github-collab-lab
    git init

    # Create initial project
    echo "# Collaboration Project" > README.md
    echo "A project for practising GitHub collaboration" >> README.md

    git add README.md
    git commit -m "Initial commit"
    ```
    </details>

2. **Push to GitHub**

    Create a GitHub repository and push your code. On GitHub: Click "+" → "New repository" → Name it "github-collab-lab" → Create repository.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Add remote (replace YOUR-USERNAME)
    git remote add origin git@github.com:YOUR-USERNAME/github-collab-lab.git

    # Verify remote
    git remote -v

    # Push to GitHub
    git branch -M main
    git push -u origin main

    # Verify on GitHub that your code is there
    ```

    The `-u` flag sets up tracking between local and remote branches.
    </details>

3. **Create Feature Branch and Pull Request**

    Make a change on a feature branch and create a pull request.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create feature branch
    git checkout -b feature/add-auth

    # Add authentication feature
    mkdir src
    echo "function login(username, password) { /* TODO */ }" > src/auth.js

    # Commit changes
    git add src/auth.js
    git commit -m "Add authentication module"

    # Push feature branch
    git push -u origin feature/add-auth

    # On GitHub:
    # 1. Go to your repository
    # 2. Click "Compare & pull request"
    # 3. Add title: "Add user authentication"
    # 4. Add description explaining the feature
    # 5. Click "Create pull request"
    ```

    Pull requests allow code review before merging.
    </details>

4. **Add Pull Request Template**

    Create a template for consistent pull requests.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Return to main branch
    git checkout main
    git pull origin main

    # Create .github directory
    mkdir -p .github

    # Create PR template
    cat > .github/PULL_REQUEST_TEMPLATE.md << 'EOF'
## Description
<!-- Briefly describe what this PR does -->

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Tests pass
- [ ] Manual testing completed

## Related Issues
<!-- Reference issues: Fixes #123 -->
EOF

    git add .github/PULL_REQUEST_TEMPLATE.md
    git commit -m "Add pull request template"
    git push origin main
    ```

    Templates ensure consistent information in all PRs.
    </details>

### Part 2: Fork Workflow

5. **Simulate Fork and Upstream**

    Practise the open source contribution model.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Simulate by creating a second directory
    cd ..
    mkdir github-collab-fork
    cd github-collab-fork
    git init

    # Add both remotes
    git remote add origin git@github.com:YOUR-USERNAME/github-collab-lab-fork.git
    git remote add upstream git@github.com:YOUR-USERNAME/github-collab-lab.git

    # Fetch from upstream
    git fetch upstream

    # Create main branch from upstream
    git checkout -b main upstream/main

    # Verify remotes
    git remote -v
    # Shows both origin (your fork) and upstream (original)

    # Create feature branch
    git checkout -b feature/error-handling

    echo "class ValidationError extends Error { }" > error-handling.js
    git add error-handling.js
    git commit -m "Add custom error handling"

    # Push to your fork
    git push -u origin feature/error-handling

    # Create PR from your fork to upstream repository
    ```

    Fork workflow: origin is your fork, upstream is the original.
    </details>

6. **Sync Fork with Upstream**

    Keep your fork up to date with the original repository.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Fetch latest changes from upstream
    git fetch upstream

    # Switch to main
    git checkout main

    # Merge upstream changes
    git merge upstream/main

    # Push updates to your fork
    git push origin main

    # Update feature branch with latest main
    git checkout feature/error-handling
    git rebase main

    # Force push updated feature branch
    git push --force-with-lease origin feature/error-handling
    ```

    Regular syncing prevents merge conflicts.
    </details>

### Part 3: Managing Pull Requests

7. **Use Draft Pull Requests**

    Get early feedback with draft PRs.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Return to original repository
    cd ../github-collab-lab

    # Create experimental feature
    git checkout -b feature/experimental-cache

    echo "const cache = new Map();" > cache.js
    echo "// TODO: Add expiration" >> cache.js

    git add cache.js
    git commit -m "WIP: Add caching module (draft)"
    git push -u origin feature/experimental-cache

    # On GitHub:
    # 1. Create PR
    # 2. Select "Create draft pull request"
    # 3. Add note: "Early draft - feedback welcome"
    # 4. When ready, click "Ready for review"
    ```

    Draft PRs get architectural feedback before investing time.
    </details>

8. **Merge Pull Request and Clean Up**

    Complete the PR workflow by merging and cleaning up branches.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # On GitHub:
    # 1. Go to your merged PR
    # 2. Click "Merge pull request"
    # 3. Choose merge strategy (Merge, Squash, or Rebase)
    # 4. Confirm merge
    # 5. Click "Delete branch"

    # Locally, update main branch
    git checkout main
    git pull origin main

    # Delete merged feature branch locally
    git branch -d feature/add-auth

    # Prune deleted remote branches
    git fetch --prune

    # View clean branch list
    git branch -a
    ```

    Always clean up merged branches to keep repository tidy.
    </details>

## Challenge Exercises

### Challenge 1: Complete Fork Workflow

Simulate a complete open source contribution:
1. Create and clone a fork
2. Create a feature branch
3. Make changes and commit
4. Push to your fork
5. Create PR to upstream
6. Sync your fork when upstream changes

<details>
<summary>Hint</summary>

Set up two remotes: `origin` (your fork) and `upstream` (original). Use `git fetch upstream` and `git merge upstream/main` to stay synced.
</details>

### Challenge 2: Pull Request Best Practices

Create a pull request that demonstrates best practices:
1. Link to an issue
2. Include clear description
3. Add relevant reviewers
4. Ensure CI passes
5. Address review feedback

<details>
<summary>Hint</summary>

Use keywords like "Fixes #123" in your PR description to auto-close issues. Make sure commits are logical and commit messages are clear.
</details>

## Reflection Questions

After completing this lab, consider:

1. **Fork vs Direct:** When should you use the fork workflow versus direct repository access?
2. **PR Templates:** How do pull request templates improve team collaboration?
3. **Review Process:** What information should you include in a pull request description?
4. **Draft PRs:** How do draft pull requests improve the development process?

## Clean Up

If you want to remove the lab directories:

```bash
cd ..
rm -rf github-collab-lab github-collab-fork
```

## Key Commands Summary

```bash
# Remote management
git remote add <name> <url>           # Add remote repository
git remote -v                         # List remotes
git remote remove <name>              # Remove remote

# Working with remotes
git fetch <remote>                    # Fetch from remote
git pull <remote> <branch>            # Fetch and merge
git push <remote> <branch>            # Push to remote
git push -u <remote> <branch>         # Push and set tracking

# Fork workflow
git remote add upstream <url>         # Add original repository
git fetch upstream                    # Fetch from original
git merge upstream/main               # Merge upstream changes
git push origin main                  # Push to your fork

# Branch management
git branch -d <branch>                # Delete local branch
git push origin --delete <branch>     # Delete remote branch
git fetch --prune                     # Remove deleted remote refs
```

## What You've Learnt

- Creating and managing GitHub repositories
- Pushing local repositories to GitHub
- Creating and managing pull requests
- Using pull request templates for consistency
- Understanding the fork workflow
- Syncing forks with upstream repositories
- Using draft pull requests for early feedback
- Merging pull requests and cleaning up branches
- Working with multiple remote repositories
- Best practices for GitHub collaboration

GitHub transforms Git from version control into a powerful collaboration platform!
