# Lab Appendix A: Group Collaboration Exercise

## Objective

Work as a team to collaboratively build an IT Operations Runbook using GitHub's fork and pull request workflow. This exercise simulates real-world collaboration patterns used by operations teams.

**Duration:** 60-75 minutes
**Team Size:** 3-4 people per group

## Prerequisites

- Completed Lab 1 (Git Fundamentals Review)
- GitHub account created
- Basic understanding of branches and commits
- Git configured with your name and email

## About This Lab

This is a collaborative exercise where your team will build an IT Operations Runbook together. Each team member will:
1. Fork a central repository
2. Create a branch for their contribution
3. Add documentation based on their expertise
4. Submit a pull request
5. Review teammates' pull requests
6. Resolve any conflicts that arise
7. Merge approved changes

**Why this matters:** This mirrors how operations teams use GitHub to collaboratively maintain documentation, runbooks, configuration templates, and procedures.

---

## Part 1: Team Setup (10 minutes)

### Step 1: Designate Roles

Assign these roles within your team:

1. **Repository Owner** - Creates the central repository (1 person)
2. **Contributors** - Fork and contribute to the repository (everyone including the owner)
3. **Primary Reviewer** - Leads the review process (1 person, can also contribute)

**Note:** Even the repository owner will fork their own repository to practice the workflow.

---

### Step 2: Repository Owner - Create Central Repository

**Repository Owner only performs these steps:**

1. Go to GitHub and create a new repository:
   - Name: `team-ops-runbook`
   - Description: "IT Operations Runbook - Team [Your Team Name]"
   - Public repository
   - ✅ Initialize with a README
   - ✅ Add .gitignore: None
   - License: None

2. Create the initial structure. Clone the repository:
   ```bash
   git clone https://github.com/YOUR-USERNAME/team-ops-runbook.git
   cd team-ops-runbook
   ```

3. Create the directory structure:
   ```bash
   mkdir troubleshooting procedures checklists
   touch troubleshooting/.gitkeep procedures/.gitkeep checklists/.gitkeep
   ```

4. Create a template README:
   ```bash
   cat > README.md << 'EOF'
   # IT Operations Runbook

   A collaborative runbook for IT operations procedures, troubleshooting guides, and checklists.

   ## Team Members
   - [Add your names here]

   ## Structure

   - `troubleshooting/` - Troubleshooting guides for common issues
   - `procedures/` - Step-by-step procedures for standard operations
   - `checklists/` - Checklists for routine tasks

   ## Contributing

   1. Fork this repository
   2. Create a branch for your contribution
   3. Add your documentation
   4. Submit a pull request
   5. Wait for review and approval

   ## Documentation Guidelines

   - Use clear, concise language
   - Include step-by-step instructions
   - Add examples where helpful
   - Use Markdown formatting
   EOF
   ```

5. Commit and push:
   ```bash
   git add .
   git commit -m "Initial repository structure"
   git push origin main
   ```

6. **Share the repository URL with your team members.**

---

## Part 2: Everyone - Fork and Clone (5 minutes)

**All team members (including the repository owner) perform these steps:**

### Step 3: Fork the Repository

1. Navigate to the central repository URL (provided by the repository owner)
2. Click the **Fork** button in the top-right corner
3. Select your personal account as the destination
4. Wait for the fork to complete

You now have your own copy of the repository!

### Step 4: Clone Your Fork

```bash
# Clone YOUR fork (not the central repository)
git clone https://github.com/YOUR-USERNAME/team-ops-runbook.git
cd team-ops-runbook

# Add the central repository as 'upstream'
git remote add upstream https://github.com/OWNER-USERNAME/team-ops-runbook.git

# Verify remotes
git remote -v
# You should see:
# origin    https://github.com/YOUR-USERNAME/team-ops-runbook.git (your fork)
# upstream  https://github.com/OWNER-USERNAME/team-ops-runbook.git (central repo)
```

---

## Part 3: Individual Contributions (20 minutes)

**Each team member will add documentation based on their role/expertise.**

### Step 5: Choose Your Contribution

Each team member should choose ONE of these topics (coordinate to avoid duplication):

**Troubleshooting Guides:**
- Network connectivity issues
- Database connection problems
- Server not responding
- Application errors
- Performance degradation

**Procedures:**
- User account creation
- Password reset process
- Software installation workflow
- Backup verification
- Security patch deployment

**Checklists:**
- Daily system health checks
- Weekly maintenance tasks
- Pre-deployment checklist
- Incident response checklist
- New employee onboarding

### Step 6: Create a Branch

```bash
# Create a descriptive branch name
git checkout -b add-network-troubleshooting  # Example branch name

# Or use your chosen topic:
# git checkout -b add-password-reset-procedure
# git checkout -b add-backup-checklist
```

### Step 7: Add Your Documentation

Create a new Markdown file in the appropriate directory:

```bash
# Example: Creating a troubleshooting guide
cat > troubleshooting/network-connectivity.md << 'EOF'
# Network Connectivity Troubleshooting Guide

## Symptoms
- Cannot access network resources
- "Network unreachable" errors
- Intermittent connectivity

## Quick Checks

1. **Verify physical connection**

   ```bash
   # Check network interface status
   ip link show
   # or on Windows: ipconfig /all
   ```

2. **Test local connectivity**

   ```bash
   ping 127.0.0.1  # Test loopback
   ping <gateway-ip>  # Test gateway
   ```

3. **Check DNS resolution**

   ```bash
   nslookup google.com
   # or: dig google.com
   ```

4. **Verify network configuration**
   - Check IP address assignment (DHCP vs static)
   - Confirm subnet mask
   - Verify default gateway

## Common Solutions

### Issue: No IP address assigned
**Solution:**
- Restart network service
- Check DHCP server status
- Verify network cable connection

### Issue: Can ping by IP but not by hostname
**Solution:**
- Check DNS server configuration
- Verify /etc/resolv.conf (Linux) or DNS settings (Windows)
- Flush DNS cache

### Issue: Firewall blocking connectivity
**Solution:**
- Check firewall rules
- Temporarily disable to test (re-enable after testing)
- Add necessary exceptions

## When to Escalate

- Multiple users affected
- Hardware failure suspected
- Problem persists after all checks
- Network infrastructure changes needed

## Related Documents

- [Network Configuration Procedure](../procedures/)
- [Daily Network Health Checklist](../checklists/)
EOF
```

**Customize your document:**
- Use your own expertise and experience
- Make it realistic and useful
- Aim for 20-30 lines minimum
- Use proper Markdown formatting

### Step 8: Update Team Members in README

Edit the README.md to add your name:

```bash
# Edit README.md
nano README.md
# or: code README.md
# or: vim README.md
```

Under "Team Members", add your name:
```markdown
## Team Members
- Alice Smith (Network Administration)
- Bob Jones (Database Administration)
```

### Step 9: Commit Your Changes

```bash
# Stage your new file and README
git add .

# Create a descriptive commit message
git commit -m "Add network connectivity troubleshooting guide

- Created comprehensive guide for diagnosing network issues
- Included common symptoms and solutions
- Added escalation criteria"
```

### Step 10: Push to Your Fork

```bash
# Push your branch to YOUR fork
git push origin add-network-troubleshooting
# Replace with your actual branch name
```

---

## Part 4: Create Pull Requests (10 minutes)

### Step 11: Open a Pull Request

1. Go to **your fork** on GitHub
2. You'll see a banner: "Compare & pull request" - click it
3. **Alternatively:**
   - Click "Pull requests" tab
   - Click "New pull request"
   - Ensure base repository is the **central repo** (upstream)
   - Ensure base branch is `main`
   - Ensure compare is your branch

4. Fill in the pull request:
   - **Title:** "Add network connectivity troubleshooting guide" (or your topic)
   - **Description:**
     ```markdown
     ## Summary
     Added a comprehensive troubleshooting guide for network connectivity issues.

     ## Contents
     - Common symptoms
     - Step-by-step diagnostic procedures
     - Solutions for typical problems
     - Escalation criteria

     ## Checklist
     - [x] Documentation is clear and complete
     - [x] Proper Markdown formatting used
     - [x] Added my name to team members list
     ```

5. Click "Create pull request"

---

## Part 5: Code Review (15 minutes)

**Everyone reviews everyone else's pull requests!**

### Step 12: Review Teammates' Pull Requests

1. Go to the **central repository** (upstream, owned by the repository owner)
2. Click the "Pull requests" tab
3. Open a teammate's pull request
4. Review the changes:
   - Click "Files changed" tab
   - Read through the documentation
   - Check for:
     - Clear writing
     - Complete information
     - Proper Markdown formatting
     - Accuracy of technical content

5. Leave feedback:
   - **For good content:** Click "Review changes" → "Approve"
   - **For improvements needed:**
     - Click on a line number to comment
     - Suggest specific improvements
     - Click "Review changes" → "Request changes"
   - **For questions:** Just add comments without approving/rejecting

**Example review comments:**
- "Great guide! Very thorough. Approved."
- "Could you add a section about wireless connectivity issues?"
- "Minor: Line 23 has a typo - 'teh' should be 'the'"
- "This is really helpful. I'd suggest adding the Windows equivalent commands."

### Step 13: Respond to Feedback

If you receive change requests:

1. Go to your **local repository**
2. Make the requested changes
3. Commit the changes:
   ```bash
   git add .
   git commit -m "Address review feedback

   - Fixed typo on line 23
   - Added Windows command equivalents
   - Expanded wireless troubleshooting section"
   ```
4. Push to your fork:
   ```bash
   git push origin your-branch-name
   ```
5. The pull request automatically updates!
6. Reply to the reviewer thanking them and noting changes made

---

## Part 6: Merge Conflicts (Intentional!) (10 minutes)

**Now let's create and resolve a merge conflict - a crucial skill!**

### Step 14: Repository Owner - Merge First PR

**Repository Owner:**
1. Choose one pull request to merge first
2. Review it, approve it, and click "Merge pull request"
3. Confirm the merge
4. Delete the branch (optional but recommended)

### Step 15: Contributors - Update Your Fork

**Everyone else:**

Your pull request now likely has conflicts (because the README was updated). Let's fix it:

```bash
# Fetch updates from central repository
git fetch upstream

# Switch to your main branch
git checkout main

# Merge upstream changes into your main
git merge upstream/main

# Switch back to your feature branch
git checkout your-branch-name

# Merge updated main into your branch
git merge main
```

**If you see conflicts:**

```bash
# Git will show: CONFLICT (content): Merge conflict in README.md

# Edit README.md
nano README.md

# You'll see conflict markers:
<<<<<<< HEAD
## Team Members
- Alice Smith (Network Administration)
=======
## Team Members
- Bob Jones (Database Administration)
>>>>>>> main

# Resolve by keeping both names:
## Team Members
- Alice Smith (Network Administration)
- Bob Jones (Database Administration)

# Save and exit, then:
git add README.md
git commit -m "Merge main and resolve conflicts"
git push origin your-branch-name
```

Your pull request is now updated and conflict-free!

### Step 16: Repository Owner - Merge Remaining PRs

**Repository Owner:**
- Review and merge the remaining pull requests
- Each contributor should see their PR merged!

---

## Part 7: Sync and Verify (5 minutes)

### Step 17: Everyone - Sync Your Fork

```bash
# Ensure you're on main branch
git checkout main

# Fetch and merge all changes from central repository
git fetch upstream
git merge upstream/main

# Push to your fork to keep it in sync
git push origin main

# View the beautiful result
git log --oneline --graph --all -10
```

### Step 18: Verify the Final Result

1. Visit the **central repository** on GitHub
2. Browse the `troubleshooting/`, `procedures/`, and `checklists/` directories
3. Admire your team's collaborative work!
4. Check that all team members are listed in README.md

---

## Reflection Questions

Discuss as a team:

1. **What was the most challenging part of this exercise?**
   - Forking and cloning?
   - Creating pull requests?
   - Resolving conflicts?

2. **How did the fork and PR model differ from working directly on the main branch?**
   - What are the advantages?
   - When might this workflow be overkill?

3. **What happened when multiple people edited README.md?**
   - How did you resolve the conflicts?
   - How could you minimize conflicts in the future?

4. **How could your team use this workflow in real work?**
   - Documentation projects?
   - Configuration management?
   - Runbook maintenance?
   - Knowledge sharing?

5. **What would you do differently next time?**

---

## Challenge Exercises (If Time Permits)

### Challenge 1: Add a Diagram

Add a flowchart or diagram to one of the documents:
- Create a simple diagram using a tool (draw.io, Mermaid, or even ASCII art)
- Add it to your document
- Submit a new pull request

### Challenge 2: Establish Contributing Guidelines

Create a `CONTRIBUTING.md` file that includes:
- Naming conventions for files
- Required sections for each document type
- Review process guidelines
- How to report issues

### Challenge 3: Add GitHub Issue Templates

Create issue templates for:
- New documentation request
- Documentation update needed
- Broken link or error report

### Challenge 4: Protected Branches

**Repository Owner:**
- Enable branch protection on `main`
- Require pull request reviews
- Try to push directly to main (it should be blocked!)

---

## What You've Learned

Through this exercise, you've practiced:

- **Forking repositories** - Creating your own copy of a project
- **Remote management** - Working with origin and upstream
- **Branch workflows** - Creating feature branches for contributions
- **Pull requests** - Proposing changes to a central repository
- **Code review** - Reviewing and providing feedback on contributions
- **Merge conflicts** - Identifying and resolving conflicts
- **Collaboration** - Working as a team on a shared project
- **Fork synchronization** - Keeping your fork up to date

**Real-world application:** This is exactly how teams collaborate on GitHub for documentation, infrastructure as code, configuration management, and internal tools.

---

## Clean-up (Optional)

If you want to clean up after the exercise:

```bash
# Delete your local repository
cd ..
rm -rf team-ops-runbook

# On GitHub, you can:
# 1. Delete your fork (Settings → Delete this repository)
# 2. Repository owner can delete the central repository
```

---

## Troubleshooting

### "Permission denied" when pushing

**Problem:** You're trying to push to the central repository directly.

**Solution:** Make sure you're pushing to `origin` (your fork), not `upstream`:
```bash
git push origin your-branch-name
```

### Pull request shows unexpected commits

**Problem:** You didn't create a branch; you worked directly on `main`.

**Solution:**
```bash
# Create a branch from your current state
git checkout -b proper-branch-name
git push origin proper-branch-name
# Then create PR from this branch
```

### Cannot create pull request

**Problem:** Your fork is not up to date with central repository.

**Solution:**
```bash
git fetch upstream
git checkout main
git merge upstream/main
git checkout your-branch
git merge main
git push origin your-branch
```

### Confused about origin vs upstream

**Remember:**
- `origin` = Your fork (where you push)
- `upstream` = Central repository (where you pull from)

---

## Additional Resources

- [GitHub Docs: Fork a repo](https://docs.github.com/en/get-started/quickstart/fork-a-repo)
- [GitHub Docs: About pull requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)
- [GitHub Docs: Resolving merge conflicts](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts)

---

**Congratulations!** You've completed a realistic collaborative GitHub workflow. This is exactly how operations teams maintain runbooks, documentation, and infrastructure code in the real world.
