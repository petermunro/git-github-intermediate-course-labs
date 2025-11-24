# Lab 18: Security and Credential Management

__Objective__: To practise secure authentication methods, credential management, and protecting sensitive data in Git and GitHub

## Prerequisites

- Completion of Labs 1-17 or equivalent Git knowledge
- A GitHub account
- Git installed on your machine
- A terminal or command prompt
- GPG installed (for commit signing)
- OpenSSH (usually pre-installed on macOS and Linux)

## Important Note

This lab involves creating SSH keys and GPG keys. These are cryptographic credentials that should be treated as sensitive information. Never share private keys, and store them securely.

## Step-by-Step Exercise

### Part 1: SSH Authentication

1. **Generate SSH Keys for GitHub**

    Create secure SSH keys for authentication. Use the `ssh-keygen` command with the Ed25519 algorithm (recommended for security and performance).

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Check for existing SSH keys
    ls -la ~/.ssh
    # Look for id_rsa.pub, id_ed25519.pub, or similar

    # Generate new SSH key (Ed25519 algorithm)
    ssh-keygen -t ed25519 -C "your.email@example.com"

    # When prompted:
    # - File location: Press Enter for default (~/.ssh/id_ed25519)
    # - Passphrase: Enter a strong passphrase (recommended)
    # - Confirm passphrase

    # View the generated files
    ls -la ~/.ssh/
    # Shows:
    # id_ed25519       (private key - never share)
    # id_ed25519.pub   (public key - safe to share)

    # View your public key
    cat ~/.ssh/id_ed25519.pub
    ```

    Output shows two keys: the private key (keep secret) and public key (share with GitHub).

    **Security:** Private key stays on your machine; public key goes to GitHub.
    </details>

2. **Add SSH Key to GitHub**

    Copy your public key and add it to GitHub. Start the SSH agent using `eval "$(ssh-agent -s)"`, then add your key with `ssh-add`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Start SSH agent
    eval "$(ssh-agent -s)"
    # Shows: Agent pid XXXXX

    # Add SSH key to agent
    ssh-add ~/.ssh/id_ed25519
    # Enter your passphrase if you set one

    # On macOS, persist key in keychain
    ssh-add --apple-use-keychain ~/.ssh/id_ed25519

    # Copy public key to clipboard
    # macOS:
    pbcopy < ~/.ssh/id_ed25519.pub
    # Linux:
    cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard
    # Or just display and copy manually:
    cat ~/.ssh/id_ed25519.pub

    # On GitHub:
    # 1. Click your profile picture → Settings
    # 2. Click "SSH and GPG keys" in sidebar
    # 3. Click "New SSH key"
    # 4. Title: "My Laptop" (descriptive name)
    # 5. Key type: Authentication Key
    # 6. Paste your public key
    # 7. Click "Add SSH key"

    # Test connection
    ssh -T git@github.com
    # Should see: "Hi username! You've successfully authenticated..."
    ```

    **Verification:** Successful SSH authentication message from GitHub.
    </details>

3. **Clone Repository Using SSH**

    Test your SSH setup by cloning a repository. Use the SSH URL format: `git@github.com:username/repo.git`

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir security-lab
    cd security-lab

    # Create a test repository on GitHub first
    # Then clone using SSH (not HTTPS)
    git clone git@github.com:YOUR-USERNAME/test-repo.git

    cd test-repo

    # Make a test change
    echo "Testing SSH authentication" > test.txt
    git add test.txt
    git commit -m "Test SSH push"
    git push origin main
    # Should work without password prompt!

    git log --oneline -1
    ```

    **Benefit:** No password prompts with SSH authentication.
    </details>

### Part 2: Commit Signing with GPG

4. **Generate GPG Key**

    Create a GPG key for signing commits. Use `gpg --full-generate-key` and select RSA 4096-bit with 1-year expiration.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Check if GPG is installed
    gpg --version

    # If not installed:
    # macOS: brew install gpg
    # Linux: sudo apt-get install gnupg

    # Generate new GPG key
    gpg --full-generate-key

    # Prompts:
    # 1. Key type: (1) RSA and RSA
    # 2. Key size: 4096
    # 3. Expiration: 1y (1 year recommended)
    # 4. Name: Your Name
    # 5. Email: your.email@example.com (must match GitHub email)
    # 6. Comment: (optional, e.g., "Git signing key")
    # 7. Passphrase: Enter strong passphrase

    # List keys to get key ID
    gpg --list-secret-keys --keyid-format=long

    # Output looks like:
    # sec   rsa4096/ABCD1234EFGH5678 2024-01-01 [SC] [expires: 2025-01-01]
    # The part after rsa4096/ is your key ID
    ```

    **Key ID:** Save your key ID (e.g., ABCD1234EFGH5678) for next steps.
    </details>

5. **Configure Git to Sign Commits**

    Tell Git to use your GPG key for signing. Use `git config --global user.signingkey` with your key ID.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Set signing key in Git (replace with your key ID)
    git config --global user.signingkey ABCD1234EFGH5678

    # Enable automatic signing for all commits
    git config --global commit.gpgsign true

    # Test signing
    mkdir test-signing
    cd test-signing
    git init

    echo "Test file" > test.txt
    git add test.txt
    git commit -m "Test signed commit"
    # Enter GPG passphrase when prompted

    # Verify commit is signed
    git log --show-signature

    # Shows: "gpg: Good signature from 'Your Name'"

    cd ..
    ```

    **Result:** All commits now automatically signed.
    </details>

6. **Add GPG Key to GitHub**

    Export your public GPG key and add it to GitHub to display the "Verified" badge on commits.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Export public GPG key (replace with your key ID)
    gpg --armor --export ABCD1234EFGH5678

    # Copy entire output, including BEGIN and END lines

    # On GitHub:
    # 1. Profile picture → Settings
    # 2. SSH and GPG keys
    # 3. Click "New GPG key"
    # 4. Paste your entire public key
    # 5. Click "Add GPG key"

    # Test with GitHub repository
    cd test-repo
    echo "Signed commit test" >> test.txt
    git add test.txt
    git commit -m "Test verified commit"
    git push origin main

    # On GitHub, view the commit - should see green "Verified" badge!
    ```

    **Verified:** GitHub shows "Verified" badge on signed commits.
    </details>

### Part 3: Protecting Secrets

7. **Create .gitignore for Sensitive Files**

    Prevent accidentally committing sensitive data. Create a comprehensive `.gitignore` file.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir secret-prevention
    cd secret-prevention
    git init

    # Create .gitignore for sensitive files
    cat > .gitignore << 'EOF'
# Environment files
.env
.env.local
.env.*.local

# Secrets
secrets.yml
secrets.json
*.key
*.pem

# Configuration with credentials
config/credentials.yml

# IDE files that might contain credentials
.vscode/settings.json
.idea/

# OS files
.DS_Store
EOF

    git add .gitignore
    git commit -m "Add gitignore for sensitive files"

    # Create example environment file
    cat > .env.example << 'EOF'
# Example environment variables
# Copy to .env and fill in your values

DATABASE_URL=postgresql://user:password@localhost:5432/dbname
API_KEY=your-api-key-here
SECRET_TOKEN=your-secret-token-here
EOF

    git add .env.example
    git commit -m "Add environment variable template"

    # Create actual .env (will be ignored)
    cat > .env << 'EOF'
DATABASE_URL=postgresql://admin:supersecret@prod.example.com:5432/production
API_KEY=sk_live_abc123realkey
SECRET_TOKEN=jwt_secret_very_secret
EOF

    # Verify .env is ignored
    git status
    # Should NOT show .env

    git log --oneline
    ```

    **Protection:** Sensitive files automatically excluded from commits.
    </details>

8. **Scan for Accidentally Committed Secrets**

    Check your repository for common secret patterns using `git grep`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Search for common secret patterns
    git grep -i 'api[_-]key'
    git grep -i 'password\s*='
    git grep -i 'token\s*='
    git grep -i 'secret\s*='

    # Search history for removed secrets
    git log -p -S 'password' --all

    # No secrets should be found in a clean repository
    echo "No secrets found - repository is clean!"

    cd ..
    ```

    **Vigilance:** Regular scanning helps catch accidental commits.
    </details>

### Part 4: Personal Access Tokens (Optional)

9. **Create Personal Access Token**

    For HTTPS authentication or API access, generate a personal access token in GitHub Settings.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # On GitHub:
    # 1. Profile picture → Settings
    # 2. Developer settings (bottom of sidebar)
    # 3. Personal access tokens → Tokens (classic)
    # 4. Generate new token (classic)
    # 5. Note: "Lab Exercise Token"
    # 6. Expiration: 30 days (recommended)
    # 7. Select scopes:
    #    ✓ repo (full repository access)
    # 8. Generate token
    # 9. COPY TOKEN NOW (only shown once!)

    # Configure credential helper to store token
    # macOS:
    git config --global credential.helper osxkeychain

    # Linux:
    git config --global credential.helper cache

    # Windows:
    git config --global credential.helper wincred

    echo "Token can be used for HTTPS operations"
    echo "Store securely - treat like a password!"
    ```

    **Token management:** Tokens expire and can be revoked, making them safer than passwords.
    </details>

## Challenge Exercises

### Challenge 1: Complete Security Audit

Perform a security audit of a repository:
1. Check for committed secrets in history
2. Verify .gitignore covers all sensitive files
3. Ensure SSH keys are properly configured
4. Verify commits are signed
5. Document findings

<details>
<summary>Hint</summary>

Use `git log -p -S 'password'` to search history, verify gitignore patterns with `git status`, check SSH with `ssh -T git@github.com`, verify signatures with `git log --show-signature`, and create a checklist of findings.
</details>

### Challenge 2: Secret Rotation Procedure

Create a procedure for rotating exposed secrets:
1. Identify what needs rotation (API keys, passwords, tokens)
2. Generate new secrets
3. Update applications and services
4. Revoke old secrets
5. Document the process

<details>
<summary>Hint</summary>

Create a step-by-step guide that includes: immediate revocation, generating replacements, updating configurations, testing with new secrets, and verifying old secrets no longer work. Include a template for documenting each rotation.
</details>

## Reflection Questions

After completing this lab, consider:

1. What are the trade-offs between SSH and HTTPS authentication?
2. Why is commit signing important for security?
3. How do you balance security with developer experience?
4. What should you do if you accidentally commit a secret?
5. How often should credentials be rotated?

## Clean Up

If you want to remove the lab directories:

```bash
cd ..
rm -rf security-lab test-repo test-signing secret-prevention
```

**DO NOT delete your SSH or GPG keys unless you created them only for this lab!**

## Key Commands Reference

```bash
# SSH Keys
ssh-keygen -t ed25519 -C "email@example.com"
ssh-add ~/.ssh/id_ed25519
ssh -T git@github.com

# GPG Keys
gpg --full-generate-key
gpg --list-secret-keys --keyid-format=long
gpg --armor --export KEY_ID

# Git Configuration
git config --global user.signingkey KEY_ID
git config --global commit.gpgsign true
git config --global credential.helper osxkeychain

# Verification
git log --show-signature
git grep -i 'api[_-]key'
```

## What You've Learned

- Generating and managing SSH keys for authentication
- Adding SSH keys to GitHub for password-less access
- Creating GPG keys for commit signing
- Configuring Git to automatically sign commits
- Adding GPG keys to GitHub for verified badges
- Creating comprehensive .gitignore patterns for sensitive files
- Scanning repositories for accidentally committed secrets
- Understanding personal access tokens
- Security best practices for Git workflows
- Balancing security with usability

Security is not a one-time setup but an ongoing practice that requires vigilance and regular review!
