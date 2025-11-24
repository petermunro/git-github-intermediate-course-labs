# Lab 10: Understanding Branching Strategies

__Objective__: To practise implementing different branching strategies (GitHub Flow, Git Flow, and Trunk-Based Development)

## Prerequisites

- Completion of Labs 1-9 or equivalent Git knowledge
- Understanding of branching and merging
- Git installed on your machine
- A terminal or command prompt

## Step-by-Step Exercise

### Part 1: GitHub Flow

1. **Implement GitHub Flow**

    Experience the simplicity of GitHub Flow with one main branch. Use `git init` to create a repository.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir github-flow-lab
    cd github-flow-lab
    git init

    # Create initial project on main
    echo "# Web Application" > README.md
    git add README.md
    git commit -m "Initial project setup"

    # Feature 1: Create branch from main
    git checkout -b feature-user-login
    echo "function login() { /* authenticate */ }" > auth.js
    git add auth.js
    git commit -m "Add user login"

    # Merge to main with --no-ff
    git checkout main
    git merge feature-user-login --no-ff
    git branch -d feature-user-login

    # View history
    git log --oneline --graph
    ```

    **GitHub Flow:** Everything merges to main, which is always deployable.
    </details>

2. **Simulate Continuous Deployment**

    Add deployment tags to track releases.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create feature
    git checkout -b feature-dashboard
    echo "function render() { /* display */ }" > dashboard.js
    git add dashboard.js
    git commit -m "Add dashboard"

    # Merge and tag deployment
    git checkout main
    git merge feature-dashboard --no-ff
    git tag -a deploy-$(date +%Y%m%d) -m "Deploy dashboard"
    git branch -d feature-dashboard

    # View tags
    git log --oneline --decorate
    ```

    **Note:** Each merge to main triggers deployment.
    </details>

### Part 2: Trunk-Based Development

3. **Implement Trunk-Based Development**

    Experience rapid integration with feature flags.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cd ..
    mkdir trunk-based-lab
    cd trunk-based-lab
    git init

    # Initial commit
    echo "# Trunk-Based Development" > README.md
    echo "const features = {};" > feature-flags.js
    git add .
    git commit -m "Initial commit"

    # Very short-lived branch
    git checkout -b quick-fix
    echo "function fix() { return true; }" > utils.js
    git add utils.js
    git commit -m "Quick fix"

    # Merge immediately (same day)
    git checkout main
    git merge quick-fix
    git branch -d quick-fix

    # View linear history
    git log --oneline
    ```

    **Trunk-Based:** Merge frequently, use feature flags for incomplete work.
    </details>

4. **Use Feature Flags**

    Understand how flags enable trunk-based development.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create feature flags system
    cat > feature-flags.js << 'EOF'
const features = {
    newFeature: false,
    analytics: true
};

function isEnabled(feature) {
    return features[feature] === true;
}
EOF

    git add feature-flags.js
    git commit -m "Add feature flag system"

    # Add incomplete feature (disabled)
    echo "function newFeature() { if (isEnabled('newFeature')) { /* new */ } }" > feature.js
    git add feature.js
    git commit -m "Add new feature (flagged off)"

    # Later, enable it
    cat > feature-flags.js << 'EOF'
const features = {
    newFeature: true,  // Enabled!
    analytics: true
};

function isEnabled(feature) {
    return features[feature] === true;
}
EOF

    git add feature-flags.js
    git commit -m "Enable new feature"
    ```

    **Benefit:** Code integrated early, released when ready.
    </details>

### Part 3: Git Flow

5. **Implement Git Flow**

    Experience the structured Git Flow workflow.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cd ..
    mkdir git-flow-lab
    cd git-flow-lab
    git init

    # Create main branch
    echo "# Git Flow Project" > README.md
    git add README.md
    git commit -m "Initial commit"
    git tag v0.0.0

    # Create develop branch
    git checkout -b develop
    echo "Development branch" >> README.md
    git add README.md
    git commit -m "Start development"

    # View branches
    git log --oneline --graph --all
    ```

    **Git Flow:** Two permanent branches: `main` and `develop`.
    </details>

6. **Add Features Using Git Flow**

    Implement the feature branch workflow.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Feature 1
    git checkout develop
    git checkout -b feature/user-auth

    echo "function authenticate() { /* auth */ }" > auth.js
    git add auth.js
    git commit -m "Add authentication"

    # Merge to develop
    git checkout develop
    git merge feature/user-auth --no-ff
    git branch -d feature/user-auth

    # Feature 2
    git checkout -b feature/payment
    echo "function pay() { /* pay */ }" > payment.js
    git add payment.js
    git commit -m "Add payment"

    git checkout develop
    git merge feature/payment --no-ff
    git branch -d feature/payment

    # View history
    git log --oneline --graph --all
    ```

    **Git Flow:** Features develop on `develop`, not `main`.
    </details>

7. **Create a Release**

    Prepare and merge a release.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create release branch
    git checkout -b release/1.0.0 develop

    # Update version
    echo "v1.0.0" > version.txt
    git add version.txt
    git commit -m "Bump version to 1.0.0"

    # Merge to main
    git checkout main
    git merge release/1.0.0 --no-ff
    git tag -a v1.0.0 -m "Release 1.0.0"

    # Merge back to develop
    git checkout develop
    git merge release/1.0.0 --no-ff

    # Delete release branch
    git branch -d release/1.0.0

    # View result
    git log --oneline --graph --all
    ```

    **Release branch:** Preparation happens here, then merges to both branches.
    </details>

8. **Handle a Hotfix**

    Fix a critical production bug using Git Flow.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Critical bug in production
    git checkout main
    git checkout -b hotfix/1.0.1

    # Fix bug
    echo "function securityFix() { /* patch */ }" > security.js
    git add security.js
    git commit -m "Fix critical bug"

    echo "v1.0.1" > version.txt
    git add version.txt
    git commit -m "Bump version to 1.0.1"

    # Merge to main
    git checkout main
    git merge hotfix/1.0.1 --no-ff
    git tag -a v1.0.1 -m "Hotfix 1.0.1"

    # Merge to develop
    git checkout develop
    git merge hotfix/1.0.1 --no-ff

    # Delete hotfix branch
    git branch -d hotfix/1.0.1

    # View result
    git log --oneline --graph --all
    ```

    **Hotfix:** Branch from main, merge to both main and develop.
    </details>

## Challenge Exercises

### Challenge 1: Complete GitHub Flow Cycle

Create a repository and implement 5 features using GitHub Flow with proper branch naming, merge commits, and deployment tags.

<details>
<summary>Hint</summary>

For each feature: create branch from main → make changes → merge to main with `--no-ff` → tag deployment → delete branch.
</details>

### Challenge 2: Git Flow Release Cycle

Implement a complete Git Flow workflow with develop branch, 3 features, a release branch, and a hotfix.

<details>
<summary>Hint</summary>

Create develop, add features to develop, branch release from develop, merge release to main and develop, then create hotfix from main.
</details>

## Reflection Questions

1. Which strategy works best for continuous deployment?
2. How do feature flags change the way you merge code?
3. What are the trade-offs between simplicity and structure?
4. In what scenarios would you choose Git Flow over GitHub Flow?

## What You've Learned

- Implementing GitHub Flow for continuous deployment
- Using Trunk-Based Development with feature flags
- Following the complete Git Flow workflow
- Creating and managing release branches
- Handling hotfixes in different strategies
- Understanding when to use each strategy
- Comparing complexity versus control trade-offs

## Clean Up

```bash
cd ..
rm -rf github-flow-lab trunk-based-lab git-flow-lab
```
