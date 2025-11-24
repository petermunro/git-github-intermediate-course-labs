# Lab 22: GitHub Actions with Multiple Environments

__Objective__: To practise managing multiple deployment environments with GitHub Actions, including environment-specific secrets, protection rules, and deployment workflows

## Prerequisites

- Completion of Labs 20-21 or equivalent GitHub Actions knowledge
- Understanding of deployment workflows and environments
- A GitHub account with repository creation permissions
- A terminal or command prompt

## Important Note

This lab focuses on GitHub's environment features including protection rules, secrets, and deployment gates. Public repositories have full access to these features. Private repositories may have limitations depending on your GitHub plan.

## Step-by-Step Exercise

### Part 1: Setting Up Multiple Environments

1. **Create Multi-Environment Project**

    Set up a project with environment-specific configuration files. Create `config/environments/` directory with separate configs for development, staging, and production.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir github-environments-lab
    cd github-environments-lab
    git init

    # Create environment configurations
    mkdir -p config/environments

    cat > config/environments/development.json << 'EOF'
{
  "name": "development",
  "apiUrl": "http://localhost:3000",
  "debug": true,
  "features": {
    "newUI": true,
    "betaFeatures": true
  }
}
EOF

    cat > config/environments/staging.json << 'EOF'
{
  "name": "staging",
  "apiUrl": "https://api-staging.example.com",
  "debug": true,
  "features": {
    "newUI": true,
    "betaFeatures": false
  }
}
EOF

    cat > config/environments/production.json << 'EOF'
{
  "name": "production",
  "apiUrl": "https://api.example.com",
  "debug": false,
  "features": {
    "newUI": false,
    "betaFeatures": false
  }
}
EOF

    # Create simple build script
    cat > build.js << 'EOF'
const fs = require('fs');
const env = process.env.NODE_ENV || 'development';

console.log(`Building for: ${env}`);

// Load environment config
const config = JSON.parse(
    fs.readFileSync(`config/environments/${env}.json`, 'utf8')
);

// Create dist
if (!fs.existsSync('dist')) {
    fs.mkdirSync('dist');
}

// Write config
fs.writeFileSync('dist/config.json', JSON.stringify(config, null, 2));

// Build metadata
const buildInfo = {
    environment: env,
    buildDate: new Date().toISOString(),
    version: process.env.VERSION || '1.0.0',
    commit: process.env.GITHUB_SHA || 'local'
};

fs.writeFileSync('dist/build-info.json', JSON.stringify(buildInfo, null, 2));
console.log('Build complete!');
EOF

    cat > package.json << 'EOF'
{
  "name": "github-environments-lab",
  "version": "1.0.0",
  "scripts": {
    "build": "node build.js",
    "test": "echo 'Tests passed' && exit 0"
  }
}
EOF

    cat > README.md << 'EOF'
# GitHub Environments Lab

Multi-environment deployment with GitHub Actions.

## Environments
- Development: Latest changes
- Staging: Pre-production testing
- Production: Live environment
EOF

    git add .
    git commit -m "Initial commit: multi-environment project"
    git branch -M main
    ```

    Create a repository on GitHub and push.
    </details>

2. **Configure GitHub Environments**

    Set up three environments in GitHub with different protection rules. Navigate to Settings → Environments.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Push code first
    git push origin main
    ```

    **On GitHub:**

    1. **Go to Settings → Environments → New environment**

    2. **Create Development environment:**
       - Name: `development`
       - No protection rules
       - Add variable: `API_URL` = `https://api-dev.example.com`
       - Add secret: `API_KEY` = `dev-key-12345`

    3. **Create Staging environment:**
       - Name: `staging`
       - Optional: Wait timer = 5 minutes
       - Add variable: `API_URL` = `https://api-staging.example.com`
       - Add secret: `API_KEY` = `staging-key-67890`

    4. **Create Production environment:**
       - Name: `production`
       - Required reviewers: Add yourself
       - Wait timer: 10 minutes (optional)
       - Deployment branches: Only `main`
       - Add variable: `API_URL` = `https://api.example.com`
       - Add secret: `API_KEY` = `prod-key-abcdef`

    Each environment now has its own secrets and rules!
    </details>

### Part 2: Multi-Environment Workflows

3. **Create Automated Multi-Environment Pipeline**

    Build a workflow that deploys to appropriate environment based on branch. Development auto-deploys, production requires approval.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir -p .github/workflows

    cat > .github/workflows/multi-env-deploy.yml << 'EOF'
name: Multi-Environment Deploy

on:
  push:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm test

  deploy-staging:
    if: github.ref == 'refs/heads/main'
    needs: test
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build for staging
        env:
          NODE_ENV: staging
          VERSION: ${{ github.run_number }}
        run: npm run build

      - name: Deploy to staging
        env:
          API_KEY: ${{ secrets.API_KEY }}
          API_URL: ${{ vars.API_URL }}
        run: |
          echo "Deploying to staging..."
          echo "API URL: $API_URL"
          cat dist/build-info.json
          echo "Staging complete!"

  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build for production
        env:
          NODE_ENV: production
          VERSION: ${{ github.run_number }}
        run: npm run build

      - name: Deploy to production
        env:
          API_KEY: ${{ secrets.API_KEY }}
          API_URL: ${{ vars.API_URL }}
        run: |
          echo "Deploying to production..."
          echo "API URL: $API_URL"
          cat dist/build-info.json
          echo "Production complete!"
EOF

    git add .github/workflows/multi-env-deploy.yml
    git commit -m "Add multi-environment workflow"
    git push origin main
    ```

    Workflow deploys to staging, then waits for approval before production!
    </details>

4. **Create Manual Environment Selector**

    Build a workflow allowing manual deployment to any environment. Use `workflow_dispatch` with environment choice.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/manual-deploy.yml << 'EOF'
name: Manual Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy to'
        required: true
        type: choice
        options:
          - development
          - staging
          - production
      version:
        description: 'Version to deploy'
        default: 'latest'

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build for ${{ inputs.environment }}
        env:
          NODE_ENV: ${{ inputs.environment }}
          VERSION: ${{ inputs.version }}
        run: npm run build

      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
          API_URL: ${{ vars.API_URL }}
        run: |
          echo "=== Deployment Information ==="
          echo "Environment: ${{ inputs.environment }}"
          echo "Version: ${{ inputs.version }}"
          echo "Triggered by: ${{ github.actor }}"
          echo "API URL: $API_URL"
          cat dist/build-info.json
          echo "Deployment complete!"
EOF

    git add .github/workflows/manual-deploy.yml
    git commit -m "Add manual environment selector"
    git push origin main
    ```

    Go to **Actions** → **Manual Deploy** → **Run workflow** to test!
    </details>

### Part 3: Environment Management

5. **Create Environment Promotion Workflow**

    Build a workflow to promote a version from one environment to another. Validate promotion paths (e.g., no dev → prod).

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/promote.yml << 'EOF'
name: Promote Environment

on:
  workflow_dispatch:
    inputs:
      from-environment:
        description: 'Source environment'
        required: true
        type: choice
        options: [development, staging]
      to-environment:
        description: 'Target environment'
        required: true
        type: choice
        options: [staging, production]
      version:
        description: 'Version to promote'
        required: true

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Validate promotion
        run: |
          from="${{ inputs.from-environment }}"
          to="${{ inputs.to-environment }}"

          echo "Promotion: $from → $to"

          # Prevent dev → prod
          if [ "$from" = "development" ] && [ "$to" = "production" ]; then
            echo "❌ Cannot promote dev to prod directly"
            echo "Must go through staging first"
            exit 1
          fi

          if [ "$from" = "$to" ]; then
            echo "❌ Cannot promote to same environment"
            exit 1
          fi

          echo "✓ Valid promotion path"

  promote:
    needs: validate
    runs-on: ubuntu-latest
    environment:
      name: ${{ inputs.to-environment }}
      url: https://${{ inputs.to-environment }}.example.com
    steps:
      - uses: actions/checkout@v3
        with:
          ref: ${{ inputs.version }}

      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build for ${{ inputs.to-environment }}
        env:
          NODE_ENV: ${{ inputs.to-environment }}
          VERSION: ${{ inputs.version }}
        run: npm run build

      - name: Promote
        run: |
          echo "Promoting to ${{ inputs.to-environment }}..."
          echo "Version: ${{ inputs.version }}"
          echo "From: ${{ inputs.from-environment }}"
          cat dist/build-info.json
          echo "Promotion complete!"
EOF

    git add .github/workflows/promote.yml
    git commit -m "Add promotion workflow"
    git push origin main
    ```

    Validates promotion paths before deploying!
    </details>

6. **Add Environment Health Monitoring**

    Create a scheduled workflow that checks health of all environments. Use matrix to test multiple environments.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/health-check.yml << 'EOF'
name: Environment Health Check

on:
  schedule:
    - cron: '0 */6 * * *'  # Every 6 hours
  workflow_dispatch:

jobs:
  health-check:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [development, staging, production]
    steps:
      - name: Check ${{ matrix.environment }}
        run: |
          echo "Checking ${{ matrix.environment }}..."

          # Simulate health checks
          status_code=200
          response_time=150

          echo "Status: $status_code"
          echo "Response time: ${response_time}ms"

          if [ $status_code -eq 200 ] && [ $response_time -lt 1000 ]; then
            echo "✓ ${{ matrix.environment }} is healthy"
          else
            echo "✗ Health check failed"
            exit 1
          fi

      - name: Alert on failure
        if: failure()
        run: echo "🚨 Alert: ${{ matrix.environment }} health check failed!"

  report:
    needs: health-check
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Summary
        run: |
          echo "=== Health Report ==="
          echo "Timestamp: $(date -u)"
          echo "Status: ${{ needs.health-check.result }}"
EOF

    git add .github/workflows/health-check.yml
    git commit -m "Add health monitoring"
    git push origin main
    ```

    Regular health checks for all environments!
    </details>

### Part 4: Advanced Environment Operations

7. **Create Environment Comparison Report**

    Generate a report comparing configurations across environments. Show differences in settings.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/compare-environments.yml << 'EOF'
name: Compare Environments

on:
  workflow_dispatch:
  schedule:
    - cron: '0 9 * * MON'  # Weekly on Monday

jobs:
  compare:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Generate comparison
        run: |
          echo "=== Environment Comparison ===" > report.txt
          echo "Generated: $(date -u)" >> report.txt
          echo "" >> report.txt

          for env in development staging production; do
            echo "## $env" >> report.txt
            cat config/environments/${env}.json >> report.txt
            echo "" >> report.txt
          done

          cat report.txt

      - name: Create summary
        run: |
          cat >> $GITHUB_STEP_SUMMARY << SUMMARY
          # Environment Comparison

          ## Debug Settings
          - Development: $(jq -r '.debug' config/environments/development.json)
          - Staging: $(jq -r '.debug' config/environments/staging.json)
          - Production: $(jq -r '.debug' config/environments/production.json)

          ## Features
          ### Development
          \`\`\`json
          $(jq '.features' config/environments/development.json)
          \`\`\`

          ### Production
          \`\`\`json
          $(jq '.features' config/environments/production.json)
          \`\`\`
          SUMMARY

      - uses: actions/upload-artifact@v3
        with:
          name: environment-report
          path: report.txt
EOF

    git add .github/workflows/compare-environments.yml
    git commit -m "Add environment comparison"
    git push origin main
    ```

    View differences across environments in workflow summary!
    </details>

8. **Implement Environment Rollback**

    Create a quick rollback workflow for any environment. Store deployment history and enable fast recovery.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > .github/workflows/environment-rollback.yml << 'EOF'
name: Environment Rollback

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to rollback'
        required: true
        type: choice
        options: [staging, production]
      version:
        description: 'Version to rollback to'
        required: true
      reason:
        description: 'Rollback reason'
        required: true

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - name: Validate rollback
        run: |
          echo "=== Rollback Request ==="
          echo "Environment: ${{ inputs.environment }}"
          echo "Target version: ${{ inputs.version }}"
          echo "Reason: ${{ inputs.reason }}"
          echo "Requested by: ${{ github.actor }}"

      - uses: actions/checkout@v3
        with:
          ref: ${{ inputs.version }}

      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Build previous version
        env:
          NODE_ENV: ${{ inputs.environment }}
          VERSION: ${{ inputs.version }}
        run: npm run build

      - name: Backup current
        run: echo "Backup created: $(date)"

      - name: Deploy previous version
        run: |
          echo "Rolling back ${{ inputs.environment }}..."
          cat dist/build-info.json
          echo "Rollback complete!"

      - name: Verify
        run: echo "Rollback verification passed"

      - name: Create record
        run: |
          cat << RECORD
          === Rollback Record ===
          Environment: ${{ inputs.environment }}
          To version: ${{ inputs.version }}
          Reason: ${{ inputs.reason }}
          By: ${{ github.actor }}
          Time: $(date -u)
          RECORD
EOF

    git add .github/workflows/environment-rollback.yml
    git commit -m "Add environment rollback"
    git push origin main
    ```

    Quick rollback for any environment with audit trail!
    </details>

## Challenge Exercises

### Challenge 1: Complete Environment System

Create a comprehensive environment management system:
- Three environments with protection rules
- Automated dev → staging → prod pipeline
- Manual rollback for each environment
- Health monitoring every hour
- Weekly comparison reports
- Promotion workflow with validation

<details>
<summary>Hint</summary>

Combine workflows from this lab. Set up environment protection in GitHub Settings. Use `needs` for pipeline dependencies. Use `schedule` for monitoring and reports. Add `environment` to jobs requiring approval.
</details>

### Challenge 2: Blue-Green per Environment

Implement blue-green deployment for each environment:
- Each environment has blue and green slots
- Deploy to inactive slot first
- Run health checks
- Switch traffic
- Keep old slot for quick rollback

<details>
<summary>Hint</summary>

Use environment variables to track active slot (blue/green). Deploy to inactive, verify, then switch. Create separate GitHub environments for blue and green if needed. Store active slot state as artifact or in repository.
</details>

## Reflection Questions

1. How do environment protection rules improve deployment safety?
2. When should secrets be environment-specific versus repository-wide?
3. What's the benefit of environment promotion versus direct deployment?
4. How do you prevent environment configuration drift?
5. When should deployments require manual approval?

## What You've Learned

- Creating and configuring GitHub environments
- Setting up environment protection rules
- Managing environment-specific secrets and variables
- Building multi-environment deployment workflows
- Creating manual environment selectors
- Implementing environment promotion with validation
- Monitoring environment health
- Comparing configurations across environments
- Environment-specific rollback procedures
- Using deployment URLs for tracking

## Clean Up

```bash
cd ..
rm -rf github-environments-lab
```

You can also delete the repository and environments on GitHub if desired.
