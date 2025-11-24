# Lab 14: Submodules and Subtrees

__Objective__: To practise managing external dependencies using Git submodules and subtrees

## Prerequisites

- Completion of Labs 1-13 or equivalent Git knowledge
- Understanding of branching and repository management
- Git installed on your machine
- A terminal or command prompt

## About This Lab

In Module 14, you learnt about two approaches for including external repositories:

**Submodules:**
- **`git submodule add <url> <path>`** - Add external repository
- **`git submodule update --init`** - Initialise after cloning
- Stores a reference (commit hash) to external code
- External repo remains separate

**Subtrees:**
- **`git subtree add --prefix=<path> <remote> <branch>`** - Add external code
- **`git subtree pull --prefix=<path> <remote> <branch>`** - Update external code
- Copies files into your repository
- No special commands needed after cloning

## Step-by-Step Exercise

### Part 1: Git Submodules

1. **Create Repository Structure**

    Set up a main project and a library to use as a submodule.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create lab directory
    mkdir submodules-lab
    cd submodules-lab

    # Create external library
    mkdir auth-library
    cd auth-library
    git init

    echo "# Authentication Library" > README.md
    echo "function authenticate() { return true; }" > auth.js
    git add .
    git commit -m "Initial auth library"

    # Create main project
    cd ..
    mkdir main-project
    cd main-project
    git init

    echo "# Main Application" > README.md
    git add README.md
    git commit -m "Initial project"
    ```
    </details>

2. **Add Your First Submodule**

    Include the auth library as a submodule using `git submodule add`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Add submodule
    git submodule add ../auth-library lib/auth

    # View what was created
    cat .gitmodules
    ls lib/auth/

    # Commit the submodule
    git add .gitmodules lib/auth
    git commit -m "Add auth library as submodule"

    # Check submodule status
    git submodule status
    ```

    The `.gitmodules` file tracks submodule configuration.
    </details>

3. **Clone Repository with Submodules**

    Simulate cloning the project and initialising submodules.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cd ../..
    mkdir submodules-lab/clone-test
    cd submodules-lab/clone-test

    # Regular clone
    git clone ../main-project cloned-project
    cd cloned-project

    # Submodule directories exist but are empty!
    ls lib/auth/

    # Initialise and update submodules
    git submodule init
    git submodule update

    # Now files are present
    ls lib/auth/

    # Alternative: Clone with submodules in one step
    cd ..
    git clone --recurse-submodules ../main-project cloned-with-subs
    cd cloned-with-subs
    ls lib/auth/  # Submodules automatically populated
    ```

    Always use `--recurse-submodules` when cloning to get submodules automatically.
    </details>

4. **Update a Submodule**

    Make changes to the library and update the submodule reference.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Update the auth library
    cd ../../auth-library
    echo "function logout() { return true; }" >> auth.js
    git add auth.js
    git commit -m "Add logout function"

    # Update submodule in main project
    cd ../main-project/lib/auth
    git pull origin main

    # Return to main project root
    cd ../..

    # Git sees submodule as modified (new commit)
    git status

    # Commit the updated submodule reference
    git add lib/auth
    git commit -m "Update auth library to latest version"
    ```

    Two-step process: update in submodule, then commit in parent.
    </details>

### Part 2: Git Subtrees

5. **Create Project with Subtree**

    Use the subtree approach as an alternative to submodules.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create new project
    cd ..
    mkdir subtree-project
    cd subtree-project
    git init

    echo "# Subtree Demo Project" > README.md
    git add README.md
    git commit -m "Initial commit"

    # Add remote for library
    git remote add auth-lib ../auth-library

    # Fetch the library
    git fetch auth-lib

    # Add as subtree (--squash keeps history clean)
    git subtree add --prefix=lib/auth auth-lib main --squash

    # Check the result
    ls lib/auth/
    cat lib/auth/auth.js

    git log --oneline
    ```

    Subtree copies files into your repo - no special .gitmodules file.
    </details>

6. **Update a Subtree**

    Pull updates from the subtree source.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Make changes to auth library
    cd ../auth-library
    echo "function verifyToken() { return true; }" >> auth.js
    git add auth.js
    git commit -m "Add token verification"

    # Update subtree in main project
    cd ../subtree-project

    # Pull latest changes
    git subtree pull --prefix=lib/auth auth-lib main --squash

    # View updated files
    cat lib/auth/auth.js

    git log --oneline
    ```

    One command to update - simpler than submodules.
    </details>

7. **Compare Submodules vs Subtrees**

    Understand the trade-offs between the two approaches.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > ../comparison.md << 'EOF'
# Submodules vs Subtrees

## Submodules
**Pros:**
- Smaller repository size
- Clear separation of external code
- Each submodule has its own history

**Cons:**
- More complex workflow
- Requires --recurse-submodules when cloning
- Two-step update process
- Can be confusing for team members

## Subtrees
**Pros:**
- Simpler for contributors (no special commands)
- Works with standard Git commands
- Everything in one repository

**Cons:**
- Larger repository size
- External code mixed with project code
- History can become cluttered

## When to Use Each

**Use Submodules if:**
- You need clear boundaries between projects
- External code has its own release cycle
- Team is experienced with Git

**Use Subtrees if:**
- You want simplicity
- Contributors shouldn't worry about external dependencies
- You occasionally contribute back to external project
EOF

    cat ../comparison.md
    ```
    </details>

## Challenge Exercises

### Challenge 1: Multi-Submodule Project

Create a project with two submodules:
1. Authentication library
2. UI components library

Then clone the project and verify both submodules work.

<details>
<summary>Hint</summary>

Use `git submodule add` twice with different paths (e.g., `lib/auth` and `lib/ui`). When cloning, use `git clone --recurse-submodules` to get both automatically.
</details>

### Challenge 2: Subtree Workflow

Create a subtree-based project:
1. Add external library as subtree
2. Make changes to the subtree code
3. Pull updates from original library
4. Practice the complete workflow

<details>
<summary>Hint</summary>

Use `git subtree add` to include the library, then `git subtree pull` to get updates. Remember to use `--squash` to keep history clean.
</details>

## Reflection Questions

After completing this lab, consider:

1. **Choice:** When would you choose submodules over subtrees and vice versa?
2. **CI/CD:** How do submodules affect your CI/CD pipeline?
3. **Team:** What are the team knowledge requirements for each approach?
4. **Alternatives:** When should you use package managers (npm, pip) instead of Git submodules/subtrees?

## Clean Up

If you want to remove the lab directory:

```bash
cd ../../..
rm -rf submodules-lab
```

## Key Commands Summary

### Submodules
```bash
# Add submodule
git submodule add <url> <path>

# Clone with submodules
git clone --recurse-submodules <url>

# Initialise after regular clone
git submodule init
git submodule update

# Update submodules
git submodule update --remote

# View status
git submodule status
```

### Subtrees
```bash
# Add remote and subtree
git remote add <name> <url>
git subtree add --prefix=<path> <remote> <branch> --squash

# Update subtree
git subtree pull --prefix=<path> <remote> <branch> --squash

# Push changes back
git subtree push --prefix=<path> <remote> <branch>
```

## What You've Learnt

- Understanding Git submodules and their purpose
- Adding external repositories as submodules
- Cloning repositories with submodules
- Updating submodules to latest versions
- Understanding Git subtrees as an alternative
- Adding external code as subtrees
- Updating subtrees from source
- Comparing submodules vs subtrees
- Choosing the right approach for your project
- Understanding when to use package managers instead

Both submodules and subtrees solve the problem of including external code, but with different trade-offs in complexity and convenience!
