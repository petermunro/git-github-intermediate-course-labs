# Lab 2: Git's Object Model and Internal Structure

__Objective__: To explore Git's internal structure and understand how Git stores data using its object model

## Prerequisites

- Completion of Lab 1 or equivalent Git knowledge
- Git installed on your machine
- A terminal or command prompt

## About This Lab

In Module 2, you learned about Git's internal object model and the difference between **porcelain commands** (the user-friendly commands like `git add` and `git commit`) and **plumbing commands** (the low-level commands that expose Git's internals).

In this lab, we'll explore Git's four object types in order of complexity:
1. **Blobs** - file contents
2. **Trees** - directory structures
3. **Commits** - snapshots with metadata
4. **References** - branches and HEAD

The main plumbing commands you'll use are:
- **`git ls-files -s`** - View the staging area
- **`git cat-file -t <hash>`** - View an object's type
- **`git cat-file -p <hash>`** - View an object's contents

## Step-by-Step Exercise

### Part 1: Understanding Blobs (File Storage)

1. **Set Up Your Lab Repository**

    Create a new repository called `git-internals-lab` and navigate into it.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    mkdir git-internals-lab
    cd git-internals-lab
    git init
    ```
    </details>

2. **Create Your First Blob**

    Create a file called `hello.txt` with the content "Hello, Git Internals!", then stage it. Use `git ls-files -s` to see the blob hash that was created.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    echo "Hello, Git Internals!" > hello.txt
    git add hello.txt

    # View the staging area
    git ls-files -s
    ```

    Output: `100644 47a0d4c4b52b43fcaa53f74e63f05bc6edb30f67 0	hello.txt`

    The long hexadecimal string is the SHA-1 hash - the blob's "address" in Git's database.
    </details>

3. **Examine the Blob**

    Use `git cat-file -t <hash>` to check the object type, and `git cat-file -p <hash>` to view its contents. Use the first 7 characters of your hash (e.g., `47a0d4c`).

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Check the type
    git cat-file -t 47a0d4c

    # View the contents
    git cat-file -p 47a0d4c
    ```

    The type is `blob`, and the contents are "Hello, Git Internals!"
    </details>

4. **Test Content Deduplication**

    Create two files with identical content: `file1.txt` and `file2.txt`, both containing "Same content". Stage both files, then use `git ls-files -s` to view them.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    echo "Same content" > file1.txt
    echo "Same content" > file2.txt
    git add file1.txt file2.txt

    # View staging area
    git ls-files -s
    ```

    Both files have the **same blob hash**! Git stores identical content only once.
    </details>

5. **Show That Different Content Gets Different Hashes**

    Create a file called `version1.txt` with "Version 1" and stage it. Note its hash. Then modify it to contain "Version 2", stage it again, and compare the hashes using `git ls-files -s`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Create version 1
    echo "Version 1" > version1.txt
    git add version1.txt
    git ls-files -s version1.txt

    # Modify to version 2
    echo "Version 2" > version1.txt
    git add version1.txt
    git ls-files -s version1.txt
    ```

    The hash is different! Different content = different hash = different blob.

    This is how Git tracks file changes - each version is a separate blob.
    </details>

### Part 2: Understanding Trees (Directory Structure)

6. **Create a Commit to Generate a Tree**

    Commit all your staged files with the message "Initial commit". Then use `git cat-file -p HEAD` to view the commit and find the tree hash.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    git commit -m "Initial commit"

    # View the commit
    git cat-file -p HEAD
    ```

    Look for the line starting with `tree` - that's the hash of the tree object.
    </details>

7. **Examine the Tree Object**

    Copy the tree hash from the previous exercise and use `git cat-file -p <tree-hash>` to view the tree's contents.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # Replace <tree-hash> with your actual tree hash
    git cat-file -p <tree-hash>
    ```

    The tree lists all files with their:
    - File mode (e.g., `100644`)
    - Object type (`blob`)
    - Hash
    - Filename

    **This is how Git represents a directory snapshot!**
    </details>

### Part 3: Understanding Commits (Snapshots with Metadata)

8. **Examine Commit Structure**

    Use `git cat-file -p HEAD` again and identify all the parts of a commit.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    git cat-file -p HEAD
    ```

    A commit contains:
    - `tree` - the root directory snapshot
    - `author` - who created the changes
    - `committer` - who made the commit
    - Commit message

    (Later commits will also show `parent` - the previous commit)
    </details>

9. **Watch a Commit Link to Its Parent**

    Create a new file called `second.txt`, stage it, and commit with message "Second commit". Then view this commit with `git cat-file -p HEAD`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    echo "Second file" > second.txt
    git add second.txt
    git commit -m "Second commit"

    # View the commit
    git cat-file -p HEAD
    ```

    This commit now has a `parent` line pointing to the first commit!

    This is how Git builds the chain of history.
    </details>

### Part 4: Understanding References (HEAD and Branches)

10. **See What HEAD Points To**

    View the contents of the HEAD file using `cat .git/HEAD`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    cat .git/HEAD
    ```

    Output: `ref: refs/heads/main`

    HEAD is a symbolic reference - it points to a branch, not directly to a commit.
    </details>

11. **See How Branches Are Stored**

    View the contents of your branch file at `.git/refs/heads/main`. Compare it to `git rev-parse HEAD`.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    # View the branch file
    cat .git/refs/heads/main

    # Compare with HEAD's commit
    git rev-parse HEAD
    ```

    Both show the same commit hash!

    **A branch is just a file containing a commit hash.** This is why branches are so lightweight in Git.
    </details>

12. **Watch a Branch Move**

    Note the current hash in `.git/refs/heads/main`, create another file, commit it, then check the hash again.

    <details>
    <summary>Reveal Solution</summary>

    ```bash
    echo "Before:"
    cat .git/refs/heads/main

    # Make a new commit
    echo "Third file" > third.txt
    git add third.txt
    git commit -m "Third commit"

    echo "After:"
    cat .git/refs/heads/main
    ```

    The hash changed - the branch moved forward automatically!
    </details>

## Challenge Exercises

### Challenge 1: Explore the Objects Directory

Find where your blob objects are physically stored on disk. Use `find .git/objects -type f` to list them. Notice how the directory name uses the first 2 characters of the hash.

### Challenge 2: Manual Blob Creation

Create a blob manually using `git hash-object -w --stdin`, then verify you can read it back with `git cat-file -p`.

<details>
<summary>Hint</summary>

```bash
echo "Manual blob" | git hash-object -w --stdin
# Returns a hash - save it!

git cat-file -p <hash>
```
</details>

## Reflection Questions

1. Why do identical files result in the same blob hash?
2. How does Git know when a file has changed between versions?
3. What's the relationship between a commit, a tree, and blobs?
4. Why are branches so cheap to create in Git?

## What You've Learned

- **Blobs** store file contents and are identified by content hash
- Identical content produces identical hashes (deduplication)
- Different content produces different hashes (versioning)
- **Trees** represent directory structures by referencing blobs
- **Commits** are snapshots that reference a tree and contain metadata
- Commits form chains via parent references
- **Branches** are simply files containing commit hashes
- **HEAD** is a symbolic reference pointing to the current branch

## Clean Up

```bash
cd ..
rm -rf git-internals-lab
```
