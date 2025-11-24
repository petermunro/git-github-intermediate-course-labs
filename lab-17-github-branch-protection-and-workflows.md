# Lab 17: GitHub Branch Protection and Workflows

__Objective__: To practise configuring branch protection rules and implementing quality gates for collaborative development

## Prerequisites

- Completion of Labs 1-16 or equivalent Git knowledge
- A GitHub account with repository admin access
- Understanding of pull requests and code review
- Git installed on your machine
- A terminal or command prompt

## About This Lab

In Module 17, you learnt about GitHub branch protection features:

- **Branch Protection Rules** - Prevent direct pushes, require reviews
- **Required Status Checks** - Ensure CI passes before merge
- **CODEOWNERS** - Automatically assign reviewers
- **Linear History** - Enforce clean git history

Key protection settings:
- Require pull requests for merging
- Require approvals (1-6 reviewers)
- Require status checks to pass
- Require conversation resolution
- Restrict who can push to branches

**Note:** This lab requires repository admin access. Some features may require GitHub Pro or Organisation account.

## Step-by-Step Exercise

### Part 1: Basic Branch Protection

1. **Create Repository for Protection**

    Set up a repository to configure branch protection.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir branch-protection-lab
    cd branch-protection-lab
    git init

    # Create project structure
    echo "# Branch Protection Demo" > README.md
    mkdir src tests

    echo "function greet() { return 'Hello'; }" > src/app.js
    echo "// Tests here" > tests/app.test.js

    git add .
    git commit -m "Initial project setup"
    git branch -M main

    # Push to GitHub (create repository first)
    # git remote add origin git@github.com:YOUR-USERNAME/branch-protection-lab.git
    # git push -u origin main
    ```
    </details>

2. **Configure Basic Protection**

    Enable basic branch protection on main branch.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # On GitHub:
    # 1. Settings → Branches
    # 2. Click "Add rule"
    # 3. Branch name pattern: main
    # 4. Enable: "Require a pull request before merging"
    # 5. Click "Create"

    # Test protection - try direct push
    echo "// Direct change" >> src/app.js
    git add src/app.js
    git commit -m "Try direct push"
    git push origin main
    # Fails: "Protected branch update failed"

    # Correct approach: Use feature branch
    git reset --soft HEAD~1
    git checkout -b feature/test-protection

    git add src/app.js
    git commit -m "Add via pull request"
    git push -u origin feature/test-protection

    # On GitHub: Create pull request, then merge
    ```

    Direct pushes to main are now blocked - must use PRs.
    </details>

3. **Require Pull Request Reviews**

    Add review requirements before merging.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # On GitHub:
    # Settings → Branches → Edit rule for main
    # Enable: "Require approvals" and set to 1
    # Save changes

    # Create a feature
    git checkout main
    git pull origin main
    git checkout -b feature/add-feature

    echo "function newFeature() { }" >> src/app.js
    git add src/app.js
    git commit -m "Add new feature"
    git push -u origin feature/add-feature

    # On GitHub:
    # 1. Create PR
    # 2. Try to merge → Blocked: "Review required"
    # 3. Approve the PR (or have colleague approve)
    # 4. Now can merge
    ```

    Changes must be reviewed before merging.
    </details>

### Part 2: Status Checks and CI

4. **Set Up GitHub Actions Workflow**

    Create CI workflow and require it to pass.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create GitHub Actions workflow
    mkdir -p .github/workflows

    cat > .github/workflows/ci.yml << 'EOF'
    name: CI

    on:
      push:
        branches: [ main ]
      pull_request:
        branches: [ main ]

    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v3
          - name: Run tests
            run: |
              echo "Running tests..."
              test -f tests/app.test.js
              echo "✓ Tests passed"
    EOF

    git add .github/workflows/ci.yml
    git commit -m "Add CI workflow"
    git push origin main

    # On GitHub:
    # Settings → Branches → Edit main rule
    # Enable: "Require status checks to pass"
    # Add check: "test"
    # Save changes
    ```

    Only code that passes CI can be merged.
    </details>

5. **Test Status Check Protection**

    Create a PR and verify CI must pass.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    git checkout -b feature/test-ci

    # Make a change
    echo "function anotherFeature() { }" >> src/app.js
    git add src/app.js
    git commit -m "Add another feature"
    git push -u origin feature/test-ci

    # On GitHub:
    # 1. Create PR
    # 2. CI runs automatically
    # 3. Can't merge until CI passes
    # 4. Once CI passes, can merge (after approval)
    ```

    Status checks ensure code quality before merge.
    </details>

### Part 3: CODEOWNERS and Advanced Protection

6. **Create CODEOWNERS File**

    Automatically assign reviewers based on file paths.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    git checkout main
    git pull origin main

    mkdir -p .github
    cat > .github/CODEOWNERS << 'EOF'
    # Default owners
    * @YOUR-USERNAME

    # Source code requires review
    /src/ @YOUR-USERNAME

    # Tests require thorough review
    /tests/ @YOUR-USERNAME

    # CI configuration is critical
    /.github/ @YOUR-USERNAME
    EOF

    git add .github/CODEOWNERS
    git commit -m "Add CODEOWNERS file"
    git push origin main

    # On GitHub:
    # Settings → Branches → Edit main rule
    # Enable: "Require review from Code Owners"
    # Save changes
    ```

    Right people automatically review relevant changes.
    </details>

7. **Require Linear History**

    Enforce clean, linear git history.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # On GitHub:
    # Settings → Branches → Edit main rule
    # Enable: "Require linear history"
    # Save changes

    # This disables merge commits
    # Only "Squash and merge" or "Rebase and merge" allowed

    # Test with a feature
    git checkout -b feature/linear-test

    echo "// Update 1" >> src/app.js
    git add src/app.js
    git commit -m "Update 1"

    echo "// Update 2" >> src/app.js
    git add src/app.js
    git commit -m "Update 2"

    git push -u origin feature/linear-test

    # On GitHub:
    # 1. Create PR
    # 2. "Create a merge commit" is disabled
    # 3. Must use "Squash and merge" or "Rebase and merge"
    # 4. Choose "Squash and merge"

    git checkout main
    git pull origin main
    git log --oneline --graph -5
    ```

    Linear history without merge commits keeps history clean.
    </details>

8. **Document Branch Protection Policy**

    Create documentation for your team.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir -p docs
    cat > docs/branch-policy.md << 'EOF'
# Branch Protection Policy

## Main Branch Protection

### Required Checks
- ✅ All CI tests must pass
- ✅ At least 1 approval required
- ✅ Code owner review required
- ✅ Conversations must be resolved

### Restrictions
- ❌ No direct pushes
- ❌ No force pushes
- ❌ No deletions
- ✅ Must use pull requests

### Merge Requirements
- ✅ Status checks must pass
- ✅ Approvals must be current
- ✅ Linear history (squash or rebase only)

## Development Workflow

1. Create feature branch from main
2. Make changes and commit
3. Push feature branch
4. Create pull request
5. Wait for CI to pass
6. Get code owner approval
7. Squash and merge
8. Delete feature branch

## Emergency Process

For critical hotfixes:
1. Create hotfix PR
2. Fast-track review
3. Merge with approval
4. Document in post-mortem
EOF

    git add docs/branch-policy.md
    git commit -m "Add branch protection policy documentation"
    git push origin main
    ```

    Document your protection rules for the team.
    </details>

9. **Test Complete Protection Workflow**

    Verify all protection rules work together.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create comprehensive feature
    git checkout -b feature/complete-test

    # Add new feature with tests
    echo "function validate(input) { return input !== null; }" > src/validator.js

    cat > tests/validator.test.js << 'EOF'
    // Test validation
    console.log('Tests would run here');
    EOF

    git add src/validator.js tests/validator.test.js
    git commit -m "Add input validation

Implements validation:
- Null check
- Tests included

Refs #123"

    git push -u origin feature/complete-test

    # Create PR on GitHub
    # Verify all protections work:
    # ✅ CI runs automatically
    # ✅ Code owner assigned
    # ✅ Can't merge without approval
    # ✅ Can't merge until CI passes
    # ✅ Must use squash/rebase merge

    # After approval and CI pass, merge with "Squash and merge"
    ```

    All protection mechanisms working together.
    </details>

## Challenge Exercises

### Challenge 1: Multi-Environment Protection

Configure different protection rules for:
- `main` (production): Strictest rules
- `develop` (staging): Medium rules
- `feature/*` (development): No protection

<details>
<summary>Hint</summary>

Create separate branch protection rules for each pattern. Main should require 2 approvals and all checks; develop should require 1 approval and status checks; feature branches need no protection.
</details>

### Challenge 2: CODEOWNERS Strategy

Create a sophisticated CODEOWNERS file for a project with:
- Frontend code
- Backend code
- Database migrations
- CI/CD configuration
- Documentation

Assign different reviewers to each area.

<details>
<summary>Hint</summary>

Use path patterns like `/frontend/`, `*.sql`, `.github/workflows/` to match different file types. Assign multiple owners where needed using `@user1 @user2` syntax.
</details>

## Reflection Questions

After completing this lab, consider:

1. **Quality vs Speed:** How do branch protection rules improve code quality while potentially slowing velocity?
2. **Approval Count:** When should you require 1 vs 2 approvals?
3. **Status Checks:** What checks should be required vs optional?
4. **Emergencies:** How do you handle emergency hotfixes with strict protection?

## Clean Up

If you want to remove the lab directory:

```bash
cd ..
rm -rf branch-protection-lab
```

## Key Protection Settings

### Required for All Teams
- Require pull requests before merging
- Require at least 1 approval
- Require status checks to pass

### Recommended
- Require review from code owners
- Dismiss stale reviews when new commits pushed
- Require linear history
- Require conversation resolution

### Advanced (if needed)
- Restrict who can push
- Require signed commits
- Include administrators in restrictions

## What You've Learnt

- Configuring basic branch protection rules
- Requiring pull request reviews
- Setting up CI status checks as requirements
- Creating and using CODEOWNERS files
- Enforcing linear history
- Documenting branch protection policies
- Testing complete protection workflows
- Balancing security with developer productivity
- Understanding different protection levels for different branches
- Creating emergency bypass procedures

Branch protection rules are essential for maintaining code quality and preventing accidental changes to critical branches!
