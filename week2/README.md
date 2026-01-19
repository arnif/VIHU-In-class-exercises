# Git Exercises with Git Merge Examples

Here's a beginner-friendly guide with added examples for using Git merge.

---

## **Key Concepts**

Before diving into the exercises, here are the fundamental Git concepts you'll use:

| Command | Purpose |
|---------|---------|
| `git status` | Shows which files are staged, modified, or untracked |
| `git log` | Shows commit history |
| `git diff` | Shows changes between commits, branches, or working directory |
| `git add` | Stages changes for the next commit |
| `git commit` | Saves staged changes as a new commit |
| `git branch` | Lists, creates, or deletes branches |
| `git checkout` / `git switch` | Switches between branches |
| `git merge` | Combines changes from one branch into another |
| `git rebase` | Reapplies commits on top of another branch |

---

## **Setup and Basics**

### **Exercise 1: Git Configuration**

**Why?** Git needs to know who you are so it can associate your commits with your identity. This is essential for collaborative work.

1. Set up your Git username and email. (this step can be skipped if you already have Git setup, go to Step 2 to verify)
    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "your_email@example.com"
    ```
2. Verify your configuration.
    ```bash
    git config --list
    ```

---

### **Exercise 2: Initialize a Repository**

**Why?** The `git init` command creates a hidden `.git` folder that tracks all changes in your project. Without this, Git won't monitor your files.

1. Create a new directory and initialize a repository.
    ```bash
    mkdir git-merge-exercises
    cd git-merge-exercises
    git init
    ```
2. Confirm your default branch name.
    - Some setups use `master` instead of `main`. If `main` is not your default branch, update later steps to use your branch name.
    ```bash
    git branch --show-current
    ```

---

## **Working with Files**

### **Exercise 3: Adding and Committing Files**

**Why?** Git uses a two-step process: first you *stage* changes (`git add`), then you *commit* them. This lets you choose exactly which changes to include in each commit.

1. Create a file and commit it.
    ```bash
    echo "Hello, Git!" > hello.txt
    git add hello.txt
    git commit -m "Add hello.txt with initial content"
    ```
2. Check your repository state.
    ```bash
    git status
    ```
    - **Clean working tree**: No uncommitted changes
    - **Changes not staged**: Modified files that haven't been added
    - **Changes to be committed**: Files ready for the next commit

---

### **Exercise 4: Inspecting Your Work**

**Why?** Before committing or merging, you should always review what's changed. These commands help you understand your current state.

1. View commit history.
    ```bash
    git log --oneline
    ```
    - The `--oneline` flag shows a compact view (one commit per line)
    - Each commit has a unique hash (e.g., `a1b2c3d`)

2. View a more detailed history with branch visualization.
    ```bash
    git log --oneline --graph --decorate --all
    ```

3. Make a change to see `git diff` in action.
    ```bash
    echo "A new line" >> hello.txt
    git diff
    ```
    - Lines starting with `+` are additions
    - Lines starting with `-` are deletions

4. Stage and view the difference between staged and last commit.
    ```bash
    git add hello.txt
    git diff --staged
    ```

5. Commit the change.
    ```bash
    git commit -m "Add a new line to hello.txt"
    ```

---

## **Branching and Merging**

### **Exercise 5: Merging Fast-Forward**

**Why?** A fast-forward merge happens when the target branch hasn't changed since you branched off. Git simply moves the pointer forward - no new merge commit is needed. This keeps history clean and linear.

1. Create and switch to a new branch.
    ```bash
    git checkout -b feature-fast-forward
    ```
    - Alternative command: `git switch -c feature-fast-forward`
2. Add a new file and commit it.
    ```bash
    echo "This is a fast-forward merge example." > fast-forward.txt
    git add fast-forward.txt
    git commit -m "Add fast-forward.txt"
    ```
3. Switch back to the `main` branch.
    ```bash
    git checkout main
    ```
4. Merge the `feature-fast-forward` branch into `main`.
    ```bash
    git merge feature-fast-forward
    ```
    - Notice Git says "Fast-forward" - no merge commit was created
5. Verify the merge by checking the files in the `main` branch.
    ```bash
    ls
    ```
6. Inspect the commit history to see the fast-forward merge.
    ```bash
    git log --oneline --graph --decorate
    ```

---

### **Exercise 6: Merging with Divergence (Conflict Resolution)**

**Why?** Conflicts occur when two branches modify the same lines in a file. Git can't automatically decide which change to keep, so you must resolve it manually. This is a critical skill for collaborative development.

1. Create and switch to a new branch.
    ```bash
    git checkout -b feature-divergent
    ```
    - Alternative command: `git switch -c feature-divergent`
2. Modify `hello.txt` and commit the change.
    ```bash
    echo "This is a change in the feature-divergent branch." >> hello.txt
    git add hello.txt
    git commit -m "Update hello.txt in feature-divergent branch"
    ```
3. Switch back to the `main` branch and make a different modification to `hello.txt`.
    ```bash
    git checkout main
    echo "This is a change in the main branch." >> hello.txt
    git add hello.txt
    git commit -m "Update hello.txt in main branch"
    ```
4. Merge the `feature-divergent` branch into `main`.
    ```bash
    git merge feature-divergent
    ```
5. **Resolve the conflict**:
    - Open `hello.txt`, and you will see conflict markers:
      ```
      <<<<<<< HEAD
      (your changes in main)
      =======
      (changes from feature-divergent)
      >>>>>>> feature-divergent
      ```
    - **What these mean:**
      - `<<<<<<< HEAD`: Start of your current branch's changes
      - `=======`: Separator between the two versions
      - `>>>>>>> feature-divergent`: End of the incoming branch's changes
    - Edit the file to resolve the conflict by keeping the desired content (remove the markers).
    - Confirm the file looks correct.
      ```bash
      cat hello.txt
      ```
    - After resolving the conflict, stage the file.
      ```bash
      git add hello.txt
      ```
    - Complete the merge.
      ```bash
      git commit -m "Resolve merge conflict between main and feature-divergent"
      ```

---

### **Exercise 7: Merging Without Conflicts**

**Why?** When branches modify different files (or different parts of the same file), Git can automatically merge them. This is the ideal scenario in collaborative work.

1. Create and switch to another new branch.
    ```bash
    git checkout -b feature-no-conflict
    ```
    - Alternative command: `git switch -c feature-no-conflict`
2. Create a new file and commit it.
    ```bash
    echo "This file will not conflict." > no-conflict.txt
    git add no-conflict.txt
    git commit -m "Add no-conflict.txt"
    ```
3. Switch back to the `main` branch and merge the `feature-no-conflict` branch.
    ```bash
    git checkout main
    git merge feature-no-conflict
    ```
4. Verify the merge by checking the files in the `main` branch.
    ```bash
    ls
    ```

---

### **Exercise 8: Merge vs Rebase Comparison**

**Why?** Both merge and rebase combine changes, but they do it differently:
- **Merge**: Creates a new "merge commit" that combines both branches. Preserves complete history.
- **Rebase**: Rewrites history by placing your commits on top of the target branch. Creates a linear history.

**When to use which?**
- Use **merge** for shared/public branches (preserves history)
- Use **rebase** for local/feature branches before merging (cleaner history)
- **Never rebase commits that have been pushed to a shared repository**

1. Create and switch to a branch.
    ```bash
    git checkout -b feature-rebase
    ```
    - Alternative command: `git switch -c feature-rebase`
2. Modify `hello.txt` and commit the change.
    ```bash
    echo "This is a rebase example." >> hello.txt
    git add hello.txt
    git commit -m "Update hello.txt in feature-rebase branch"
    ```
3. Switch back to the `main` branch and make another change.
    ```bash
    git checkout main
    echo "This is another change in main." >> hello.txt
    git add hello.txt
    git commit -m "Update hello.txt in main branch"
    ```
4. Rebase the `feature-rebase` branch onto `main`.
    ```bash
    git checkout feature-rebase
    git rebase main
    ```
    - If there's a conflict, resolve it, then run `git add <file>` and `git rebase --continue`
5. Switch back to `main` and merge the rebased branch.
    ```bash
    git checkout main
    git merge feature-rebase
    ```
    - Notice this is now a fast-forward merge because rebase put our commits on top of main
6. Inspect the history to compare rebase vs merge results.
    ```bash
    git log --oneline --graph --decorate
    ```

---

## **Essential Git Skills**

### **Exercise 9: Using .gitignore**

**Why?** Some files should never be committed: build artifacts, dependencies, secrets, IDE settings. The `.gitignore` file tells Git to ignore these files.

1. Create some files that should be ignored.
    ```bash
    echo "secret_api_key=12345" > .env
    mkdir node_modules
    echo "dependency" > node_modules/package.txt
    echo "compiled code" > app.build.js
    ```

2. Check status - Git sees all these files.
    ```bash
    git status
    ```

3. Create a `.gitignore` file.
    ```bash
    cat > .gitignore << 'EOF'
    # Environment variables (secrets)
    .env

    # Dependencies
    node_modules/

    # Build artifacts
    *.build.js

    # IDE settings
    .vscode/
    .idea/

    # OS files
    .DS_Store
    Thumbs.db
    EOF
    ```

4. Check status again - the ignored files no longer appear.
    ```bash
    git status
    ```

5. Commit the `.gitignore` file.
    ```bash
    git add .gitignore
    git commit -m "Add .gitignore file"
    ```

**Common patterns:**
- `*.log` - Ignore all files ending in `.log`
- `build/` - Ignore the entire `build` directory
- `!important.log` - Exception: don't ignore this specific file
- `**/temp` - Ignore `temp` directories anywhere in the project

---

### **Exercise 10: Using Git Stash**

**Why?** Sometimes you need to switch branches but have uncommitted changes. `git stash` temporarily saves your changes so you can switch branches cleanly, then restore them later.

1. Make some changes without committing.
    ```bash
    echo "Work in progress..." >> hello.txt
    echo "Another WIP file" > wip.txt
    git status
    ```

2. Try to switch branches (this might cause issues with conflicts).
    ```bash
    git stash
    ```
    - Your working directory is now clean
    - Changes are saved in the stash

3. Verify the working directory is clean.
    ```bash
    git status
    ```

4. View your stashed changes.
    ```bash
    git stash list
    ```

5. Do some other work (simulate switching context).
    ```bash
    git checkout -b quick-fix
    echo "Quick fix" > quickfix.txt
    git add quickfix.txt
    git commit -m "Add quick fix"
    git checkout main
    git merge quick-fix
    ```

6. Restore your stashed changes.
    ```bash
    git stash pop
    ```
    - `pop` applies the stash and removes it from the stash list
    - Use `git stash apply` to keep the stash for later

7. Verify your changes are back.
    ```bash
    git status
    cat hello.txt
    ```

8. Clean up - discard these work-in-progress changes.
    ```bash
    git checkout -- hello.txt
    rm wip.txt
    ```

**Useful stash commands:**
- `git stash` - Stash current changes
- `git stash list` - List all stashes
- `git stash pop` - Apply most recent stash and remove it
- `git stash apply` - Apply most recent stash but keep it
- `git stash drop` - Delete most recent stash
- `git stash clear` - Delete all stashes

---

### **Exercise 11: Undoing Changes**

**Why?** Everyone makes mistakes. Knowing how to undo changes safely is crucial. Different commands handle different situations.

#### **Scenario A: Discard uncommitted changes in a file**
```bash
# Make a change
echo "Mistake!" >> hello.txt

# View the change
git diff hello.txt

# Discard the change (restore to last commit)
git checkout -- hello.txt
# Or using newer syntax:
git restore hello.txt
```

#### **Scenario B: Unstage a file (keep changes, remove from staging)**
```bash
# Stage a file
echo "New content" >> hello.txt
git add hello.txt

# Unstage it (changes remain in working directory)
git reset HEAD hello.txt
# Or using newer syntax:
git restore --staged hello.txt

# Clean up
git checkout -- hello.txt
```

#### **Scenario C: Undo the last commit (keep changes)**
```bash
# Make and commit a change
echo "Oops" >> hello.txt
git add hello.txt
git commit -m "Accidental commit"

# Undo the commit but keep changes staged
git reset --soft HEAD~1

# Or undo the commit and unstage changes
git reset HEAD~1

# Clean up
git checkout -- hello.txt
```

#### **Scenario D: Completely remove the last commit (discard changes)**
```bash
# Make and commit a change
echo "Bad commit" >> hello.txt
git add hello.txt
git commit -m "This will be removed"

# Completely remove the commit AND changes (DANGEROUS!)
git reset --hard HEAD~1
```

**Warning:** `git reset --hard` permanently deletes changes. Use with caution!

#### **Scenario E: Undo a commit that's already pushed (safe way)**
```bash
# git revert creates a NEW commit that undoes a previous commit
# This is safe to use on shared branches
git revert HEAD
```

---

### **Exercise 12: Using Git Reflog (Recovery)**

**Why?** Even if you accidentally delete commits with `git reset --hard`, Git keeps a log of where HEAD has been. The reflog can save you!

1. Create some commits to work with.
    ```bash
    echo "Commit 1" > recovery.txt
    git add recovery.txt
    git commit -m "Recovery commit 1"

    echo "Commit 2" >> recovery.txt
    git add recovery.txt
    git commit -m "Recovery commit 2"

    echo "Commit 3" >> recovery.txt
    git add recovery.txt
    git commit -m "Recovery commit 3"
    ```

2. View your commits.
    ```bash
    git log --oneline -5
    ```

3. "Accidentally" delete the last 2 commits.
    ```bash
    git reset --hard HEAD~2
    git log --oneline -5
    ```
    - The commits seem to be gone!

4. Use reflog to find the lost commits.
    ```bash
    git reflog
    ```
    - Find the commit hash before the reset

5. Recover the lost commits.
    ```bash
    # Replace <hash> with the commit hash from reflog
    git reset --hard <hash>
    git log --oneline -5
    ```
    - Your commits are back!

**Note:** Reflog entries expire after 90 days by default.

---

## **Working with Remote Repositories**

### **Exercise 13: Cloning a Repository**

**Why?** `git clone` downloads a complete copy of a repository, including all history. This is how you start working on existing projects.

1. Clone a sample repository.
    ```bash
    cd ..
    git clone https://github.com/octocat/Hello-World.git sample-repo
    cd sample-repo
    ```

2. Explore the cloned repository.
    ```bash
    git log --oneline -5
    git branch -a
    git remote -v
    ```
    - `origin` is the default name for the remote you cloned from
    - `-a` shows all branches including remote branches

---

### **Exercise 14: Working with Remotes**

**Why?** Remote repositories enable collaboration. You push your changes to share them and pull others' changes to stay updated.

**Note:** For this exercise, you'll need a GitHub/GitLab account and create your own repository.

1. Go back to your exercise repository.
    ```bash
    cd ../git-merge-exercises
    ```

2. Add a remote (replace with your own repository URL).
    ```bash
    # git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
    ```

3. View configured remotes.
    ```bash
    git remote -v
    ```

4. **Pushing changes** (sending your commits to remote).
    ```bash
    # First push - set upstream branch
    git push -u origin main

    # Subsequent pushes
    git push
    ```

5. **Pulling changes** (getting commits from remote).
    ```bash
    git pull
    ```
    - `git pull` = `git fetch` + `git merge`

6. **Fetching changes** (download without merging).
    ```bash
    git fetch origin
    git log origin/main --oneline -5
    ```
    - Fetch downloads changes but doesn't modify your working directory
    - Useful to see what's new before merging

**Common remote commands:**
| Command | Purpose |
|---------|---------|
| `git remote -v` | List all remotes |
| `git remote add <name> <url>` | Add a new remote |
| `git push` | Upload commits to remote |
| `git pull` | Download and merge remote changes |
| `git fetch` | Download remote changes without merging |
| `git push -u origin <branch>` | Push and set upstream tracking |

---

### **Exercise 15: Creating a Pull Request Workflow**

**Why?** Pull requests (PRs) are how teams review and discuss changes before merging them. This exercise simulates a typical PR workflow.

1. Create a feature branch.
    ```bash
    git checkout -b feature/add-readme-section
    ```

2. Make changes.
    ```bash
    echo "## Contributing" >> README.md 2>/dev/null || echo "## Contributing" > README.md
    echo "We welcome contributions!" >> README.md
    git add README.md
    git commit -m "Add contributing section to README"
    ```

3. Push the branch to remote.
    ```bash
    # git push -u origin feature/add-readme-section
    ```

4. On GitHub/GitLab:
    - Navigate to your repository
    - Click "Compare & pull request" or "Create merge request"
    - Add a description of your changes
    - Request reviewers
    - After approval, merge the PR

5. After merging, clean up locally.
    ```bash
    git checkout main
    git pull
    git branch -d feature/add-readme-section
    ```

---

## **Optional Cleanup**

1. Delete feature branches after merging to keep things tidy.
    ```bash
    git branch -d feature-fast-forward
    git branch -d feature-divergent
    git branch -d feature-no-conflict
    git branch -d feature-rebase
    git branch -d quick-fix
    ```

2. View final repository state.
    ```bash
    git log --oneline --graph --decorate --all
    ```

---

## **Quick Reference**

### **Daily Workflow**
```bash
git pull                    # Get latest changes
git checkout -b feature     # Create feature branch
# ... make changes ...
git add .                   # Stage changes
git commit -m "message"     # Commit changes
git push -u origin feature  # Push to remote
# ... create PR, get review, merge ...
git checkout main           # Switch back to main
git pull                    # Get merged changes
git branch -d feature       # Delete local branch
```

### **When Things Go Wrong**
| Situation | Solution |
|-----------|----------|
| Undo uncommitted changes | `git checkout -- <file>` |
| Unstage a file | `git reset HEAD <file>` |
| Undo last commit (keep changes) | `git reset HEAD~1` |
| Undo last commit (discard changes) | `git reset --hard HEAD~1` |
| Undo a pushed commit | `git revert <hash>` |
| Find lost commits | `git reflog` |
| Save work temporarily | `git stash` |
| Wrong branch? | `git stash`, switch, `git stash pop` |

---

## **Next Steps**

After completing these exercises, explore:
- **Git hooks**: Automate tasks on commit/push
- **Git bisect**: Find which commit introduced a bug
- **Git cherry-pick**: Apply specific commits to another branch
- **Git submodules**: Include other repositories in your project
- **Signed commits**: Verify commit authenticity with GPG
