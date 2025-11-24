# Lab 16: Code Review with GitHub

__Objective__: To practise effective code review workflows using GitHub's review features including comments, suggestions, and review states

## Prerequisites

- Completion of Labs 1-15 or equivalent Git knowledge
- A GitHub account with an existing repository
- Understanding of pull requests and collaboration workflows
- Git installed on your machine
- A terminal or command prompt

## About This Lab

In Module 16, you learnt about GitHub's code review features:

- **Line Comments** - Comment on specific code lines
- **Suggestions** - Propose exact code changes
- **Review States** - Comment, Approve, or Request Changes
- **Review Conversations** - Track feedback threads

Key review practices:
- Review states: **Comment** (feedback), **Approve** (ready to merge), **Request Changes** (blockers)
- Categorise feedback: **Blockers** (must fix), **Important** (should fix), **Suggestions** (nice to have)
- Always include positive feedback alongside critical comments

## Step-by-Step Exercise

### Part 1: Setting Up for Review

1. **Create Practice Repository**

    Set up a repository with code to review.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir code-review-lab
    cd code-review-lab
    git init

    # Create project structure
    mkdir src

    # Add code with review points
    cat > src/user-service.js << 'EOF'
    function createUser(name, email) {
        // TODO: Add validation
        return {name: name, email: email};
    }

    module.exports = { createUser };
    EOF

    git add .
    git commit -m "Initial user service"
    git branch -M main

    # Push to GitHub (create repository first)
    # git remote add origin git@github.com:YOUR-USERNAME/code-review-lab.git
    # git push -u origin main
    ```
    </details>

2. **Create Pull Request with Review Points**

    Create a feature with intentional areas for improvement.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create feature branch
    git checkout -b feature/user-auth

    cat > src/auth-service.js << 'EOF'
    function login(email, password) {
        // TODO: Add password validation
        return {success: true, token: "abc123"};
    }

    function validatePassword(password) {
        return password.length > 6;  // Too simple
    }

    module.exports = { login, validatePassword };
    EOF

    git add src/auth-service.js
    git commit -m "Add authentication service

Ready for review."

    git push -u origin feature/user-auth

    # On GitHub: Create pull request
    ```

    Review points: weak password validation, hardcoded token, no error handling.
    </details>

### Part 2: Performing Code Review

3. **Perform Self-Review**

    Review your own code before requesting team review.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Review your changes
    git diff main...feature/user-auth

    # Make improvements
    cat > src/auth-service.js << 'EOF'
    function login(email, password) {
        if (!email || !password) {
            throw new Error('Email and password required');
        }

        if (!validatePassword(password)) {
            return {success: false, error: 'Invalid password'};
        }

        // TODO: Implement actual authentication
        return {success: true, token: generateToken()};
    }

    function validatePassword(password) {
        // Minimum 8 characters, letter and number
        return password.length >= 8 &&
               /[a-zA-Z]/.test(password) &&
               /[0-9]/.test(password);
    }

    function generateToken() {
        return Math.random().toString(36).substring(7);
    }

    module.exports = { login, validatePassword };
    EOF

    git add src/auth-service.js
    git commit -m "Self-review improvements

- Add input validation
- Improve password requirements
- Extract token generation"

    git push origin feature/user-auth
    ```

    Always self-review before requesting team review.
    </details>

4. **Add Line Comments as Reviewer**

    Provide specific, actionable feedback.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # On GitHub, in the "Files changed" tab:
    # 1. Click the line number to add a comment
    # 2. Add comments like:

    # Line 5 (Input Validation):
    # ✅ Good: Input validation at start of function

    # Line 11 (Error message):
    # 💭 Consider: Should we return the same error for
    # "user not found" vs "invalid password" to prevent
    # user enumeration attacks?

    # Line 15 (Password validation):
    # 🔍 Suggestion: Consider using a library like 'validator'
    # for more comprehensive password validation

    # Line 25 (Token generation):
    # ⚠️ Important: This token generation is not cryptographically
    # secure. Use crypto.randomBytes() instead
    ```

    Use emojis or prefixes to categorise feedback severity.
    </details>

5. **Use GitHub Suggestions**

    Propose exact code changes that can be applied directly.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # On GitHub, add a suggestion:
    # Click "+" on line, then use suggestion syntax:

    # ```suggestion
    # const crypto = require('crypto');
    #
    # function generateToken() {
    #     return crypto.randomBytes(32).toString('hex');
    # }
    # ```

    # The author can click "Commit suggestion" to apply it directly
    ```

    Suggestions make it easy for authors to apply fixes.
    </details>

### Part 3: Review States and Feedback

6. **Submit Different Review States**

    Practise using Comment, Approve, and Request Changes appropriately.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # On GitHub, click "Review changes" button

    # COMMENT (general feedback, no decision):
    # "Overall approach looks good. A few suggestions for improvement."

    # APPROVE (ready to merge):
    # "Nice work! Code is solid. Minor suggestions can be addressed
    # now or in follow-up PR."

    # REQUEST CHANGES (blockers must be fixed):
    # "Changes required before merge:
    # 🚫 Token generation is insecure (line 25)
    # 🚫 No error handling for email validation
    # Please fix these critical issues."
    ```

    Use REQUEST CHANGES only for critical issues that must be fixed.
    </details>

7. **Address Review Feedback**

    Respond to and fix review comments.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Address critical feedback
    cat > src/auth-service.js << 'EOF'
    const crypto = require('crypto');

    function login(email, password) {
        if (!email || !password) {
            throw new Error('Email and password required');
        }

        // Validate email format
        if (!validateEmail(email)) {
            return {success: false, error: 'Invalid email'};
        }

        if (!validatePassword(password)) {
            return {success: false, error: 'Invalid password'};
        }

        return {success: true, token: generateToken()};
    }

    function validateEmail(email) {
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    }

    function validatePassword(password) {
        return password.length >= 8 &&
               /[a-zA-Z]/.test(password) &&
               /[0-9]/.test(password);
    }

    function generateToken() {
        return crypto.randomBytes(32).toString('hex');
    }

    module.exports = { login, validatePassword, validateEmail };
    EOF

    git add src/auth-service.js
    git commit -m "Address review feedback

✅ Use crypto.randomBytes for secure tokens
✅ Add email validation
✅ Improve error handling

All blocking issues addressed."

    git push origin feature/user-auth

    # On GitHub:
    # - Reply to comments: "Fixed in commit abc123"
    # - Mark conversations as "Resolved"
    # - Re-request review
    ```

    Address all feedback, acknowledge reviewers, and re-request review.
    </details>

8. **Categorise Feedback Appropriately**

    Distinguish between blocking and non-blocking feedback.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > feedback-categories.md << 'EOF'
# Code Review Feedback Categories

## 🚫 BLOCKERS (Must Fix)
- Security vulnerabilities
- Critical bugs
- Data loss issues
- Breaking changes without deprecation

## ⚠️ IMPORTANT (Should Fix)
- Missing error handling
- Poor performance
- Missing tests
- Design concerns

## 💭 SUGGESTIONS (Nice to Have)
- Code style improvements
- Better variable names
- Additional comments
- Minor refactoring

## ✅ PRAISE (Acknowledge Good Work)
- Good design decisions
- Well-written tests
- Clear documentation
- Elegant solutions

Always include positive feedback!
EOF

    cat feedback-categories.md
    ```

    Be clear about what's essential vs optional.
    </details>

9. **Create Review Checklist**

    Use a consistent checklist for thorough reviews.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat > review-checklist.md << 'EOF'
# Code Review Checklist

## Before Starting
- [ ] Read PR description and linked issues
- [ ] Understand the problem being solved
- [ ] Check PR size (<400 lines preferred)

## Code Quality
- [ ] Code is readable and maintainable
- [ ] Functions are focused and single-purpose
- [ ] Variable names are descriptive
- [ ] No commented-out code

## Functionality
- [ ] Code does what PR claims
- [ ] Edge cases are handled
- [ ] Error handling is appropriate
- [ ] Input validation is present

## Security
- [ ] Input is validated and sanitised
- [ ] No secrets in code
- [ ] Authentication/authorisation correct
- [ ] No SQL injection, XSS, etc.

## Testing
- [ ] Tests are included
- [ ] Tests cover edge cases
- [ ] All tests passing

## Documentation
- [ ] Public APIs documented
- [ ] Complex logic has comments
- [ ] README updated if needed
EOF

    cat review-checklist.md
    ```

    Checklists ensure consistent, thorough reviews.
    </details>

## Challenge Exercises

### Challenge 1: Complete Review Cycle

Practise the full review workflow:
1. Create PR with intentional issues (security, bugs)
2. Perform self-review and fix obvious problems
3. Simulate review feedback
4. Address all feedback
5. Get approval and merge

<details>
<summary>Hint</summary>

Include a SQL injection vulnerability, missing error handling, no tests, and unclear variable names. Then address each category systematically.
</details>

### Challenge 2: Constructive Feedback

Create three PRs with different issues:
1. Critical security issues (Request Changes)
2. Design concerns (Comment)
3. Minor style issues (Approve with suggestions)

Write appropriate feedback for each.

<details>
<summary>Hint</summary>

Use appropriate review states. For security issues, clearly explain the vulnerability and suggest fixes. For style issues, note they're non-blocking.
</details>

## Reflection Questions

After completing this lab, consider:

1. **Review States:** When should you use Request Changes versus just Comment?
2. **Feedback Quality:** What makes feedback constructive versus destructive?
3. **Balance:** How do you balance thoroughness with review speed?
4. **Disagreements:** How should you handle disagreements in code review?

## Clean Up

If you want to remove the lab directory:

```bash
cd ..
rm -rf code-review-lab
```

## Key Practices Summary

**As Reviewer:**
- Be specific and actionable
- Explain the "why" behind feedback
- Acknowledge good work
- Focus on the code, not the person
- Categorise feedback by severity

**As Author:**
- Self-review before requesting review
- Respond to all feedback
- Ask questions if unclear
- Don't take feedback personally
- Re-request review after fixes

## What You've Learnt

- Setting up code for effective review
- Performing thorough self-review
- Adding line-specific comments
- Using GitHub's suggestion feature
- Understanding review states (Comment, Approve, Request Changes)
- Addressing review feedback professionally
- Distinguishing blocking vs non-blocking feedback
- Creating and using review checklists
- Giving constructive, actionable feedback
- Best practices for both reviewer and author roles

Effective code review is one of the most valuable practices for maintaining code quality and sharing knowledge!
