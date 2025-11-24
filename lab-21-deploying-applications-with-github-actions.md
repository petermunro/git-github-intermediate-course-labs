# Lab 21: Deploying Applications with GitHub Actions

__Objective__: To practise automating application deployments using GitHub Actions workflows

## Prerequisites

- Completion of Lab 20 or equivalent GitHub Actions knowledge
- Understanding of GitHub Actions workflows and YAML syntax
- A GitHub account with repository creation permissions
- A terminal or command prompt

## Important Note

This lab focuses on deployment workflows. Most exercises simulate deployments, but some can be used with real platforms like GitHub Pages, Vercel, or Netlify. Real deployments may incur costs depending on the platform.

## Step-by-Step Exercise

### Part 1: Basic Deployment Pipeline

1. **Create Deployable Application**

    Set up a simple static website project ready for deployment. Create `public/` directory with HTML, CSS, and JavaScript files.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir github-deploy-lab
    cd github-deploy-lab
    git init

    # Create static website
    mkdir -p public/css public/js

    cat > public/index.html << 'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Deployment Demo</title>
    <link rel="stylesheet" href="css/style.css">
</head>
<body>
    <header>
        <h1>GitHub Actions Deployment</h1>
    </header>
    <main>
        <p>This application is deployed automatically via GitHub Actions.</p>
        <div id="build-info"></div>
    </main>
    <script src="js/app.js"></script>
</body>
</html>
EOF

    cat > public/css/style.css << 'EOF'
body {
    font-family: Arial, sans-serif;
    max-width: 800px;
    margin: 50px auto;
    padding: 20px;
}

header {
    background: #667eea;
    color: white;
    padding: 20px;
    text-align: centre;
    border-radius: 10px;
}

main {
    margin-top: 30px;
}

#build-info {
    background: #f4f4f4;
    padding: 15px;
    margin-top: 20px;
    border-radius: 5px;
}
EOF

    cat > public/js/app.js << 'EOF'
document.addEventListener('DOMContentLoaded', function() {
    const buildInfo = document.getElementById('build-info');
    buildInfo.innerHTML = '<strong>Version:</strong> 1.0.0';
});
EOF

    # Create build script
    cat > build.js << 'EOF'
const fs = require('fs');
console.log('Building application...');

if (!fs.existsSync('dist')) {
    fs.mkdirSync('dist', { recursive: true });
}

// Copy public files to dist
function copyDir(src, dest) {
    fs.mkdirSync(dest, { recursive: true });
    const entries = fs.readdirSync(src, { withFileTypes: true });
    for (let entry of entries) {
        const srcPath = `${src}/${entry.name}`;
        const destPath = `${dest}/${entry.name}`;
        if (entry.isDirectory()) {
            copyDir(srcPath, destPath);
        } else {
            fs.copyFileSync(srcPath, destPath);
        }
    }
}

copyDir('public', 'dist');

// Create build metadata
const buildInfo = {
    version: process.env.VERSION || '1.0.0',
    buildDate: new Date().toISOString(),
    commit: process.env.GITHUB_SHA || 'local'
};

fs.writeFileSync('dist/build-info.json', JSON.stringify(buildInfo, null, 2));
console.log('Build complete!');
EOF

    cat > package.json << 'EOF'
{
  "name": "github-deploy-lab",
  "version": "1.0.0",
  "scripts": {
    "build": "node build.js",
    "test": "echo 'Tests passed' && exit 0"
  }
}
EOF

    cat > README.md << 'EOF'
# GitHub Actions Deployment Lab

Automated deployment with GitHub Actions.

## Features
- Automated deployment pipeline
- Build artifact generation
- Deployment verification
EOF

    git add .
    git commit -m "Initial commit: deployable application"
    git branch -M main
    ```

    Create a repository on GitHub and push when ready.
    </details>

2. **Create Basic Deployment Workflow**

    Build a workflow that tests, builds, and simulates deployment on push to main. Use separate jobs for each stage.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir -p .github/workflows

    cat > .github/workflows/deploy.yml << 'EOF'
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build application
        env:
          VERSION: ${{ github.run_number }}
          GITHUB_SHA: ${{ github.sha }}
        run: npm run build

      - name: Upload build
        uses: actions/upload-artifact@v3
        with:
          name: production-build
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download build
        uses: actions/download-artifact@v3
        with:
          name: production-build
          path: dist/

      - name: Deploy
        run: |
          echo "Deploying to production..."
          echo "Build artifacts:"
          ls -la dist/
          cat dist/build-info.json
          echo "Deployment complete!"
EOF

    git add .github/workflows/deploy.yml
    git commit -m "Add deployment workflow"
    git push origin main
    ```

    Check the **Actions** tab to watch the deployment pipeline!
    </details>

### Part 2: Real Deployments

3. **Deploy to GitHub Pages**

    Configure automatic deployment to GitHub Pages. Use the `actions/deploy-pages@v2` action with proper permissions.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/github-pages.yml << 'EOF'
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build site
        env:
          VERSION: ${{ github.run_number }}
          GITHUB_SHA: ${{ github.sha }}
        run: npm run build

      - name: Upload pages artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: dist/

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2

      - name: Display URL
        run: echo "Deployed to ${{ steps.deployment.outputs.page_url }}"
EOF

    git add .github/workflows/github-pages.yml
    git commit -m "Add GitHub Pages deployment"
    git push origin main
    ```

    Enable GitHub Pages in repository **Settings** → **Pages** → Source: **GitHub Actions**.
    Site will be live at `https://YOUR-USERNAME.github.io/github-deploy-lab/`
    </details>

4. **Add Health Checks After Deployment**

    Verify deployments succeed with automated health checks. Add verification steps that confirm the deployment is working.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/deploy-with-checks.yml << 'EOF'
name: Deploy with Checks

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build
        run: npm run build

      - name: Deploy
        run: |
          echo "Deploying application..."
          sleep 10
          echo "Deployment complete"

      - name: Wait for propagation
        run: sleep 20

      - name: Smoke tests
        run: |
          echo "Running smoke tests..."

          # Test 1: Check files exist
          if [ -f dist/index.html ]; then
            echo "✓ index.html exists"
          else
            echo "✗ index.html missing"
            exit 1
          fi

          if [ -f dist/js/app.js ]; then
            echo "✓ JavaScript files present"
          else
            echo "✗ JavaScript missing"
            exit 1
          fi

          if [ -f dist/css/style.css ]; then
            echo "✓ CSS files present"
          else
            echo "✗ CSS missing"
            exit 1
          fi

          echo "All smoke tests passed!"

      - name: Deployment summary
        if: always()
        run: |
          echo "=== Deployment Summary ==="
          echo "Status: ${{ job.status }}"
          echo "Commit: ${{ github.sha }}"
          echo "Actor: ${{ github.actor }}"
EOF

    git add .github/workflows/deploy-with-checks.yml
    git commit -m "Add deployment health checks"
    git push origin main
    ```

    Health checks verify the deployment before considering it successful.
    </details>

### Part 3: Advanced Deployment Strategies

5. **Create Rollback Workflow**

    Build a manually-triggered workflow to rollback to a previous version. Use `workflow_dispatch` with version input.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/rollback.yml << 'EOF'
name: Rollback

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Git tag or commit to rollback to'
        required: true
      reason:
        description: 'Reason for rollback'
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    steps:
      - name: Validate rollback
        run: |
          echo "=== Rollback Request ==="
          echo "Version: ${{ inputs.version }}"
          echo "Reason: ${{ inputs.reason }}"
          echo "Requested by: ${{ github.actor }}"

      - name: Checkout target version
        uses: actions/checkout@v3
        with:
          ref: ${{ inputs.version }}

      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build previous version
        run: |
          echo "Building version: ${{ inputs.version }}"
          npm run build

      - name: Backup current deployment
        run: |
          echo "Creating backup..."
          echo "Backup timestamp: $(date)"

      - name: Deploy previous version
        run: |
          echo "Rolling back to: ${{ inputs.version }}"
          sleep 10
          echo "Rollback complete"

      - name: Verify rollback
        run: |
          echo "Verifying rollback..."
          cat dist/build-info.json
          echo "Verification passed"

      - name: Notify
        if: always()
        run: |
          echo "Rollback status: ${{ job.status }}"
          echo "Version: ${{ inputs.version }}"
EOF

    git add .github/workflows/rollback.yml
    git commit -m "Add rollback workflow"
    git push origin main
    ```

    Test from **Actions** tab → **Rollback** → **Run workflow**
    </details>

6. **Add Deployment Notifications**

    Create workflow summaries and notifications for deployment status. Use `$GITHUB_STEP_SUMMARY` for rich output.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/deploy-with-notifications.yml << 'EOF'
name: Deploy with Notifications

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build
        run: npm run build

      - name: Deploy
        id: deploy
        run: |
          echo "Deploying..."
          sleep 10
          echo "deployment_url=https://example.com" >> $GITHUB_OUTPUT

      - name: Success notification
        if: success()
        run: |
          cat << NOTIFICATION
          ✅ Deployment Successful!

          Repository: ${{ github.repository }}
          Branch: ${{ github.ref_name }}
          Commit: ${{ github.sha }}
          URL: ${{ steps.deploy.outputs.deployment_url }}

          Deployed by: ${{ github.actor }}
          NOTIFICATION

      - name: Failure notification
        if: failure()
        run: |
          echo "❌ Deployment Failed!"
          echo "Check logs for details"

      - name: Create summary
        if: always()
        run: |
          cat >> $GITHUB_STEP_SUMMARY << SUMMARY
          ## Deployment Summary

          **Status:** ${{ job.status }}
          **Environment:** Production
          **Commit:** \`${{ github.sha }}\`
          **Branch:** ${{ github.ref_name }}
          **Triggered by:** ${{ github.actor }}
          **URL:** ${{ steps.deploy.outputs.deployment_url }}

          ### Changes
          $(git log -1 --pretty=format:"- %s (%h)")
          SUMMARY
EOF

    git add .github/workflows/deploy-with-notifications.yml
    git commit -m "Add deployment notifications"
    git push origin main
    ```

    View the rich summary in the workflow run details!
    </details>

### Part 4: Production-Ready Deployments

7. **Implement Deployment with Approval**

    Require manual approval before production deployment. Use GitHub environments with required reviewers.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/deploy-with-approval.yml << 'EOF'
name: Deploy with Approval

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
      - run: npm test
      - run: npm run build

      - uses: actions/upload-artifact@v3
        with:
          name: production-build
          path: dist/

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - name: Download build
        uses: actions/download-artifact@v3
        with:
          name: production-build
          path: dist/

      - name: Deploy to staging
        run: |
          echo "Deploying to staging..."
          sleep 5
          echo "Staging deployment complete"

      - name: Run tests
        run: echo "Integration tests passed"

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - name: Download build
        uses: actions/download-artifact@v3
        with:
          name: production-build
          path: dist/

      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          echo "This runs only after approval"
          sleep 10
          echo "Production deployment complete"

      - name: Verify
        run: echo "Production verification passed"
EOF

    git add .github/workflows/deploy-with-approval.yml
    git commit -m "Add deployment with approval"
    git push origin main
    ```

    Configure approval in **Settings** → **Environments** → **production** → Required reviewers.
    The workflow will pause until approved!
    </details>

8. **Create Canary Deployment**

    Implement gradual rollout with configurable traffic percentage. Use manual triggers with percentage input.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/canary-deploy.yml << 'EOF'
name: Canary Deployment

on:
  workflow_dispatch:
    inputs:
      percentage:
        description: 'Canary traffic percentage'
        required: true
        type: choice
        default: '10'
        options: ['10', '25', '50', '100']

jobs:
  deploy-canary:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build
        run: npm run build

      - name: Deploy canary
        run: |
          echo "Deploying canary: ${{ inputs.percentage }}% traffic"
          sleep 5
          echo "Canary deployed"

      - name: Monitor
        run: |
          echo "Monitoring canary deployment..."
          sleep 30

          # Simulate metrics
          error_rate=0.1
          response_time=150

          echo "Error rate: $error_rate%"
          echo "Response time: ${response_time}ms"

          if (( $(echo "$error_rate < 1.0" | bc -l) )); then
            echo "✓ Canary metrics acceptable"
          else
            echo "✗ Error rate too high"
            exit 1
          fi

      - name: Next steps
        if: success()
        run: |
          if [ "${{ inputs.percentage }}" = "100" ]; then
            echo "Full deployment complete!"
          else
            echo "Canary successful at ${{ inputs.percentage }}%"
            echo "Ready to increase traffic"
          fi

      - name: Auto rollback
        if: failure()
        run: |
          echo "Canary failed! Rolling back..."
          echo "Rollback complete"
EOF

    git add .github/workflows/canary-deploy.yml
    git commit -m "Add canary deployment"
    git push origin main
    ```

    Run with 10%, then gradually increase to 100%.
    </details>

## Challenge Exercises

### Challenge 1: Complete CI/CD Pipeline

Create a comprehensive deployment pipeline with:
- Automated testing and linting
- Build with version tagging
- Deploy to staging automatically
- Staging smoke tests
- Manual approval for production
- Production health checks
- Deployment notifications

<details>
<summary>Hint</summary>

Combine multiple jobs with `needs` dependencies. Use environments for staging and production. Add approval to production environment. Use artifacts to share builds. Include conditional steps for different environments.
</details>

### Challenge 2: Multi-Environment Deployment

Build a workflow that deploys to development, staging, and production sequentially:
- Development: Auto-deploy on every push
- Staging: Deploy after development succeeds
- Production: Require approval and additional checks
- Each environment has different verification steps

<details>
<summary>Hint</summary>

Use three deployment jobs with `needs` chain. Create three GitHub environments. Add different health checks per environment. Use `if` conditions for environment-specific behaviour.
</details>

## Reflection Questions

1. What's the difference between continuous delivery and continuous deployment?
2. How do you balance deployment speed with safety?
3. When should deployments require manual approval?
4. What metrics indicate a successful deployment?
5. How do canary deployments reduce risk?

## What You've Learned

- Creating automated deployment pipelines
- Building multi-stage deployment workflows
- Deploying to GitHub Pages
- Implementing health checks and verification
- Creating rollback workflows
- Adding deployment notifications
- Requiring manual approval for deployments
- Implementing canary deployments
- Managing deployment artifacts
- Using environment URLs for tracking

## Clean Up

```bash
cd ..
rm -rf github-deploy-lab
```

You can also delete the repository and disable workflows on GitHub if desired.
