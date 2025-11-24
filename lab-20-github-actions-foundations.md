# Lab 20: GitHub Actions Foundations

__Objective__: To practise creating and configuring GitHub Actions workflows for continuous integration

## Prerequisites

- Completion of Labs 1-19 or equivalent Git and GitHub knowledge
- A GitHub account with repository creation permissions
- Understanding of YAML syntax
- A terminal or command prompt

## Important Note

GitHub Actions provides free CI/CD minutes for public repositories and limited minutes for private repositories on free accounts. The exercises are designed to be minimal and efficient.

## Step-by-Step Exercise

### Part 1: Getting Started with Workflows

1. **Create Project and First Workflow**

    Set up a simple Node.js project and create your first GitHub Actions workflow. Use `mkdir`, `git init`, and create a `.github/workflows` directory.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir github-actions-lab
    cd github-actions-lab
    git init

    # Create simple Node.js project
    cat > package.json << 'EOF'
{
  "name": "github-actions-lab",
  "version": "1.0.0",
  "scripts": {
    "test": "node test.js"
  }
}
EOF

    # Create test script
    cat > test.js << 'EOF'
console.log('Running tests...');
const result = 2 + 2;
if (result !== 4) {
    console.error('Test failed');
    process.exit(1);
}
console.log('All tests passed!');
process.exit(0);
EOF

    # Create first workflow
    mkdir -p .github/workflows
    cat > .github/workflows/ci.yml << 'EOF'
name: CI

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Run tests
        run: npm test
EOF

    git add .
    git commit -m "Initial commit with CI workflow"
    git branch -M main

    # Push to GitHub (create repo first on GitHub)
    # git remote add origin git@github.com:YOUR-USERNAME/github-actions-lab.git
    # git push -u origin main
    ```

    After pushing, visit the **Actions** tab on GitHub to see your workflow run!
    </details>

2. **Add Multiple Jobs**

    Expand your workflow to include separate jobs for linting and testing. Create a `lint.js` script and add a `lint` job that runs before `test`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create lint script
    cat > lint.js << 'EOF'
const fs = require('fs');
console.log('Running linter...');
const files = fs.readdirSync('.');
files.forEach(file => {
    if (file.endsWith('.js')) {
        const content = fs.readFileSync(file, 'utf8');
        if (content.includes('console.log')) {
            console.log(`Info: console.log found in ${file}`);
        }
    }
});
console.log('Linting complete!');
EOF

    # Update package.json
    cat > package.json << 'EOF'
{
  "name": "github-actions-lab",
  "version": "1.0.0",
  "scripts": {
    "test": "node test.js",
    "lint": "node lint.js"
  }
}
EOF

    # Update workflow
    cat > .github/workflows/ci.yml << 'EOF'
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm test
EOF

    git add .
    git commit -m "Add linting job"
    git push origin main
    ```

    The `needs: lint` makes `test` wait for `lint` to complete successfully.
    </details>

### Part 2: Matrix Builds and Conditional Steps

3. **Test Across Multiple Node Versions**

    Use a matrix strategy to test your code on Node 16, 18, and 20 simultaneously. Modify the `test` job to use `strategy: matrix`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/ci.yml << 'EOF'
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        node-version: [16, 18, 20]
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
EOF

    git add .github/workflows/ci.yml
    git commit -m "Add matrix testing for multiple Node versions"
    git push origin main
    ```

    You'll see three test jobs running in parallel - one for each Node version!
    </details>

4. **Add Conditional Steps**

    Add steps that only run on the `main` branch or for pull requests. Use `if` conditions to control when steps execute.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/ci.yml << 'EOF'
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        node-version: [16, 18, 20]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test

      - name: Report (main only)
        if: github.ref == 'refs/heads/main' && matrix.node-version == 18
        run: echo "Tests passed on main branch!"

      - name: PR Comment
        if: github.event_name == 'pull_request'
        run: echo "Tests passed for pull request!"
EOF

    git add .github/workflows/ci.yml
    git commit -m "Add conditional steps"
    git push origin main
    ```

    Different messages appear for main branch pushes versus pull requests!
    </details>

### Part 3: Artifacts, Caching, and Manual Triggers

5. **Work with Build Artifacts**

    Create a build step that generates artifacts and upload them for later use or download. Use `actions/upload-artifact@v3`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create build script
    cat > build.js << 'EOF'
const fs = require('fs');
console.log('Building project...');
if (!fs.existsSync('dist')) {
    fs.mkdirSync('dist');
}
fs.writeFileSync('dist/app.js', 'console.log("Built application");');
fs.writeFileSync('dist/build-info.txt', `Build: ${new Date().toISOString()}`);
console.log('Build complete!');
EOF

    # Update package.json
    cat > package.json << 'EOF'
{
  "name": "github-actions-lab",
  "version": "1.0.0",
  "scripts": {
    "test": "node test.js",
    "lint": "node lint.js",
    "build": "node build.js"
  }
}
EOF

    # Create workflow with artifacts
    cat > .github/workflows/build.yml << 'EOF'
name: Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm run build

      - name: Upload artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-output
          path: dist/
          retention-days: 7

  verify:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-output
          path: downloaded/

      - name: Verify build
        run: |
          ls -la downloaded/
          cat downloaded/build-info.txt
EOF

    git add .
    git commit -m "Add build workflow with artifacts"
    git push origin main
    ```

    Artifacts appear in the workflow run summary and can be downloaded!
    </details>

6. **Implement Dependency Caching**

    Speed up workflows by caching Node.js dependencies. Add caching to the Node.js setup step using the built-in `cache` option.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create package-lock.json
    npm install

    # Update CI workflow with caching
    cat > .github/workflows/ci.yml << 'EOF'
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  test:
    runs-on: ubuntu-latest
    needs: lint
    strategy:
      matrix:
        node-version: [16, 18, 20]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test
EOF

    git add package-lock.json .github/workflows/ci.yml
    git commit -m "Add dependency caching"
    git push origin main
    ```

    The second run will be faster as dependencies are cached!
    </details>

7. **Create Manual Workflow Trigger**

    Build a workflow that can be triggered manually with input parameters. Use `workflow_dispatch` with inputs for environment selection.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/manual-build.yml << 'EOF'
name: Manual Build

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to build for'
        required: true
        type: choice
        options:
          - development
          - staging
          - production
      log-level:
        description: 'Log level'
        type: choice
        default: 'info'
        options:
          - debug
          - info
          - warning

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build for ${{ inputs.environment }}
        run: |
          echo "Building for: ${{ inputs.environment }}"
          echo "Log level: ${{ inputs.log-level }}"
          npm run build

      - name: Display info
        run: |
          echo "=== Build Information ==="
          echo "Environment: ${{ inputs.environment }}"
          echo "Triggered by: ${{ github.actor }}"
          echo "Branch: ${{ github.ref }}"
EOF

    git add .github/workflows/manual-build.yml
    git commit -m "Add manual workflow with inputs"
    git push origin main
    ```

    Go to **Actions** tab, select "Manual Build", and click "Run workflow" to test!
    </details>

### Part 4: Advanced Workflow Features

8. **Add Status Badges and PR Checks**

    Create a workflow specifically for pull request validation and add a status badge to your README.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create PR validation workflow
    cat > .github/workflows/pr-checks.yml << 'EOF'
name: PR Checks

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Check PR title
        run: |
          PR_TITLE="${{ github.event.pull_request.title }}"
          echo "PR Title: $PR_TITLE"
          if [ -z "$PR_TITLE" ]; then
            echo "Error: PR title cannot be empty"
            exit 1
          fi

      - name: Run checks
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - run: npm run lint
      - run: npm test
EOF

    # Update README with badge
    cat > README.md << 'EOF'
# GitHub Actions Lab

![CI](https://github.com/YOUR-USERNAME/github-actions-lab/workflows/CI/badge.svg)

A project for practising GitHub Actions workflows.

## Features
- Automated testing across multiple Node.js versions
- Build artifact generation
- Dependency caching
- Manual workflow triggers
EOF

    git add .
    git commit -m "Add PR checks and status badge"
    git push origin main
    ```

    Replace `YOUR-USERNAME` with your actual GitHub username!
    </details>

## Challenge Exercises

### Challenge 1: Complete CI/CD Pipeline

Create a comprehensive workflow that:
- Runs lint and test in parallel where possible
- Uses matrix strategy for Node 16, 18, 20
- Caches dependencies
- Only uploads artifacts on main branch
- Includes timeout protection (use `timeout-minutes`)

<details>
<summary>Hint</summary>

Use `needs` to control job order. Run `lint` and `test` in parallel by removing the `needs` dependency. Add a `build` job that depends on both. Use `if: github.ref == 'refs/heads/main'` for conditional artifact upload. Add `timeout-minutes: 10` to prevent stuck jobs.
</details>

### Challenge 2: Scheduled Workflow

Create a workflow that runs automatically every day at 2am UTC to verify the project still builds correctly. Use `schedule` with cron syntax.

<details>
<summary>Hint</summary>

```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # Runs at 02:00 UTC daily
  workflow_dispatch:      # Also allow manual trigger
```

Include steps to run tests, build, and generate a report artifact.
</details>

## Reflection Questions

1. How do GitHub Actions compare to other CI/CD tools?
2. When should jobs run in parallel versus sequentially?
3. What types of checks belong in CI versus pre-commit hooks?
4. Why is caching important for workflow performance?
5. When should you use matrix builds?

## What You've Learned

- Creating GitHub Actions workflows with YAML syntax
- Configuring workflow triggers (push, pull request, schedule, manual)
- Building multi-job workflows with dependencies
- Using actions from the GitHub Marketplace
- Implementing matrix builds for cross-version testing
- Adding conditional steps based on branch or event type
- Uploading and downloading artifacts
- Caching dependencies to improve performance
- Creating manually-triggered workflows with inputs
- Adding PR validation and status badges

## Clean Up

```bash
cd ..
rm -rf github-actions-lab
```

You can also disable or delete workflows in your GitHub repository settings if desired.
