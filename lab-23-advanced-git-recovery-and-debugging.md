# Lab 23: Advanced Git Recovery and Debugging

__Objective__: To master Git recovery techniques and debugging tools for handling complex repository issues

## Prerequisites

- Completion of Labs 1-22 or equivalent advanced Git knowledge
- Strong understanding of Git internals and workflow
- A terminal or command prompt
- Confidence working with potentially destructive Git operations

## Important Note

This is the **final lab** of the course. Some exercises intentionally create "broken" states to practise recovery. Always work in isolated test repositories. The reflog and Git's content-addressable storage make most operations recoverable, but practise in safe environments first.

## Step-by-Step Exercise

### Part 1: Understanding Recovery Tools

1. **Set Up Test Repository and Master Reflog**

    Create a repository for recovery practice and learn to navigate the reflog. Use `git reflog` to track all HEAD movements.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir git-recovery-lab
    cd git-recovery-lab
    git init

    # Create initial commits
    echo "# Recovery Lab" > README.md
    git add README.md
    git commit -m "Initial commit"

    echo "function main() { return 0; }" > app.js
    git add app.js
    git commit -m "Add main function"

    echo "function helper() { return 'help'; }" > utils.js
    git add utils.js
    git commit -m "Add utilities"

    # View reflog
    git reflog

    # Reflog with dates
    git reflog --date=iso

    # Detailed reflog
    git log -g --pretty=format:"%h %gD: %gs" -10
    ```

    The reflog shows every HEAD movement for 90 days - your safety net!

    **Understanding reflog entries:**
    - `HEAD@{0}` = Current position
    - `HEAD@{1}` = Previous position
    - `HEAD@{n}` = n positions ago

    Check reflog expiry: `git config --get gc.reflogExpire`
    </details>

2. **Recover from Accidental Hard Reset**

    Practise the most common recovery scenario - recovering commits after `git reset --hard`. Use reflog to find and restore lost commits.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Save current state
    original_commit=$(git rev-parse HEAD)
    echo "Original: $original_commit"

    git log --oneline -3

    # Destructive reset (lose 2 commits)
    git reset --hard HEAD~2

    echo "After reset:"
    git log --oneline -3
    echo "Lost 2 commits!"

    # Recovery using reflog
    git reflog -5

    # Option 1: Reset back
    git reset --hard HEAD@{1}

    # Option 2: Create branch at lost commit
    # git branch recovered HEAD@{1}

    # Verify recovery
    recovered=$(git rev-parse HEAD)
    if [ "$original_commit" = "$recovered" ]; then
        echo "✓ Successfully recovered!"
    fi

    git log --oneline -3
    ```

    **Key lesson:** Hard reset doesn't delete commits - they're in reflog for 90 days!
    </details>

3. **Recover Deleted Branch**

    Restore a branch that was deleted before merging. Use reflog to find the branch tip and recreate it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create feature branch
    git checkout -b feature/important
    echo "Critical feature" > feature.js
    git add feature.js
    git commit -m "Implement critical feature"

    branch_tip=$(git rev-parse HEAD)
    echo "Branch tip: $branch_tip"

    # Switch and delete
    git checkout main
    git branch -D feature/important
    echo "Branch deleted!"

    git branch --list "feature/important"
    echo "Not in branch list"

    # Recovery: Find in reflog
    git reflog | grep "important"

    # Find commit where we were on that branch
    deleted_commit=$(git reflog | grep "checkout.*important" | head -1 | awk '{print $1}')
    echo "Found at: $deleted_commit"

    # Recreate branch
    git branch feature/important $deleted_commit

    # Verify
    git log feature/important --oneline -2
    echo "✓ Branch recovered!"
    ```

    Branches are just pointers - recreating them is simple with the commit hash!
    </details>

### Part 2: Advanced Debugging

4. **Use Git Bisect to Find Bugs**

    Find the exact commit that introduced a bug using binary search. Let Git automatically narrow down thousands of commits.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create history with bug in commit 6
    for i in {1..10}; do
        if [ $i -eq 6 ]; then
            # Introduce bug
            echo "function add(a, b) { return a - b; }" > calculator.js
        else
            # Correct code
            echo "function add(a, b) { return a + b; }" > calculator.js
        fi

        git add calculator.js
        git commit -m "Update calculator v$i"
    done

    # Create test script
    cat > test.js << 'EOF'
    const fs = require('fs');
    const code = fs.readFileSync('calculator.js', 'utf8');
    eval(code);

    if (add(2, 3) === 5) {
        console.log("Test passed");
        process.exit(0);
    } else {
        console.log("Test failed");
        process.exit(1);
    }
    EOF

    # Start bisect
    git bisect start

    # Mark current as bad
    git bisect bad

    # Mark old commit as good
    git bisect good HEAD~9

    # Automated bisect
    git bisect run node test.js

    # Git finds the bad commit!
    echo "Bisect found the bug!"

    # Reset
    git bisect reset
    ```

    **Bisect is powerful:** Finds bugs in O(log n) time - 1000 commits take only ~10 tests!
    </details>

5. **Recover Lost Stash**

    Find and recover stashed changes that were dropped. Use `git fsck` to find dangling stash commits.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create work in progress
    echo "Work in progress" > wip.js
    git add wip.js
    echo "More changes" >> README.md

    # Stash it
    git stash push -m "Important WIP"
    git stash list
    echo "Stash created"

    # Accidentally drop
    git stash drop
    echo "Stash dropped!"

    git stash list
    echo "Stash list empty"

    # Recovery: Stashes are commits!
    git fsck --unreachable | grep commit

    # Find stash commit
    stash_commit=$(git fsck --unreachable | grep commit | head -1 | awk '{print $3}')
    echo "Found stash: $stash_commit"

    # View contents
    git show $stash_commit

    # Recover
    git stash apply $stash_commit

    # Or create branch
    # git branch recovered-stash $stash_commit

    git status
    echo "✓ Stash recovered!"

    # Clean up
    git checkout .
    rm -f wip.js
    ```

    Stashes are just commits in disguise - findable with `fsck`!
    </details>

### Part 3: Repository Health and History

6. **Verify Repository Integrity**

    Use Git's built-in tools to diagnose and fix repository issues. Run `git fsck` to check for corruption.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Check repository integrity
    echo "Checking integrity..."
    git fsck --full

    # Check for unreachable objects
    git fsck --unreachable

    # Check for dangling objects
    git fsck --dangling

    # Garbage collection
    echo "Running garbage collection..."
    git gc

    # Verify after GC
    git fsck --full

    echo "✓ Repository healthy!"
    ```

    **If corruption occurs:**
    ```bash
    # 1. Backup
    cp -r .git .git.backup

    # 2. Try repair
    git fsck --full
    git gc --aggressive

    # 3. Recover from remote
    git fetch --all
    git reset --hard origin/main

    # 4. Last resort: re-clone
    ```

    Regular `fsck` catches problems early!
    </details>

7. **Advanced Log Searching and Blame**

    Master techniques for finding specific changes and understanding code history. Use pickaxe search and blame.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create code with history
    cat > auth.js << 'EOF'
    function login(username, password) {
        return authenticate(username, password);
    }

    function authenticate(user, pass) {
        return user === "admin";
    }
    EOF

    git add auth.js
    git commit -m "Add authentication"

    # Add logout
    echo "function logout() { clearSession(); }" >> auth.js
    git add auth.js
    git commit -m "Add logout function"

    # Search commit messages
    git log --grep="authentication" --oneline

    # Search by author
    git log --author="$(git config user.name)" --oneline -5

    # Search code changes (pickaxe)
    git log -S"authenticate" --oneline
    git log -S"function login" --oneline

    # Regex search
    git log -G"function.*login" --oneline

    # Blame: Who changed each line
    git blame auth.js

    # Blame specific lines
    git blame -L 1,5 auth.js

    # Track line changes over time
    git log -L :login:auth.js

    # Find when file was created
    git log --diff-filter=A -- auth.js

    # Find deleted code
    git log -S"deleted code" --diff-filter=D

    # Commits between dates
    git log --since="1 week ago" --oneline
    ```

    **Search toolkit:**
    - `-S"text"`: Find additions/deletions of text
    - `-G"regex"`: Search with regex patterns
    - `--grep`: Search commit messages
    - `blame`: Find who changed lines
    - `-L`: Track specific lines or functions
    </details>

### Part 4: Complex Recovery Scenarios

8. **Understand and Recover from Detached HEAD**

    Master detached HEAD state - what it is, when it happens, and how to recover work. Create commits in detached HEAD and recover them.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Current state
    git log --oneline -3

    # Enter detached HEAD
    commit_hash=$(git rev-parse HEAD~2)
    echo "Checking out: $commit_hash"
    git checkout $commit_hash

    git status
    echo "In detached HEAD state!"

    # Make commits while detached
    echo "Detached work" > detached.txt
    git add detached.txt
    git commit -m "Work while detached"

    echo "More detached work" >> detached.txt
    git add detached.txt
    git commit -m "More detached work"

    detached_commit=$(git rev-parse HEAD)
    echo "Made commits: $detached_commit"

    # Switch back (Git warns!)
    git checkout main

    echo "Commits appear lost"
    git log --oneline -5

    # Recovery Method 1: Create branch
    git branch recovered-detached $detached_commit
    git log --oneline --all --graph -8

    # Recovery Method 2: Using reflog
    git reflog -10
    detached_reflog=$(git reflog | grep "detached" | head -1 | awk '{print $1}')
    git branch another-recovery $detached_reflog

    # Merge if needed
    git merge recovered-detached --no-edit

    echo "✓ Recovered from detached HEAD!"
    ```

    **Detached HEAD happens when:**
    - `git checkout <commit-hash>`
    - `git checkout <tag>`
    - During rebase operations

    **To work safely:** Create a branch immediately with `git checkout -b temp-branch`

    Detached commits stay in reflog for 90 days!
    </details>

9. **Complete Recovery Scenario**

    Handle a complex situation with multiple issues: deleted branch, lost commits, dropped stash, and wrong branch commits. Use all recovery techniques together.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    echo "=== Creating Complex Scenario ==="

    # 1. Feature branch with work
    git checkout -b feature/complex
    echo "Feature" > feature.txt
    git add feature.txt
    git commit -m "Feature work"

    # 2. Stash some work
    echo "Stashed" > stashed.txt
    git add stashed.txt
    git stash push -m "Important stash"

    # 3. More commits
    echo "More work" >> feature.txt
    git add feature.txt
    git commit -m "More feature work"

    # 4. Back to main
    git checkout main
    echo "Main work" > main.txt
    git add main.txt
    git commit -m "Work on main"

    # Now create problems:

    # Problem 1: Delete feature branch
    git branch -D feature/complex
    echo "Problem 1: Branch deleted!"

    # Problem 2: Hard reset (lose commit)
    git reset --hard HEAD~1
    echo "Problem 2: Lost commit!"

    # Problem 3: Drop stash
    git stash drop 2>/dev/null || true
    echo "Problem 3: Stash dropped!"

    # Problem 4: Commit to wrong branch
    git checkout -b wrong-branch
    echo "Wrong" > wrong.txt
    git add wrong.txt
    git commit -m "Wrong branch commit"
    git checkout main

    echo -e "\n=== Recovery Process ==="

    # Step 1: Analyse reflog
    echo "Analysing reflog..."
    git reflog -20 | head -10

    # Step 2: Recover deleted branch
    deleted_branch=$(git reflog | grep "checkout.*complex" | head -1 | awk '{print $1}')
    git branch feature/complex $deleted_branch
    echo "✓ Branch recovered"

    # Step 3: Recover lost commit
    lost_commit=$(git reflog | grep "Work on main" | head -1 | awk '{print $1}')
    git cherry-pick $lost_commit
    echo "✓ Lost commit recovered"

    # Step 4: Recover stash
    stash_commit=$(git fsck --unreachable | grep commit | head -1 | awk '{print $3}')
    if [ -n "$stash_commit" ]; then
        git branch recovered-stash $stash_commit
        echo "✓ Stash recovered"
    fi

    # Step 5: Move commit from wrong branch
    git checkout wrong-branch
    wrong_commit=$(git rev-parse HEAD)
    git checkout main
    git cherry-pick $wrong_commit
    git branch -D wrong-branch
    echo "✓ Commit moved"

    # Verify
    echo -e "\n=== Verification ==="
    echo "Main branch:"
    git log --oneline -5

    echo -e "\nFeature branch:"
    git log --oneline feature/complex -3

    echo -e "\nAll branches:"
    git branch -a

    echo -e "\n✓ Complete recovery successful!"
    ```

    **Recovery techniques used:**
    - Reflog analysis
    - Branch recreation from hash
    - Cherry-pick for moving commits
    - fsck for finding unreachable objects
    - Strategic branch creation

    **Key lesson:** Nearly everything is recoverable if you understand reflog and Git's object model!
    </details>

## Challenge Exercises

### Challenge 1: Ultimate Recovery Test

Create the most challenging recovery scenario:
- Delete multiple branches with unmerged work
- Perform several `reset --hard` operations
- Create commits in detached HEAD
- Drop multiple stashes
- Abandon a rebase mid-operation

Then recover everything using only reflog and fsck. Document your recovery process.

<details>
<summary>Hint</summary>

Start with comprehensive reflog analysis (`git reflog --all`). Create a timeline of all operations. Use `git fsck --unreachable` to find orphaned objects. Create branches for all important commits before attempting any cleanup. Verify recovery by checking file contents, not just commit counts.
</details>

### Challenge 2: Forensic Analysis

Investigate a repository to answer:
- When was specific code added?
- Who modified particular lines and why?
- When did a specific bug appear?
- What commits changed a given file?
- Which branches contain a specific commit?

<details>
<summary>Hint</summary>

**Forensic toolkit:**
```bash
# When code was added
git log -S"specific code" --source --all

# Who modified lines
git blame -L start,end file
git log -L start,end:file

# Find bug introduction
git bisect start HEAD v1.0.0
git bisect run ./test-script.sh

# File history
git log --follow --oneline -- file

# Branches containing commit
git branch --contains <commit>
git branch -r --contains <commit>
```
</details>

## Reflection Questions

After completing this lab and the **entire course**, consider:

1. What makes Git's recovery capabilities so powerful?
2. How does understanding Git's object model help with recovery?
3. When should you use `reset` versus `revert`?
4. What's your strategy for preventing data loss?
5. How do you balance Git safety with productivity?
6. What recovery techniques are most valuable in daily work?
7. How has this course changed your Git workflow?
8. What Git topics do you want to explore further?

## Course Completion

**Congratulations!** You've completed all 23 labs of the Intermediate Git and GitHub course.

### Your Journey

You've progressed from:
- **Labs 1-3:** Git fundamentals and internals
- **Labs 4-11:** Merging strategies and branch management
- **Labs 12-14:** Context switching and submodules
- **Labs 15-19:** GitHub collaboration and hooks
- **Labs 20-22:** CI/CD with GitHub Actions
- **Lab 23:** Advanced recovery and debugging

### Skills Mastered

You can now:
- ✓ Understand Git's object model and internals
- ✓ Use various merge strategies appropriately
- ✓ Resolve complex merge conflicts
- ✓ Rebase interactively and maintain clean history
- ✓ Implement branching strategies for teams
- ✓ Collaborate effectively via GitHub
- ✓ Automate workflows with GitHub Actions
- ✓ Manage multi-environment deployments
- ✓ Recover from any Git disaster
- ✓ Debug complex repository issues
- ✓ Mentor others in Git best practices

### What's Next?

1. **Practise regularly:** Apply these skills in real projects
2. **Explore advanced topics:**
   - Git internals and plumbing commands
   - Custom Git commands and aliases
   - Git server administration
   - Advanced GitHub Actions patterns
3. **Share knowledge:** Help colleagues learn Git
4. **Contribute:** Use your skills in open source
5. **Stay current:** Follow Git and GitHub updates
6. **Optimise workflows:** Adapt strategies to your team

### Remember

**Git is forgiving.** With reflog and proper knowledge, mistakes are learning opportunities, not disasters. The safety net lasts 90 days, and with your new skills, you can confidently work knowing recovery is always possible.

**You're now equipped to handle any Git scenario with confidence!**

## What You've Learned

### Throughout the Entire Course

**Git Fundamentals:**
- Object model (commits, trees, blobs, references)
- Configuration and customisation
- Core Git concepts

**Merging and Branching:**
- Fast-forward, recursive, and octopus merges
- Complex conflict resolution
- Cherry-picking selective changes
- Rebasing for linear history
- Interactive rebase for history editing
- Git Flow, GitHub Flow, trunk-based development
- Choosing strategies for your team

**Context Switching:**
- Git stash for temporary work
- Worktrees for parallel development
- Submodules and subtrees

**GitHub Collaboration:**
- Fork and pull request workflows
- Code review processes
- Branch protection rules
- Security and credential management
- Git hooks for automation

**CI/CD with GitHub Actions:**
- Workflow creation and syntax
- Automated testing and building
- Deployment automation
- Multi-environment management
- Environment protection and secrets

**Recovery and Debugging:**
- Using reflog for recovery
- Recovering deleted branches and commits
- Finding bugs with bisect
- Advanced log searching and blame
- Repository verification and repair
- Handling detached HEAD
- Debugging Git operations
- Complex recovery scenarios

**Professional Skills:**
- Knowing when to use each Git feature
- Balancing safety with productivity
- Team collaboration best practices
- Incident recovery procedures
- Git workflow optimisation
- Capability to mentor others

### Next Steps in Your Git Journey

Continue growing:
- Explore Git internals more deeply
- Build custom Git tools
- Contribute to open source projects
- Share your knowledge with others
- Stay updated with Git developments

## Clean Up

```bash
cd ..
rm -rf git-recovery-lab
```

---

## Final Words

You've completed an intensive journey through intermediate and advanced Git concepts. You started with fundamentals and now possess the skills to handle any version control scenario with confidence.

The reflog and Git's content-addressable storage make most operations recoverable. With this knowledge, you can work boldly, knowing that mistakes are temporary and recovery is within reach.

**Remember:** Git rewards understanding. The more you know about how it works, the more powerful and safe your workflows become.

**Congratulations on completing the Intermediate Git and GitHub course!**

You're now ready to tackle any version control challenge and mentor others on their Git journey.
